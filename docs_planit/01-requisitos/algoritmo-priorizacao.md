# Design — Algoritmo de Priorização

> Substitui `_archive/algoritmo-de-priorizacao-v1.md` (proposta inicial) e detalha/formaliza `novas_decisoes.md` §4. Esta é a versão de referência atual.
> Rastreia RF-002, RF-004, RF-014, RN-001, RN-003.

## 1. Terminologia (ver também `backlog.md` §0)

- **data_início**: data de criação da tarefa.
- **data_final**: prazo definido no cadastro (obrigatório).
- **prazo**: hoje → data_final.
- **bloco_de_tempo**: data_início → data_final.

## 2. Por que dois valores separados, não um score único

Um score único (misturando urgência e importância desde o início) é suficiente para *ranquear* tarefas, mas insuficiente para *classificar* em um dos 4 quadrantes da Matriz de Eisenhower — duas tarefas podem chegar ao mesmo score por motivos opostos (uma muito urgente e pouco importante, outra pouco urgente e muito importante) e um número só não permite distinguir os dois casos depois de calculado.

Por isso o cálculo é feito em duas etapas independentes:

### Etapa 1 — Classificação em quadrante (usa dois valores brutos, sem misturar)

- **Urgência**: calculada como percentual do `bloco_de_tempo` já consumido (`(hoje - data_início) / (data_final - data_início)`), não como dias absolutos restantes. Isso naturalmente prioriza tarefas mais antigas mesmo quando duas tarefas têm a mesma data_final, sem precisar de um fator de "idade" à parte.
- **Importância**: valor 1-5 informado no cadastro, usado diretamente.
- Cada valor passa por um limiar independente (sim/não):
  - Urgência ≥ limiar_urgência → "urgente"
  - Importância ≥ limiar_importância → "importante"
- A combinação binária dos dois define o quadrante:
  1. Urgente e importante (Eat the Frog)
  2. Urgente e pouco importante
  3. Pouco urgente e importante
  4. Pouco urgente e pouco importante (Quick win)
- **Os valores exatos dos dois limiares ficam para calibração posterior**, com exemplos reais de tarefas (item já registrado como pendente em `backlog.md` §6). A existência de dois limiares separados, porém, é estrutural — não é um detalhe de ajuste fino, é pré-requisito para o algoritmo funcionar.

### Etapa 2 — Desempate dentro do quadrante (aqui sim, mistura os dois valores)

Depois que a tarefa já está no quadrante certo, um score combinado — curva exponencial de urgência ponderada pela importância — decide, dentro daquele quadrante, qual tarefa é *a* escolhida para aparecer como a opção representante daquele quadrante na tela inicial.

- Curva exponencial (não linear) para que a prioridade suba mais rápido conforme o bloco_de_tempo se consome, alinhado à filosofia de "aparecer cedo o suficiente para nunca virar última hora" (não apenas "reagir bem quando já é última hora").
- A importância desloca o peso da curva: tarefas mais importantes atingem prioridade alta mais cedo dentro do bloco_de_tempo do que tarefas menos importantes — mas isso já não corre o risco de acúmulo tardio que uma curva por "dias restantes" teria, porque a variável de entrada é percentual do bloco consumido (que cresce desde o dia da criação, não só perto do fim) e não existe mais o atalho de prazo padrão (+30 dias) que empurraria esse acúmulo para uma data arbitrária. O usuário é obrigado a definir uma data_final coerente com a real necessidade da tarefa, o que já evita o efeito de "lote de tarefas vencendo junto".
- Parâmetros exatos da curva (base, expoente, como a importância pondera) ficam para calibração posterior, junto dos limiares da Etapa 1.

## 3. Efeito da dependência sobre o cálculo

Decisão tomada: **filtragem simples, sem alterar o score.**

- Uma tarefa com dependência pendente tem sua prioridade calculada normalmente (Etapas 1 e 2 acima, sem nenhuma alteração).
- Ela só **não entra no conjunto de candidatas às 4 opções da matriz "Pra fazer"** enquanto a dependência não for concluída.
- Efeito colateral desejado, sem lógica extra: como só tarefas sem dependência pendente competem pelas 4 opções, a tarefa "raiz" de uma cadeia de dependências aparece naturalmente ali primeiro — não é preciso buscar manualmente qual tarefa é a mais básica, nem propagar nenhum cálculo por cadeias (A depende de B depende de C resolve-se sozinho: A só se torna elegível quando B deixa de ter dependência pendente).
- A tarefa continua visível e acionável na visão "todas as tarefas" e pode ser marcada como "Pra agora" (data/hora marcada) normalmente — a confirmação explícita na conclusão (RF-012) é a única salvaguarda contra concluir fora de ordem, não uma restrição de marcação.
- Alternativa descartada: um "teto de prioridade" recursivo (`prioridade_efetiva = min(prioridade_própria, prioridade_efetiva_da_dependência)`) resolveria o mesmo problema, mas com mais complexidade de implementação sem ganho adicional — a filtragem simples já garante o efeito desejado.

## 4. O que fica para a fase de UI (adiado conscientemente)

- Selo visual "depende de: X" na tarefa, para não descobrir a pendência só na hora de concluir.
- Validação/bloqueio de dependência circular no cadastro.
- Sinal visual de "quase entrando em atraso" dentro do próprio bloco_de_tempo (reforço para o caso raro de prazo apertado, que conscientemente não virou um bucket próprio — ver §5).

## 5. Por que "prazo pode furar tudo" não virou um bucket

A regra antiga (`_archive/algoritmo-de-priorizacao-v1.md`) prevdefinia que prazo apertado furava qualquer outra prioridade. Decisão desta rodada: **não criar um bucket dedicado para isso.**

- O próprio objetivo da curva exponencial é evitar que esse caso aconteça com frequência; tratá-lo como bucket de primeira classe normalizaria como comum algo que a filosofia do sistema trata como exceção a ser prevenida.
- O comportamento de destaque de última hora já existe estruturalmente através da transição Pra fazer/Pra agora → Atrasadas (RN-005): no dia da data_final a tarefa fica "no topo"; a partir do dia seguinte vai para Atrasadas, ordenada por mais antiga primeiro.
- Reforço adicional fica só como sinal visual (§4), não como mudança estrutural de bucket.
