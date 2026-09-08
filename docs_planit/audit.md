# Audit — Sessão de revisão crítica: Algoritmo de Priorização (Planit)

> Registro da sessão `/grill-me` sobre `01-requisitos/algoritmo-priorizacao.md` e documentos relacionados
> (`novas_decisoes.md`, `backlog.md`, `escopo.md`). Não substitui esses documentos — serve como
> rastro de decisão a ser incorporado neles.

## Contexto

A sessão partiu de uma crítica ao algoritmo de priorização vigente, que levantou:
1. Um problema estrutural na definição de urgência (percentual do bloco_de_tempo).
2. Uma proposta de adicionar mais parâmetros de cadastro + Machine Learning, trazida pelo usuário
   com base em uma conversa externa (`conversa.md`).
3. Pontos menores: divisão por zero, disputa por slot único no quadrante, brecha de dependência
   no agendamento, ausência de hierarquia de projetos (RF-013).

Cada ramo foi resolvido em profundidade antes de passar ao próximo, conforme protocolo da skill `grill-me`.

---

## Decisões tomadas

### 1. Machine Learning — descartado
Sinal de treino (aceitar/corrigir score sugerido) é teoricamente válido, mas inviável na prática:
uso pessoal, ritmo de ~5-6 tarefas/semana, poucas correções reais por semana. Juntar as
"centenas/milhares" de exemplos necessários levaria de 1 a 3+ anos (otimista) — muito mais que o
tempo necessário pra uma fórmula determinística já funcionar bem desde o dia 1. Reabrir essa
discussão só faria sentido daqui a alguns anos, se o sistema seguir em uso e o problema ainda
incomodar.

### 2. Novos campos no cadastro — descartado
Tempo estimado, dificuldade, impacto, categoria etc. não entram no cadastro. Reafirma
`novas_decisoes.md` §1 (RNF-001): atrito de cadastro é o problema #1 que motivou o sistema inteiro.
Casos como "tarefa rápida furando uma importante" já são cobertos manualmente pela escolha
Quick Win / Eat the Frog dentro do quadrante — não precisam virar campo estruturado.

### 3. Piso de urgência por dias absolutos restantes
Além do percentual do bloco_de_tempo (`(hoje - data_início) / (data_final - data_início)`),
adiciona-se um piso: se `dias_restantes ≤ limiar`, a tarefa é classificada como urgente
independente do percentual consumido. Resolve o caso de tarefas de vida curta recém-criadas
ficarem mal classificadas (percentual baixo mesmo com pouca folga real) — falha que contradizia
a filosofia central do sistema ("evitar que uma tarefa chegue à última hora").

### 4. Limiar do piso = 3 dias
Calibrado pela capacidade real do usuário: a maioria das tarefas cabe em ~3 dias dada sua
disponibilidade média. Aplica-se igual a qualquer tipo de tarefa (atômica ou parte de projeto).
Falso positivo (exigir compromisso de algo que na real é rápido) é aceito como custo baixo,
preferível a deixar passar despercebido.

### 5. Novo estado: "compromisso pendente"
Quando `dias_restantes ≤ 3` (mesmo limiar do piso, item 4), a tarefa exige que o usuário marque
data/hora — mas **isso não bloqueia o cadastro**. A captura da tarefa continua instantânea e sem
travas (nome, descrição, prazo, importância); a tarefa entra no estado "compromisso pendente" e
fica sinalizada até o usuário voltar e agendar. Separa o momento de captura (deve ser sempre
rápido) do momento de comprometimento (pode exigir atrito, mas fora do fluxo de cadastro).

### 6. Ordem de avaliação: piso antes do percentual
O piso de dias absolutos é avaliado primeiro. Se `dias_restantes ≤ 3`, a tarefa já é classificada
como urgente + compromisso pendente, e o cálculo de percentual do bloco_de_tempo nem chega a
rodar. Isso elimina por construção o risco de divisão por zero no caso degenerado
`data_final = data_início` (prazo pra hoje), sem precisar de tratamento de erro separado.

### 7. "Compromisso pendente" é um bucket/estado próprio
Sai da disputa dos 4 quadrantes de "Pra fazer" — mesmo padrão já usado por dependência pendente
(RN-003). Fica visível na lista geral e num destaque próprio (ex.: mini-bucket "aguardando
agendamento") até o usuário marcar data/hora, quando migra para "Pra agora". Evita o problema de
várias tarefas de prazo curto competindo pelo único slot visível de um quadrante.

### 8. "Compromisso pendente" só existe dentro do prazo
Ao estourar a data_final sem agendamento, a tarefa migra para "Atrasadas" (regra já existente,
`novas_decisoes.md` §2.3) e a marca de "compromisso pendente" é descartada — os dois estados não
coexistem. O bucket "Atrasadas" (mais antiga primeiro) já é sinal suficientemente forte por si só.

### 9. Sem trava no agendamento por dependência pendente
Uma tarefa com dependência pendente pode ser marcada com data/hora e aparecer normalmente em
"Pra agora" (permite trabalho preparatório). A trava de dependência age **só na conclusão**
(RF-012/RN-003), e é uma **etapa de confirmação explícita citando a pendência** — não um bloqueio.
O usuário pode concluir mesmo com dependência pendente, desde que confirme ciente disso.

---

## Em aberto (adiado conscientemente)

- **RF-013 (hierarquia de projetos/sub-tarefas)** — segue fora do MVP, sem mudança nesta sessão.
  Registrado como **risco conhecido**: o algoritmo fechado aqui (urgência × importância + piso de
  3 dias) funciona bem para tarefas soltas e mal para projetos grandes sem next-action clara — que
  é justamente o cenário mais ligado ao sintoma original "não saber por onde começar". Esse risco
  é aceitável apenas enquanto RF-013 continuar fora de escopo; se for reaberto no futuro, o
  algoritmo de priorização deve ser revisitado.
- **Valores exatos dos limiares de importância** (Etapa 1 — "≥ limiar_importância") e **parâmetros
  da curva exponencial de desempate** (Etapa 2) seguem para calibração posterior com uso real,
  como já registrado em `backlog.md` §6. Sem mudança nesta sessão.

---

## Próximo passo sugerido
Incorporar as decisões 3–9 acima em `01-requisitos/algoritmo-priorizacao.md` (seções 2 e 3) e em
`escopo.md`/`backlog.md` (novo estado "compromisso pendente" como bucket, análogo ao tratamento já
dado à dependência pendente).
