> **Status: Parcialmente substituído.** Este documento registra a primeira rodada de decisões. Uma discussão posterior revisou a seção 3 (dependência — mudou de "invisível até desbloquear" para "sempre visível + confirmação na conclusão + filtrada só da matriz") e a seção 4 (algoritmo — formalizado com urgência/importância calculadas separadamente, não um score único). As versões atuais estão em `backlog.md`, `escopo.md` e `02-design/algoritmo-priorizacao.md`. Mantido aqui como registro histórico da sessão original.

# Novas Decisões — Algoritmo de Priorização e Fluxo (Planit)

> Origem: discussão de design realizada após o `backlog.md`, revisando e detalhando RN-001, RF-001, RF-003, RF-013, RNF-002.
> Este documento substitui a abordagem inicial de "score único" e registra o fluxo definido a partir da proposta trazida em `Algoritmo de priorização.md`.

## 1. Cadastro de tarefa

| Campo | Obrigatório? | Descrição |
|---|---|---|
| Nome | Sim | Curto, só para identificar a tarefa (ex.: "atividade geo") |
| Descrição | Sim | Onde ficam os detalhes (ex.: páginas do livro, o que foi passado) |
| Prazo | Sim | Decisão consciente: **sem prazo obrigatório, tarefas simples e pouco importantes tendem a ser procrastinadas indefinidamente**. Sugestão de UX para reduzir atrito: atalho tipo "sem urgência real" que pré-preenche um prazo padrão distante (ex. +30 dias), mantendo o campo obrigatório sem exigir pensar numa data exata. |
| Importância | Sim | Escala subjetiva 1–5 |
| Data/hora marcada | Não | Quando definida, agenda a tarefa para aparecer em "Pra agora" |
| Dependência | Não | Vínculo com outra tarefa que precisa ser concluída antes |

**Esforço/dificuldade não é mais um campo do cadastro.** Decisão consciente para manter o cadastro leve (RNF-001). Julgamento de "fácil/difícil" passa a ser feito manualmente pelo usuário, olhando nome/descrição, no momento de escolher como abordar a lista "Pra fazer" (ver seção 3). Ideia registrada para o futuro, caso vire necessidade: uma tag opcional de um clique (fácil/médio/difícil), sem ser campo obrigatório de preenchimento.

## 2. Buckets de exibição

A lista principal deixa de ser uma lista única ordenada por score e passa a ser dividida em 4 buckets:

### 2.1 Pra agora
- Tarefas com data/hora marcada para hoje/agora.
- Se não houver nenhuma, a seção fica vazia.
- Uma tarefa marcada permanece em "Pra agora" até o fim do dia do prazo (ver regra de atraso, seção 2.3).

### 2.2 Pra fazer
- Tarefas sem marcação ativa para agora.
- Organizadas nos **4 quadrantes da Matriz de Eisenhower**, calculados automaticamente a partir de prazo (urgência) × importância:
  - Urgente e importante
  - Urgente e pouco importante
  - Pouco urgente e importante
  - Pouco urgente e pouco importante
- Dentro de cada quadrante, o usuário escolhe como quer ler a lista, conforme sua disponibilidade/disposição no momento:
  - **Quick win** — começar pelo que parece mais fácil
  - **Eat the Frog** — começar pelo que parece mais difícil
  - Essa escolha é sempre manual/subjetiva (não há campo de esforço estruturado por trás).
- Os limiares exatos de "urgente" e "importante" (ex.: importância ≥ 4, prazo ≤ 3 dias) ficam para a etapa de calibração do algoritmo (seção 4).

### 2.3 Atrasadas
- Bucket separado — tarefas não entram mais em "Pra fazer" depois de atrasadas.
- **Regra de transição:** se o prazo é dia 10/08, a tarefa continua em "Pra agora"/"Pra fazer" (no topo) durante todo o dia 10/08. A partir do dia 11/08, ela passa para "Atrasadas". Não existe zona de tolerância intermediária.
- Sem sistema de prioridade calculada dentro do bucket.
- Ordenação: **mais antiga primeiro** (quem está atrasada há mais tempo aparece no topo), para evitar que uma tarefa fique esquecida no fundo da lista.

