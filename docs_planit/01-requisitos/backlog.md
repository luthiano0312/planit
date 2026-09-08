# Backlog de Requisitos — Planit

> Fonte: `levantamento-bruto.md` (entrevista informal, levantamento inicial) + `novas_decisoes.md` e discussão de design subsequente (ver `02-design/algoritmo-priorizacao.md`).
> Formato: Backlog com RF/RNF classificados (projeto individual, sem contrato/regulação — não se justifica um SRS formal completo).
> **Revisão:** esta versão substitui os RF-001, RF-002 e RF-003 originais e adiciona RF-014/RF-015, refletindo a rodada de decisões sobre algoritmo de priorização, dependência entre tarefas e descarte.

## 0. Nomenclatura padronizada (uso interno de desenvolvimento)

Termos usados de forma consistente na documentação técnica a partir desta revisão. Não necessariamente os termos exibidos no frontend — isso é decisão de UI, feita à parte.

| Termo | Significado |
|---|---|
| **data_início** | Data em que a tarefa foi criada. |
| **data_final** | Data até quando a tarefa precisa ser concluída (campo obrigatório do cadastro). |
| **prazo** | Tempo restante: de hoje até data_final. |
| **bloco_de_tempo** | Intervalo total: de data_início até data_final. |

## 1. Visão geral
Sistema pessoal de organização de tarefas, hábitos e lembretes, com foco em reduzir dois atritos identificados no levantamento: (1) não saber por onde começar e (2) falta de vontade de agir mesmo sabendo o que fazer. A resposta principal do sistema é priorizar automaticamente e apontar sempre a próxima ação, para tirar do usuário a decisão de "o que fazer agora".

**Filosofia adicional (registrada nesta rodada):** o sistema não existe para lidar bem com tarefas de última hora — existe para **evitar que uma tarefa chegue à última hora**. Por isso nenhuma tarefa deve ficar invisível a ponto de ser esquecida até virar urgência (ver RNF-002).

## 2. Requisitos Funcionais (RF)

