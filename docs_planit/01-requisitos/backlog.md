# Backlog de Requisitos — Planit

> Fonte: `levantamento-bruto.md` (entrevista informal, levantamento inicial) + `novas_decisoes.md` e discussão de design subsequente (ver `02-design/algoritmo-priorizacao.md`) + `audit.md` (sessão `/grill-me` de revisão crítica do algoritmo).
> Formato: Backlog com RF/RNF classificados (projeto individual, sem contrato/regulação — não se justifica um SRS formal completo).
> **Revisão:** esta versão substitui os RF-001, RF-002 e RF-003 originais, adiciona RF-014/RF-015 (rodada anterior) e RF-016 (esta rodada), e fecha os valores numéricos do algoritmo (limiares e curva) que antes estavam marcados como "a calibrar". Ver `audit.md` para o racional completo das decisões desta rodada.

## 0. Nomenclatura padronizada (uso interno de desenvolvimento)

Termos usados de forma consistente na documentação técnica a partir desta revisão. Não necessariamente os termos exibidos no frontend — isso é decisão de UI, feita à parte.

| Termo | Significado |
|---|---|
| **data_início** | Data em que a tarefa foi criada. |
| **data_final** | Data até quando a tarefa precisa ser concluída (campo obrigatório do cadastro). |
| **prazo** | Tempo restante: de hoje até data_final. |
| **bloco_de_tempo** | Intervalo total: de data_início até data_final. |
| **compromisso pendente** | Estado novo (RF-016): tarefa com `dias_restantes ≤ 3` que ainda não tem data/hora marcada. |

## 1. Visão geral
Sistema pessoal de organização de tarefas, hábitos e lembretes, com foco em reduzir dois atritos identificados no levantamento: (1) não saber por onde começar e (2) falta de vontade de agir mesmo sabendo o que fazer. A resposta principal do sistema é priorizar automaticamente e apontar sempre a próxima ação, para tirar do usuário a decisão de "o que fazer agora".

**Filosofia adicional (registrada nesta rodada):** o sistema não existe para lidar bem com tarefas de última hora — existe para **evitar que uma tarefa chegue à última hora**. Por isso nenhuma tarefa deve ficar invisível a ponto de ser esquecida até virar urgência (ver RNF-002).

## 2. Requisitos Funcionais (RF)