### 2.4 Descartadas / não feitas
- Tarefas removidas manualmente pelo usuário (ex.: porque o escopo de um projeto mudou).
- Ordenação: **mais recente primeiro** (funciona como um log/histórico).
- Ao descartar uma tarefa, o sistema verifica se outras tarefas dependiam dela (ver seção 3) e pergunta como proceder.

## 3. Regras de dependência

- Uma tarefa com dependência **não fica disponível/visível como acionável** enquanto a tarefa da qual depende não for concluída (bloqueio, não é só uma questão de ordenação).
- **Não existe override manual para "furar" uma dependência ainda pendente** — decisão tomada para evitar mostrar como disponível uma tarefa que, na prática, não pode ser feita ainda.
- **Fluxo de descarte com dependência:** quando a tarefa da qual outras dependiam é descartada, o sistema identifica as tarefas dependentes e pergunta ao usuário:
  - Romper o vínculo (a tarefa dependente fica livre, disponível normalmente), ou
  - Descartar a tarefa dependente junto (ela também vai para "Descartadas").
- Esse fluxo resolve o caso de uso original que motivou a dependência: mudança de escopo de um projeto que invalida uma etapa, liberando a etapa seguinte.

## 4. Algoritmo de priorização (a calibrar)

- Fatores considerados: **prazo (urgência)** e **importância** (1–5, informada no cadastro).
- Esforço/tempo estimado **não entram** no cálculo por enquanto (podem ser incorporados no futuro).
- Formato da fórmula: **curva exponencial de urgência**, em que a importância desloca o "ponto de virada" da curva:
  - Tarefas de baixa importância só sobem de prioridade quando o prazo está muito apertado.
  - Tarefas de alta importância sobem de prioridade mesmo com prazo relativamente mais folgado.
- Essa fórmula alimenta apenas a classificação dentro dos 4 quadrantes de Eisenhower do bucket "Pra fazer" — não define ordenação em "Atrasadas" nem em "Descartadas".
- Próximo passo (fora do escopo desta rodada): definir os limiares numéricos dos quadrantes e calibrar a curva com exemplos reais de tarefas.

## 5. Registro de tarefas concluídas

- Toda tarefa concluída (no prazo ou atrasada) vai para um registro único de **"Tarefas concluídas"**.
- A distinção entre concluída no prazo e concluída com atraso **não é armazenada como campo/marcação própria** — é **derivada automaticamente** comparando `data_conclusão` com `prazo`:
  - `data_conclusão <= prazo` → concluída no prazo
  - `data_conclusão > prazo` → concluída com atraso (a diferença em dias fica disponível como dado derivado)
- Essa distinção é usada apenas na camada de exibição (decisão de UI a ser feita depois), por exemplo:
  - Selo/tag (ex.: "✓ no prazo" / "✓ atrasada (2 dias)")
  - Cor diferente por status
  - Filtro/agrupamento na tela de concluídas

## 6. Itens que ficaram fora do escopo desta rodada de decisões

- Lembretes com data (RF-008): decidido que ficam em uma **área separada** dos buckets de tarefas (não entram em "Pra agora" nem em "Pra fazer"), mas o desenho de UI específico ainda precisa ser feito.
- RF-006 (distribuição automática de horários na visão de calendário): continua em aberto, não foi tratado nesta rodada.
- Limiares numéricos exatos dos quadrantes de Eisenhower e calibração fina da curva exponencial: ficam para uma etapa posterior, com exemplos reais de tarefas.
- Hierarquia de sub-tarefas (RF-013): não foi retomada nesta rodada; segue como estava no backlog original (Could, a validar).