| ID     | Descrição                                                                                                              | Prioridade (MoSCoW) | Origem / rastreabilidade                                                                                                 |
| ------ | ---------------------------------------------------------------------------------------------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| RF-001 | **[Revisado]** Criar uma tarefa com nome, descrição, data_final (obrigatória) e importância (1-5, obrigatória); data/hora marcada e dependência são opcionais. Esforço/dificuldade **não** é campo de cadastro — deixou de alimentar o cálculo automático (ver decisão em `02-design/algoritmo-priorizacao.md`) | Must                 | "criar uma task... o sistema automaticamente saber como priorizar"; revisado em `novas_decisoes.md` §1                  |
| RF-002 | **[Revisado]** Calcular automaticamente a prioridade de cada tarefa a partir de urgência (derivada do bloco_de_tempo) e importância — não usa mais esforço/tempo estimado. Detalhe do cálculo em `02-design/algoritmo-priorizacao.md` | Must                | idem                                                                                                                     |
| RF-003 | **[Substituído]** Sobrepor manualmente a prioridade calculada pelo sistema — não existe mais como campo numérico de prioridade manual. O mecanismo de priorização manual passou a ser marcar uma data/hora para a tarefa, que a coloca no bucket "Pra agora" (ver RF-004) | Must                 | decisão tomada nesta rodada: comprometer-se com uma data (atrito intencional) evita o risco de "priorizar manualmente e não agir" |
| RF-004 | Exibir uma visão inicial dividida em dois blocos: **"Pra agora"** (tarefas com data/hora marcada) e **"Pra fazer"** (4 opções, uma por quadrante da Matriz de Eisenhower, calculadas automaticamente) | Must                | "lista com as tarefas diárias com uma ordem de prioridade"; detalhado em `02-design/algoritmo-priorizacao.md`            |
| RF-005 | Exibir uma visão tipo calendário (dia/semana/mês) com as atividades                                                    | Could                | "uma visão do dia/semana/mês, como a do google calendar" — segue fora do escopo desta fase, não tratado nesta rodada     |
| RF-006 | Distribuir automaticamente as atividades em horários dentro da visão de calendário                                     | Could                | mecanismo não está claro, ver "Perguntas em aberto" — não tratado nesta rodada                                            |
| RF-007 | Criar um lembrete: informação avulsa/efêmera que não é uma tarefa                                                      | Must                 | "funcionalidade de lembretes... não necessariamente são uma tarefa"; fica em área separada dos buckets de tarefa         |
| RF-008 | Associar um lembrete a uma data (quando aplicável)                                                                     | Should                | "tipo lembrar de uma data"                                                                                               |
| RF-009 | Deixar de exibir/arquivar um lembrete depois que ele perde relevância                                                  | Should                | "depois que passar eu não preciso mais ser lembrada, algo passageiro"                                                    |
| RF-010 | Registrar um hábito e marcar, dia a dia, se foi cumprido ou não                                                        | Must                 | "habit tracker... colocar o que eu fiz ou deixei de fazer"                                                               |
| RF-011 | Visualizar a continuidade/histórico de um hábito ao longo do tempo                                                     | Should                | "ajude na parte de continuidade/manutenção de hábitos"                                                                   |
| RF-012 | Marcar uma tarefa como concluída. **[Detalhado]** Se a tarefa tem dependência pendente, exige uma segunda confirmação explícita, citando qual é a tarefa da qual ela depende, antes de efetivar a conclusão | Must                 | implícito; regra de confirmação adicionada nesta rodada (ver RF-014)                                                     |
| RF-013 | Organizar tarefas maiores em sub-tarefas/passos (hierarquia)                                                           | Could                 | não retomado nesta rodada — segue como estava, a validar                                                                 |
| RF-014 | **[Novo]** Vincular uma tarefa a outra como dependência (uma dependência por tarefa; sem lógica de dependência transitiva/cadeia complexa na v1). Enquanto a dependência não é concluída, a tarefa dependente **fica de fora das 4 opções da matriz de Eisenhower** ("Pra fazer" na tela inicial), mas continua com prioridade calculada normalmente e continua visível/acionável na lista geral de todas as tarefas e no bucket "Pra agora" | Must                 | motivado pelo relato original (projeto com etapa importante travada atrás de uma mais fácil); decisão de escopo mínimo tomada nesta rodada |
| RF-015 | **[Novo]** Descartar uma tarefa (bucket "Descartadas/não feitas", ordenado por mais recente primeiro). Se outras tarefas dependem dela, o sistema pergunta ao usuário: romper o vínculo (dependente fica livre) ou descartar a dependente junto | Must                 | `novas_decisoes.md` §2.4 e §3                                                                                            |

## 3. Requisitos Não-Funcionais (RNF)

| ID | Categoria | Descrição | Prioridade |
|---|---|---|---|
| RNF-001 | Usabilidade | Registrar uma tarefa deve exigir o mínimo de campos e cliques possível (baixo atrito de entrada) — atrito alto foi apontado como causa central da procrastinação | Must |
| RNF-002 | Confiabilidade | Nenhuma tarefa pode "sumir" de vista por causa de tarefas mais urgentes aparecendo. **[Nota]** Uma tarefa com dependência pendente não viola esta regra: ela fica de fora apenas das 4 opções curadas da matriz ("Pra fazer"), mas permanece sempre visível na lista geral de tarefas — nunca fica oculta ou inacessível | Must |
| RNF-003 | Uso individual | Sistema para 1 usuário; não há requisito de autenticação/multiusuário nesta fase | Must |
| RNF-004 | Disponibilidade/acesso | Em aberto: local-only (uso num único dispositivo) ou acessível de qualquer lugar? Não tratado nesta rodada — decisão pendente que impacta bastante o Design | A decidir |

## 4. Regras de negócio (preliminares)