| ID     | Descrição                                                                                                              | Prioridade (MoSCoW) | Origem / rastreabilidade                                                                                                 |
| ------ | ---------------------------------------------------------------------------------------------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| RF-001 | **[Revisado]** Criar uma tarefa com nome, descrição, data_final (obrigatória) e importância (1-5, obrigatória); data/hora marcada e dependência são opcionais. Esforço/dificuldade **não** é campo de cadastro — deixou de alimentar o cálculo automático (ver decisão em `02-design/algoritmo-priorizacao.md`) | Must                 | "criar uma task... o sistema automaticamente saber como priorizar"; revisado em `novas_decisoes.md` §1                  |
| RF-002 | **[Revisado]** Calcular automaticamente a prioridade de cada tarefa a partir de urgência (derivada do bloco_de_tempo, com piso por dias absolutos restantes — ver RN-006) e importância — não usa mais esforço/tempo estimado. Detalhe do cálculo, incluindo limiares e curva já fechados para o MVP, em `01-requisitos/algoritmo-priorizacao.md` | Must                | idem; piso e limiares fechados em `audit.md` itens 3-4 e 11                                                              |
| RF-003 | **[Substituído]** Sobrepor manualmente a prioridade calculada pelo sistema — não existe mais como campo numérico de prioridade manual. O mecanismo de priorização manual passou a ser marcar uma data/hora para a tarefa, que a coloca no bucket "Pra agora" (ver RF-004) | Must                 | decisão tomada nesta rodada: comprometer-se com uma data (atrito intencional) evita o risco de "priorizar manualmente e não agir" |
| RF-004 | Exibir uma visão inicial dividida em dois blocos: **"Pra agora"** (tarefas com data/hora marcada) e **"Pra fazer"** (4 opções, uma por quadrante da Matriz de Eisenhower, calculadas automaticamente) | Must                | "lista com as tarefas diárias com uma ordem de prioridade"; detalhado em `01-requisitos/algoritmo-priorizacao.md`            |
| RF-005 | Exibir uma visão tipo calendário (dia/semana/mês) com as atividades                                                    | Could                | "uma visão do dia/semana/mês, como a do google calendar" — segue fora do escopo desta fase, não tratado nesta rodada     |
| RF-006 | Distribuir automaticamente as atividades em horários dentro da visão de calendário                                     | Could                | mecanismo não está claro, ver "Perguntas em aberto" — não tratado nesta rodada                                            |
| RF-007 | Criar um lembrete: informação avulsa/efêmera que não é uma tarefa                                                      | Must                 | "funcionalidade de lembretes... não necessariamente são uma tarefa"; fica em área separada dos buckets de tarefa         |
| RF-008 | Associar um lembrete a uma data (quando aplicável)                                                                     | Should                | "tipo lembrar de uma data"                                                                                               |
| RF-009 | Deixar de exibir/arquivar um lembrete depois que ele perde relevância                                                  | Should                | "depois que passar eu não preciso mais ser lembrada, algo passageiro"                                                    |
| RF-010 | Registrar um hábito e marcar, dia a dia, se foi cumprido ou não                                                        | Must                 | "habit tracker... colocar o que eu fiz ou deixei de fazer"                                                               |
| RF-011 | Visualizar a continuidade/histórico de um hábito ao longo do tempo                                                     | Should                | "ajude na parte de continuidade/manutenção de hábitos"                                                                   |
| RF-012 | Marcar uma tarefa como concluída. **[Detalhado]** Se a tarefa tem dependência pendente, exige uma segunda confirmação explícita, citando qual é a tarefa da qual ela depende, antes de efetivar a conclusão | Must                 | implícito; regra de confirmação adicionada nesta rodada (ver RF-014)                                                     |
| RF-013 | Organizar tarefas maiores em sub-tarefas/passos (hierarquia)                                                           | Could                 | **[Rationale revisado]** confirmado fora do MVP por decisão explícita de escopo (priorizar velocidade de validação da hipótese central do sistema), não por falta de dados de uso. Mitigado por RF-014 (dependência simples, encadeando etapas de um projeto como tarefas separadas). Ver `audit.md` item 10. |
| RF-014 | Vincular uma tarefa a outra como dependência (uma dependência por tarefa; sem lógica de dependência transitiva/cadeia complexa na v1). Enquanto a dependência não é concluída, a tarefa dependente **fica de fora das 4 opções da matriz de Eisenhower** ("Pra fazer" na tela inicial), mas continua com prioridade calculada normalmente e continua visível/acionável na lista geral de todas as tarefas e no bucket "Pra agora" (sem trava de agendamento — ver `audit.md` item 9) | Must                 | motivado pelo relato original (projeto com etapa importante travada atrás de uma mais fácil); decisão de escopo mínimo tomada nesta rodada |
| RF-015 | Descartar uma tarefa (bucket "Descartadas/não feitas", ordenado por mais recente primeiro). Se outras tarefas dependem dela, o sistema pergunta ao usuário: romper o vínculo (dependente fica livre) ou descartar a dependente junto | Must                 | `novas_decisoes.md` §2.4 e §3                                                                                             |
| RF-016 | **[Novo]** Marcar automaticamente uma tarefa como **"compromisso pendente"** quando `dias_restantes ≤ 3` e ela ainda não tem data/hora marcada. Não bloqueia o cadastro; a tarefa sai das 4 opções da matriz "Pra fazer" e fica em destaque próprio (ex.: mini-bucket "aguardando agendamento") até o usuário marcar data/hora (migra para "Pra agora") ou o prazo estourar (migra para "Atrasadas", perdendo o estado) | Must                 | motivado por falha identificada na sessão `/grill-me`: tarefas de vida curta recém-criadas ficavam mal classificadas pelo percentual do bloco_de_tempo; ver `audit.md` itens 3, 5, 7 e 8 e `algoritmo-priorizacao.md` §4 |

