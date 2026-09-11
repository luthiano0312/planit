# Audit — Sessão de revisão crítica: Algoritmo de Priorização (Planit)

> Registro da sessão `/grill-me` sobre `01-requisitos/algoritmo-priorizacao.md` e documentos relacionados
> (`novas_decisoes.md`, `backlog.md`, `escopo.md`). Não substitui esses documentos — serve como
> rastro de decisão a ser incorporado neles.
>
> **Atualização:** os dois itens da seção "Em aberto" abaixo foram resolvidos em uma segunda
> sessão `/grill-me`, registrada na íntegra na seção "Em aberto — resolvido" ao final deste
> documento. As seções "Contexto" e "Decisões tomadas" (1–9) permanecem como estavam, sem mudança.
>
> **Atualização 2:** uma terceira sessão `/grill-me`, focada na pergunta 2 de
> `01-requisitos/perguntas-em-aberto.md` (RF-006), resolveu o item 12 abaixo. Não altera nada das
> seções anteriores.

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

## Em aberto — resolvido

> Os dois itens abaixo estavam listados como "adiado conscientemente" na versão anterior deste
> documento. Foram resolvidos em uma segunda sessão `/grill-me`.

### 10. RF-013 (hierarquia de projetos/sub-tarefas) — confirmado fora do MVP

**Decisão:** RF-013 continua fora do MVP.

**Raciocínio:** diferente da primeira rodada (que adiava por falta de dados de uso real), a razão
desta vez é explícita: priorizar velocidade de validação da ideia central do sistema. O objetivo
do MVP é ser o mais básico possível para testar a hipótese principal (reduzir atrito + apontar a
próxima ação), não cobrir todos os casos de uso desde o início.

**Mitigação do risco documentado:** o risco já registrado no item "Em aberto" original (algoritmo
funciona mal para projetos grandes sem next-action clara — justamente o sintoma central que
motivou o Planit) é aceito, com um paliativo: RF-014 (dependência simples, uma por tarefa, sem
cadeia complexa) cobre o caso de uso original que motivou a ideia de hierarquia (etapa importante
travada atrás de uma mais fácil). Cadastra-se cada etapa do projeto como tarefa separada,
encadeada por dependência (A depende de B depende de C).

**Limitação aceita conscientemente:** isso não é hierarquia de verdade — não há entidade "projeto"
que agrupe tarefas nem visão de progresso agregado. Só resolve o efeito prático de "não deixar a
etapa errada aparecer antes da hora".

**Gatilho para reabrir:** se, no uso real, o cadastro manual de cada etapa como tarefa separada se
mostrar ele mesmo um atrito relevante, ou se a falta de visão agregada de progresso do projeto se
tornar um problema sentido na prática.

### 11. Calibração dos limiares e da curva exponencial — valores iniciais definidos

**Decisão:** o MVP não vai para produção com limiares indefinidos. Valores iniciais provisórios
(chute deliberado e documentado como não-calibrado) são necessários porque o RF-002 não é
implementável sem algum número concreto — a calibração fina com dados reais, mencionada em
`backlog.md` §6, continua sendo o objetivo de longo prazo, mas não pode bloquear o MVP.

**Etapa 1 — Classificação em quadrante:**
- `limiar_importância`: importância ≥ 4 (escala 1–5) → "importante"
- `limiar_urgência`: percentual do bloco_de_tempo consumido ≥ 60% → "urgente"
- Piso de dias absolutos (itens 3/4/6 acima) continua sendo avaliado **antes** do percentual, sem
  mudança.

**Etapa 2 — Desempate dentro do quadrante:**
- `score = importância × e^(k × urgência_percentual)`, com **k = 2**

**Validação do `k = 2`:** testado contra um cenário concreto trazido pelo usuário — para uma
tarefa de importância 5, cadastrada com um mês (30 dias) de antecedência, a expectativa é que ela
esteja "no topo de prioridade" quando faltar uma semana para o prazo (23/30 dias consumidos ≈
76,7% de urgência).
- Com `k=2`: score dessa tarefa (≈ 23,2) já ultrapassa uma tarefa de importância 3 no próprio dia
  do prazo dela (100% de urgência, score ≈ 22,2) — bate com a expectativa do usuário e com a
  filosofia registrada em `algoritmo-priorizacao.md` §2 ("a importância desloca o ponto de
  virada").
- Testado também `k=1` (margem folgada demais) e `k=3` (inverte o resultado, contradizendo a
  intuição do usuário) — `k=2` foi o único valor que passou no teste de sanidade com margem
  razoável.

**Natureza do valor:** `k=2` não é definitivo — é ponto de partida para o MVP, sujeito a
recalibração com uso real, como já previsto desde a decisão original em
`algoritmo-priorizacao.md` §2 ("parâmetros exatos da curva... ficam para calibração posterior").

---

## Em aberto — resolvido (sessão 3)

### 12. RF-006 (distribuição automática de horários / visão de calendário) — confirmado fora do MVP

**Decisão:** RF-006, e a parte de "calendário" de RF-005 associada a ele, ficam fora do MVP, com
justificativa explícita e critério de reabertura — mesmo padrão do item 10 (RF-013).

**Contexto:** RF-006 originalmente empacotava duas coisas distintas sob o mesmo requisito:
(a) **visualização** de disponibilidade (compromissos fixos + tarefas marcadas, num calendário
tipo Google Calendar) e (b) **auto-agendamento** (o sistema sugerindo/encaixando horário sozinho).

**Raciocínio:**
- (b) reabriria uma decisão já fechada conscientemente: duração/esforço estimado não entra no
  cadastro de tarefa (RNF-001, ver item 2 acima). Auto-agendamento não é implementável sem saber
  quanto tempo uma tarefa leva.
- (a) não depende dessa premissa, mas construir um calendário próprio dentro do Planit tem valor
  questionável quando uma ferramenta madura (Google Calendar) já resolve isso bem — reconstruir
  aumentaria escopo sem testar a hipótese central do sistema, na contramão da mesma filosofia de
  velocidade de validação já usada para descartar RF-013 (item 10).

**Mitigação:** usar um calendário externo (ex.: Google Calendar) para checar disponibilidade antes
de marcar uma tarefa em "Pra agora". Nenhuma feature de calendário é construída dentro do Planit
por enquanto.

**Gatilho para reabrir:** se, no uso real, o usuário perceber que está marcando tarefas em cima de
compromissos fixos com frequência, ou esquecendo de checar a agenda externa antes de agendar uma
tarefa em "Pra agora", reabre-se a discussão — inicialmente pela via mais barata (visualização,
item (a) acima), não pelo auto-agendamento.

---

## Próximo passo sugerido
Incorporar os itens 10 e 11 acima em `01-requisitos/algoritmo-priorizacao.md` (fechar os limiares
da Etapa 1 e o `k=2` da Etapa 2, hoje marcados como "a calibrar") e em `escopo.md`/`backlog.md`
(RF-013 permanece Could/fora do MVP, sem mudança de status — só a razão registrada mudou; nenhuma
outra atualização de escopo necessária).

Incorporar o item 12 acima em `01-requisitos/backlog.md` (RF-005/RF-006 — atualizar rationale e
adicionar referência a este item), `01-requisitos/escopo.md` (mover RF-005/RF-006 de "Out of
scope" genérico para uma entrada com razão e mitigação registradas) e
`01-requisitos/perguntas-em-aberto.md` (marcar a pergunta 2 como resolvida).
