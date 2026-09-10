# Design — Algoritmo de Priorização

> Substitui `_archive/algoritmo-de-priorizacao-v1.md` (proposta inicial) e detalha/formaliza `novas_decisoes.md` §4. Esta é a versão de referência atual.
> Rastreia RF-002, RF-004, RF-014, RF-016, RN-001, RN-003, RN-006.
> **Revisão:** incorpora as decisões da sessão `/grill-me` registrada em `audit.md` — limiares da Etapa 1 e `k` da Etapa 2 fechados (itens 10–11), e o piso de urgência por dias absolutos com o novo estado "compromisso pendente" (itens 3–9).

## 1. Terminologia (ver também `backlog.md` §0)

- **data_início**: data de criação da tarefa.
- **data_final**: prazo definido no cadastro (obrigatório).
- **prazo**: hoje → data_final.
- **bloco_de_tempo**: data_início → data_final.

## 2. Por que dois valores separados, não um score único

Um score único (misturando urgência e importância desde o início) é suficiente para *ranquear* tarefas, mas insuficiente para *classificar* em um dos 4 quadrantes da Matriz de Eisenhower — duas tarefas podem chegar ao mesmo score por motivos opostos (uma muito urgente e pouco importante, outra pouco urgente e muito importante) e um número só não permite distinguir os dois casos depois de calculado.

Por isso o cálculo é feito em duas etapas independentes:

### Etapa 1 — Classificação em quadrante (usa dois valores brutos, sem misturar)

- **Urgência**: calculada em duas camadas, avaliadas **nesta ordem** (ver `audit.md` itens 3, 4 e 6):
  1. **Piso de dias absolutos restantes.** Se `dias_restantes ≤ 3`, a tarefa já é classificada como "urgente", independente do percentual do bloco_de_tempo consumido. Resolve o caso de tarefas de vida curta recém-criadas (percentual consumido baixo, mas pouca folga real) — falha que contradizia a filosofia central do sistema ("evitar que uma tarefa chegue à última hora"). Por ser avaliado *antes* do cálculo percentual, este piso também elimina por construção o risco de divisão por zero no caso degenerado `data_final = data_início` (prazo pra hoje), sem precisar de tratamento de erro separado.
  2. **Percentual do bloco_de_tempo consumido**, calculado só se o piso acima não disparar: `(hoje - data_início) / (data_final - data_início)`. Isso naturalmente prioriza tarefas mais antigas mesmo quando duas tarefas têm a mesma data_final, sem precisar de um fator de "idade" à parte.
- **Importância**: valor 1-5 informado no cadastro, usado diretamente.
- **Limiares fechados para o MVP** (chute deliberado, documentado como não-calibrado — ver `audit.md` item 11; a calibração fina com uso real segue como trabalho futuro, fora desta fase):
  - `limiar_importância`: importância ≥ 4 → "importante"
  - `limiar_urgência` (percentual): bloco_de_tempo consumido ≥ 60% → "urgente"
  - `limiar_piso` (dias absolutos): dias_restantes ≤ 3 → "urgente" (avaliado antes do percentual, ver item 1 acima)
- A combinação binária dos dois valores (urgência e importância) define o quadrante:
  1. Urgente e importante (Eat the Frog)
  2. Urgente e pouco importante
  3. Pouco urgente e importante
  4. Pouco urgente e pouco importante (Quick win)
- **Quando o piso de dias absolutos dispara a classificação de "urgente"**, a tarefa entra também no estado **"compromisso pendente"** (novo — ver §4) e sai da disputa pelos 4 quadrantes de "Pra fazer", pelo mesmo motivo estrutural do §3 (dependência pendente): evitar que um cálculo de prioridade "vença" uma restrição que ainda precisa de ação do usuário.

### Etapa 2 — Desempate dentro do quadrante (aqui sim, mistura os dois valores)

Depois que a tarefa já está no quadrante certo, um score combinado — curva exponencial de urgência ponderada pela importância — decide, dentro daquele quadrante, qual tarefa é *a* escolhida para aparecer como a opção representante daquele quadrante na tela inicial.

- **Fórmula fechada para o MVP** (ver `audit.md` item 11): `score = importância × e^(k × urgência_percentual)`, com **k = 2**.
- Curva exponencial (não linear) para que a prioridade suba mais rápido conforme o bloco_de_tempo se consome, alinhado à filosofia de "aparecer cedo o suficiente para nunca virar última hora" (não apenas "reagir bem quando já é última hora").
- A importância desloca o peso da curva: tarefas mais importantes atingem prioridade alta mais cedo dentro do bloco_de_tempo do que tarefas menos importantes — mas isso já não corre o risco de acúmulo tardio que uma curva por "dias restantes" teria, porque a variável de entrada é percentual do bloco consumido (que cresce desde o dia da criação, não só perto do fim) e não existe mais o atalho de prazo padrão (+30 dias) que empurraria esse acúmulo para uma data arbitrária. O usuário é obrigado a definir uma data_final coerente com a real necessidade da tarefa, o que já evita o efeito de "lote de tarefas vencendo junto".
- **Validação do `k = 2`** (ver `audit.md` item 11): testado contra um cenário concreto — para uma tarefa de importância 5, cadastrada com um mês (30 dias) de antecedência, a expectativa é que ela esteja "no topo de prioridade" quando faltar uma semana para o prazo (23/30 dias consumidos ≈ 76,7% de urgência). Com `k=2`, o score dessa tarefa (≈ 23,2) já ultrapassa uma tarefa de importância 3 no próprio dia do prazo dela (100% de urgência, score ≈ 22,2) — bate com a expectativa do usuário e com a filosofia registrada acima. Foram testados também `k=1` (margem folgada demais) e `k=3` (inverte o resultado, contradizendo a intuição) — `k=2` foi o único valor que passou no teste de sanidade com margem razoável.
- **Natureza do valor**: `k=2` não é definitivo — é ponto de partida para o MVP, sujeito a recalibração com uso real, junto dos limiares da Etapa 1.