## 3. Requisitos Não-Funcionais (RNF)

| ID | Categoria | Descrição | Prioridade |
|---|---|---|---|
| RNF-001 | Usabilidade | Registrar uma tarefa deve exigir o mínimo de campos e cliques possível (baixo atrito de entrada) — atrito alto foi apontado como causa central da procrastinação. **[Reafirmado]** Por essa razão, novos campos de cadastro (tempo estimado, dificuldade, impacto, categoria) e Machine Learning para aprender prioridade foram avaliados e descartados conscientemente nesta rodada — ver `audit.md` itens 1-2 e `escopo.md` | Must |
| RNF-002 | Confiabilidade | Nenhuma tarefa pode "sumir" de vista por causa de tarefas mais urgentes aparecendo. **[Nota]** Uma tarefa com dependência pendente não viola esta regra: ela fica de fora apenas das 4 opções curadas da matriz ("Pra fazer"), mas permanece sempre visível na lista geral de tarefas — nunca fica oculta ou inacessível. O mesmo vale para uma tarefa em "compromisso pendente" (RF-016): sai só da matriz, nunca da lista geral | Must |
| RNF-003 | Uso individual | Sistema para 1 usuário; não há requisito de autenticação/multiusuário nesta fase | Must |
| RNF-004 | Disponibilidade/acesso | Em aberto: local-only (uso num único dispositivo) ou acessível de qualquer lugar? Não tratado nesta rodada — decisão pendente que impacta bastante o Design | A decidir |

## 4. Regras de negócio (preliminares)

| ID | Regra |
|---|---|
| RN-001 | **[Revisado]** A prioridade é derivada de dois fatores calculados separadamente — urgência (piso de dias absolutos ≤ 3, avaliado antes do percentual do bloco_de_tempo consumido — ver RN-006) e importância (1-5) — que juntos definem o quadrante da Matriz de Eisenhower da tarefa. Um score combinado (`score = importância × e^(k × urgência_percentual)`, com **k = 2**) é usado só para desempate/ordenação dentro do quadrante, nunca para a classificação em si. Limiares fechados para o MVP: importância ≥ 4 → "importante"; bloco_de_tempo consumido ≥ 60% → "urgente". Detalhe completo e racional de validação em `algoritmo-priorizacao.md` e `audit.md` item 11 |
| RN-002 | Um lembrete não é uma tarefa: não entra na lista priorizada de "o que fazer", é só uma informação a reter |
| RN-003 | Uma tarefa com dependência pendente é excluída das 4 opções da matriz "Pra fazer", mas continua elegível para os demais buckets/visões — incluindo "Pra agora" (sem trava de agendamento, `audit.md` item 9) — e pode ser concluída mediante confirmação explícita citando a dependência |
| RN-004 | Ao descartar uma tarefa da qual outras dependem, o sistema pergunta ao usuário se rompe o vínculo (dependentes ficam livres) ou descarta as dependentes em cascata |
| RN-005 | Uma tarefa entra no bucket "Atrasadas" a partir do dia seguinte à data_final, sem zona de tolerância intermediária; dentro de "Atrasadas", ordenação é por mais antiga primeiro. Uma tarefa em "compromisso pendente" que estoura o prazo migra para cá, perdendo aquele estado (RF-016) |
| RN-006 | **[Novo]** Piso de urgência por dias absolutos: se `dias_restantes ≤ 3`, a tarefa é classificada como "urgente" independentemente do percentual do bloco_de_tempo consumido, e entra em estado "compromisso pendente" (RF-016). Avaliado **antes** do cálculo percentual — elimina por construção o caso de divisão por zero (`data_final = data_início`). Ver `audit.md` itens 3, 4 e 6 |

## 5. Casos de uso (informal — único ator: o usuário)