| ID | Regra |
|---|---|
| RN-001 | **[Revisado]** A prioridade é derivada de dois fatores calculados separadamente — urgência (% do bloco_de_tempo consumido) e importância (1-5) — que juntos definem o quadrante da Matriz de Eisenhower da tarefa. Um score combinado (curva exponencial, ponderando urgência por importância) é usado só para desempate/ordenação dentro do quadrante, nunca para a classificação em si. Detalhe completo em `02-design/algoritmo-priorizacao.md` |
| RN-002 | Um lembrete não é uma tarefa: não entra na lista priorizada de "o que fazer", é só uma informação a reter |
| RN-003 | **[Novo]** Uma tarefa com dependência pendente é excluída das 4 opções da matriz "Pra fazer", mas continua elegível para os demais buckets/visões e pode ser concluída mediante confirmação explícita citando a dependência |
| RN-004 | **[Novo]** Ao descartar uma tarefa da qual outras dependem, o sistema pergunta ao usuário se rompe o vínculo (dependentes ficam livres) ou descarta as dependentes em cascata |
| RN-005 | **[Novo]** Uma tarefa entra no bucket "Atrasadas" a partir do dia seguinte à data_final, sem zona de tolerância intermediária; dentro de "Atrasadas", ordenação é por mais antiga primeiro |

## 5. Casos de uso (informal — único ator: o usuário)

1. **Registrar tarefa** — usuário informa nome, descrição, data_final e importância (opcionalmente data/hora marcada e dependência); sistema calcula urgência × importância e posiciona a tarefa no quadrante correto automaticamente.
2. **Consultar o que fazer agora** — usuário abre o sistema e vê "Pra agora" (tarefas marcadas) e "Pra fazer" (4 opções, uma por quadrante), sem precisar decidir por onde começar.
3. **Registrar lembrete** — usuário anota algo a lembrar, com ou sem data; o lembrete some sozinho quando deixa de ser relevante.
4. **Registrar hábito do dia** — usuário marca se cumpriu ou não um hábito; sistema mantém o histórico.
5. **Concluir tarefa** — usuário marca a tarefa como feita; se ela tinha dependência pendente, o sistema pede confirmação explícita antes de efetivar.
6. **Descartar tarefa** — usuário remove uma tarefa que não será mais feita; se outras dependiam dela, o sistema pergunta como proceder com as dependentes.

Com um único ator e fluxos simples, não há necessidade de diagramas UML formais de caso de uso nesta fase (ver critério em `requisitos-analise.md`) — a lista acima já é suficiente.

## 6. Perguntas em aberto (levantar antes de fechar o Design)

Itens já resolvidos nesta rodada (ver `02-design/algoritmo-priorizacao.md` e `novas_decisoes.md`):
- ~~Qual o algoritmo exato de priorização?~~ → resolvido estruturalmente (urgência via bloco_de_tempo × importância com limiares, score de desempate por curva exponencial); **os valores numéricos exatos dos limiares e da curva seguem em aberto**, para calibração com uso real.
- ~~Hierarquia de tarefas (RF-013) entra no MVP?~~ → não, segue Could/a validar, sem mudança.

Itens que continuam completamente em aberto (não tratados nesta rodada, adiados propositalmente):
- Como funciona, na prática, a distribuição automática de horários na visão de calendário (RF-006)? É MVP ou fica para depois?
- Como o usuário é notificado de um lembrete ou hábito pendente — só dentro do app, ou notificação do sistema operacional?
- Hábitos têm frequência configurável (diária, dias específicos da semana) ou só diária no MVP?
- RNF-004: o sistema roda só localmente ou precisa ser acessível de outro dispositivo/lugar?

Itens novos, adiados explicitamente para a fase de UI (não são lacuna — decisão consciente de adiar):
- Selo visual indicando "depende de: X" numa tarefa com dependência pendente.
- Validação/bloqueio de dependência circular no cadastro (A depende de B, B depende de A).
- Sinal visual de "quase entrando em atraso" dentro do próprio bloco_de_tempo, antes do dia da data_final.

## 7. Rastreabilidade
Todos os RF/RNF acima remontam a trechos do levantamento bruto em `levantamento-bruto.md` ou a decisões de design registradas em `novas_decisoes.md` e na discussão consolidada em `02-design/algoritmo-priorizacao.md`. Por ser um backlog pessoal (não um SRS regulado), não há matriz de rastreabilidade separada — a coluna "Origem" nas tabelas acima cumpre esse papel.
