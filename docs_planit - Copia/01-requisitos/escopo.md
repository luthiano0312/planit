# Escopo — Planit

## In scope

- Criar e concluir tarefas com atributos que alimentam uma priorização automática: nome, descrição, data_final (obrigatória), importância 1-5 (obrigatória) (RF-001, RF-002, RF-012).
- Tela inicial dividida em dois blocos:
  - **"Pra agora"** — tarefas com data/hora marcada (mecanismo de priorização manual, substitui a antiga ideia de campo numérico de prioridade) (RF-003, RF-004).
  - **"Pra fazer"** — 4 opções, uma por quadrante da Matriz de Eisenhower, calculadas automaticamente a partir de urgência (bloco_de_tempo) × importância; dentro de cada quadrante, desempate por score de curva exponencial (RF-002, RF-004; detalhe em `01-requisitos/algoritmo-priorizacao.md`).
- Bucket **"Atrasadas"** — tarefas que passaram da data_final sem tolerância, ordenadas por mais antiga primeiro (RN-005).
- Bucket **"Descartadas"** — tarefas removidas manualmente, ordenadas por mais recente primeiro, com fluxo de pergunta quando há tarefas dependentes (RF-015, RN-004).
- Visão "todas as tarefas" separada da tela inicial, sem curadoria de 4 opções (lista completa).
- **Dependência entre tarefas, como feature mínima e barata**: uma dependência por tarefa, sem lógica de cadeia complexa. Efeito: a tarefa dependente fica de fora das 4 opções da matriz "Pra fazer" enquanto a dependência não é concluída, mas continua visível/acionável em todos os outros lugares (RNF-002 continua valendo sem exceção), inclusive podendo ser marcada em "Pra agora" sem trava de agendamento. Concluir uma tarefa com dependência pendente exige confirmação explícita citando a pendência (RF-014, RF-012, RN-003).
- **Estado "compromisso pendente"** *(novo)*: quando `dias_restantes ≤ 3` e a tarefa ainda não tem data/hora marcada, ela sai das 4 opções da matriz "Pra fazer" e fica em destaque próprio até o usuário agendar (migra para "Pra agora") ou o prazo estourar (migra para "Atrasadas"). Não bloqueia o cadastro da tarefa. Mesmo piso de dias absolutos usado na classificação de urgência (RF-016, RN-006; detalhe em `algoritmo-priorizacao.md` §4).
- Valores iniciais fechados para o algoritmo de priorização do MVP: limiares de urgência/importância (percentual do bloco_de_tempo ≥ 60%, importância ≥ 4, piso de 3 dias) e parâmetro `k = 2` da curva exponencial de desempate (RN-001, RN-006; detalhe e validação em `algoritmo-priorizacao.md`).
- Lembretes (informações avulsas, não são tarefas), em área separada dos buckets de tarefa (RF-007, RF-008, RF-009).
- Hábitos com registro diário de cumprimento (RF-010, RF-011).

## Decisões conscientemente descartadas (não são lacunas)

Avaliadas e rejeitadas na sessão de revisão crítica registrada em `audit.md` — mantidas aqui para não serem reabertas sem motivo:

- **Machine Learning para aprender ou sugerir a prioridade de tarefas**: inviável no ritmo de uso pessoal do sistema (~5-6 tarefas/semana); juntar dados suficientes levaria anos, muito mais que o tempo necessário para uma fórmula determinística já funcionar bem desde o dia 1 (`audit.md` item 1). Pode ser reaberto no futuro se o sistema seguir em uso por anos e o problema ainda incomodar.
- **Novos campos estruturados no cadastro** (tempo estimado, dificuldade, impacto, categoria): contrariariam RNF-001 (baixo atrito de cadastro), que é a causa central que motivou o sistema inteiro. Casos como "tarefa rápida furando uma importante" já são cobertos manualmente pela escolha Quick Win / Eat the Frog dentro do quadrante (`audit.md` item 2).

## Out of scope (nesta fase — candidatos a MVP posterior ou fase de UI)

- Visão de calendário com distribuição automática de horários (RF-005, RF-006) — mecanismo ainda não está claro (ver `backlog.md`, seção "Perguntas em aberto").
- Hierarquia de tarefas em sub-tarefas/projetos (RF-013) — confirmada fora do MVP por decisão explícita de escopo (velocidade de validação da hipótese central do sistema), mitigada por RF-014 (dependência simples); ver `audit.md` item 10.
- Revisão espaçada / flashcards e protocolo fixo para "não sei o que fazer" — mencionados no relato, mas nunca formalizados como caso de uso.
- Múltiplos usuários / autenticação — sistema é de uso individual (RNF-003).
- Notificações (dentro do app ou do SO) para lembretes, hábitos pendentes ou tarefas em "compromisso pendente" — não decidido.
- Frequência configurável de hábitos (só diária confirmada por enquanto) — não decidido.
- Decisão local-only vs. acessível remotamente (RNF-004) — não decidido.
- **Recalibração fina** dos limiares de urgência/importância e do parâmetro `k` da curva exponencial com dados reais de uso — os valores iniciais para o MVP já estão fechados (ver "In scope" acima); só o ajuste fino posterior segue fora desta fase.
- Decisões de UI relacionadas à dependência e ao "compromisso pendente": selo visual "depende de: X", validação de dependência circular, sinal visual de "quase atrasando", desenho do destaque de "aguardando agendamento" — adiadas conscientemente para a fase de desenho de UI, pois o usuário já tem um modelo mental da tela inicial no qual esses sinais não são estritamente necessários.

## Restrições de negócio

- Prazo: nenhum prazo externo — desenvolvimento pessoal, sem pressão contratual.
- Orçamento: nenhum orçamento formal — decisões de custo (ex.: hospedagem, se houver) ficam a critério do próprio usuário no Design.
- Equipe: 1 pessoa, acumulando os papéis de stakeholder e desenvolvedor.

## Restrições técnicas

- Infraestrutura existente: nenhuma — projeto novo, sem legado a manter compatibilidade.
- Compatibilidade com legado: não se aplica.
- Tecnologias já adotadas: nenhuma decisão de tecnologia foi tomada ainda — Design será refeito do zero (a base anterior, Laravel + SQLite + React PWA, está arquivada em `_archive/design-v1.md` só como referência histórica, sem obrigar reuso). O modelo de dados anterior (enum de status pendente/em_andamento/concluído) precisará ser revisto para acomodar os novos buckets e estados (Atrasadas é derivado da data_final, não é status armazenado; Descartadas é status novo; "compromisso pendente" é um estado derivado do piso de dias absolutos, não um status permanente).

## Critérios de aceitação do escopo

- O usuário consegue criar uma tarefa em poucos passos e ela aparece corretamente posicionada (bucket + quadrante, quando aplicável) sem precisar decidir manualmente a ordem.
- A lista geral de tarefas nunca "esconde" uma tarefa pendente — o RNF-002 é validado na prática, inclusive para tarefas com dependência pendente e para tarefas em "compromisso pendente" (que saem só da matriz curada, não da lista geral).
- Uma tarefa com dependência pendente não pode ser concluída sem confirmação explícita citando a pendência.
- Uma tarefa com `dias_restantes ≤ 3` sem data/hora marcada é sinalizada como "compromisso pendente" e sai da matriz "Pra fazer" sem bloquear o cadastro.
- Lembretes e hábitos funcionam como funcionalidades separadas da lista de tarefas, sem se misturar na priorização.