1. **Registrar tarefa** — usuário informa nome, descrição, data_final e importância (opcionalmente data/hora marcada e dependência); sistema calcula urgência × importância e posiciona a tarefa no quadrante correto automaticamente.
2. **Consultar o que fazer agora** — usuário abre o sistema e vê "Pra agora" (tarefas marcadas) e "Pra fazer" (4 opções, uma por quadrante), sem precisar decidir por onde começar.
3. **Registrar lembrete** — usuário anota algo a lembrar, com ou sem data; o lembrete some sozinho quando deixa de ser relevante.
4. **Registrar hábito do dia** — usuário marca se cumpriu ou não um hábito; sistema mantém o histórico.
5. **Concluir tarefa** — usuário marca a tarefa como feita; se ela tinha dependência pendente, o sistema pede confirmação explícita antes de efetivar.
6. **Descartar tarefa** — usuário remove uma tarefa que não será mais feita; se outras dependiam dela, o sistema pergunta como proceder com as dependentes.
7. **[Novo] Resolver compromisso pendente** — uma tarefa cadastrada com `dias_restantes ≤ 3` (ou que chega a esse ponto) sem data/hora marcada aparece destacada como "aguardando agendamento"; o usuário marca data/hora (ela migra para "Pra agora") ou deixa passar o prazo (ela migra para "Atrasadas").

Com um único ator e fluxos simples, não há necessidade de diagramas UML formais de caso de uso nesta fase (ver critério em `requisitos-analise.md`) — a lista acima já é suficiente.

## 6. Perguntas em aberto (levantar antes de fechar o Design)

Itens totalmente resolvidos na sessão `/grill-me` (ver `audit.md`):
- ~~Qual o algoritmo exato de priorização?~~ → resolvido por completo: estrutura (urgência via piso de dias absolutos + percentual do bloco_de_tempo, avaliados nessa ordem, × importância com limiares) **e** os valores numéricos iniciais (limiar_importância ≥ 4, limiar_urgência ≥ 60%, piso ≤ 3 dias, k = 2). Segue como trabalho futuro apenas a *recalibração fina* desses valores com uso real — não é mais uma lacuna de design.
- ~~Hierarquia de tarefas (RF-013) entra no MVP?~~ → não, confirmado fora do MVP por decisão explícita de escopo (ver RF-013 e `audit.md` item 10).
- ~~Machine Learning para aprender/sugerir prioridade?~~ → descartado conscientemente (inviável no ritmo de uso pessoal — ver `audit.md` item 1).
- ~~Novos campos de cadastro (tempo estimado, dificuldade, impacto, categoria)?~~ → descartados conscientemente, para preservar o baixo atrito de cadastro (RNF-001; ver `audit.md` item 2).

Itens que continuam completamente em aberto (não tratados nesta rodada, adiados propositalmente):
- Como funciona, na prática, a distribuição automática de horários na visão de calendário (RF-006)? É MVP ou fica para depois?
- Como o usuário é notificado de um lembrete, hábito pendente ou tarefa em "compromisso pendente" — só dentro do app, ou notificação do sistema operacional?
- Hábitos têm frequência configurável (diária, dias específicos da semana) ou só diária no MVP?
- RNF-004: o sistema roda só localmente ou precisa ser acessível de outro dispositivo/lugar?

Itens novos, adiados explicitamente para a fase de UI (não são lacuna — decisão consciente de adiar):
- Selo visual indicando "depende de: X" numa tarefa com dependência pendente.
- Validação/bloqueio de dependência circular no cadastro.
- Sinal visual de "quase entrando em atraso" dentro do próprio bloco_de_tempo, antes do dia da data_final.
- Desenho concreto do destaque de "compromisso pendente" (RF-016) — mecânica fechada, forma visual em aberto.

## 7. Rastreabilidade
Todos os RF/RNF acima remontam a trechos do levantamento bruto em `levantamento-bruto.md`, a decisões de design registradas em `novas_decisoes.md`, à discussão consolidada em `01-requisitos/algoritmo-priorizacao.md`, ou à sessão de revisão crítica registrada em `audit.md`. Por ser um backlog pessoal (não um SRS regulado), não há matriz de rastreabilidade separada — a coluna "Origem" nas tabelas acima cumpre esse papel.