## 3. Efeito da dependência sobre o cálculo

Decisão tomada: **filtragem simples, sem alterar o score.**

- Uma tarefa com dependência pendente tem sua prioridade calculada normalmente (Etapas 1 e 2 acima, sem nenhuma alteração).
- Ela só **não entra no conjunto de candidatas às 4 opções da matriz "Pra fazer"** enquanto a dependência não for concluída.
- Efeito colateral desejado, sem lógica extra: como só tarefas sem dependência pendente competem pelas 4 opções, a tarefa "raiz" de uma cadeia de dependências aparece naturalmente ali primeiro — não é preciso buscar manualmente qual tarefa é a mais básica, nem propagar nenhum cálculo por cadeias (A depende de B depende de C resolve-se sozinho: A só se torna elegível quando B deixa de ter dependência pendente).
- A tarefa continua visível e acionável na visão "todas as tarefas" e pode ser marcada como "Pra agora" (data/hora marcada) normalmente — a confirmação explícita na conclusão (RF-012) é a única salvaguarda contra concluir fora de ordem, não uma restrição de marcação (decisão confirmada em `audit.md` item 9: sem trava no agendamento por dependência pendente, permitindo trabalho preparatório).
- Alternativa descartada: um "teto de prioridade" recursivo (`prioridade_efetiva = min(prioridade_própria, prioridade_efetiva_da_dependência)`) resolveria o mesmo problema, mas com mais complexidade de implementação sem ganho adicional — a filtragem simples já garante o efeito desejado.

## 4. Estado "compromisso pendente" (RF-016)

> Novo nesta revisão — ver `audit.md` itens 5, 7 e 8.

- **Gatilho**: o mesmo piso de dias absolutos da Etapa 1 (`dias_restantes ≤ 3`, ver §2). Quando dispara, a tarefa passa a exigir que o usuário marque data/hora — mas **isso não bloqueia o cadastro**. A captura da tarefa (nome, descrição, prazo, importância) continua instantânea e sem travas; separa o momento de captura (sempre rápido) do momento de comprometimento (pode exigir atrito, mas fica fora do fluxo de cadastro).
- **Efeito no cálculo/exibição**: a tarefa sai do conjunto de candidatas às 4 opções da matriz "Pra fazer" — mesmo padrão já usado para dependência pendente (§3, RN-003). Fica visível na lista geral e em um destaque próprio (ex.: mini-bucket "aguardando agendamento"), evitando o problema de várias tarefas de prazo curto disputando o único slot visível de um quadrante.
- **Saída do estado**, dois caminhos mutuamente exclusivos:
  1. O usuário marca data/hora → a tarefa migra para "Pra agora" (RF-004).
  2. A data_final estoura sem agendamento → a tarefa migra para "Atrasadas" (RN-005) e a marca de "compromisso pendente" é descartada. Os dois estados não coexistem; o bucket "Atrasadas" (mais antiga primeiro) já é sinal suficientemente forte por si só.
- **Decisões relacionadas descartadas conscientemente** (ver `audit.md` itens 1 e 2, também registradas em `escopo.md`): nem Machine Learning para sugerir/aprender prioridade, nem novos campos de cadastro (tempo estimado, dificuldade, impacto, categoria) entram no MVP — o piso + "compromisso pendente" resolvem o caso motivador (tarefas de prazo curto mal priorizadas) sem aumentar o atrito de cadastro (RNF-001).

## 5. O que fica para a fase de UI (adiado conscientemente)

- Selo visual "depende de: X" na tarefa, para não descobrir a pendência só na hora de concluir.
- Validação/bloqueio de dependência circular no cadastro.
- Sinal visual de "quase entrando em atraso" dentro do próprio bloco_de_tempo (reforço para o caso raro de prazo apertado, que conscientemente não virou um bucket próprio — ver §6).
- Desenho concreto do destaque de "compromisso pendente" (mini-bucket, badge, notificação in-app) — a mecânica está fechada em §4, mas a forma visual é decisão de fase de UI.

## 6. Por que "prazo pode furar tudo" não virou bucket

A regra antiga (`_archive/algoritmo-de-priorizacao-v1.md`) previa que prazo apertado furava qualquer outra prioridade. Decisão desta rodada: **não criar um bucket dedicado para isso.**

- O próprio objetivo da curva exponencial é evitar que esse caso aconteça com frequência; tratá-lo como bucket de primeira classe normalizaria como comum algo que a filosofia do sistema trata como exceção a ser prevenida.
- O comportamento de destaque de última hora já existe estruturalmente através da transição Pra fazer/Pra agora → Atrasadas (RN-005): no dia da data_final a tarefa fica "no topo"; a partir do dia seguinte vai para Atrasadas, ordenada por mais antiga primeiro.
- Reforço adicional fica só como sinal visual (§5), não como mudança estrutural de bucket.
- **Nota (revisão desta rodada):** isso é distinto do estado "compromisso pendente" (§4). O bucket descartado aqui *sobreporia* a prioridade de tudo o mais quando o prazo ficasse apertado; já o piso de dias absolutos + "compromisso pendente" não alteram o score de ninguém — apenas exigem uma ação do usuário (agendar) e retiram a tarefa específica da disputa pelos 4 quadrantes, sem "furar" a fila de outras tarefas.
