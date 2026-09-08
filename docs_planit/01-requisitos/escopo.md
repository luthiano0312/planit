# Escopo — Planit

## In scope

- Criar e concluir tarefas com atributos que alimentam uma priorização automática: nome, descrição, data_final (obrigatória), importância 1-5 (obrigatória) (RF-001, RF-002, RF-012).
- Tela inicial dividida em dois blocos:
  - **"Pra agora"** — tarefas com data/hora marcada (mecanismo de priorização manual, substitui a antiga ideia de campo numérico de prioridade) (RF-003, RF-004).
  - **"Pra fazer"** — 4 opções, uma por quadrante da Matriz de Eisenhower, calculadas automaticamente a partir de urgência (bloco_de_tempo) × importância; dentro de cada quadrante, desempate por score de curva exponencial (RF-002, RF-004; detalhe em `02-design/algoritmo-priorizacao.md`).
- Bucket **"Atrasadas"** — tarefas que passaram da data_final sem tolerância, ordenadas por mais antiga primeiro (RN-005).
- Bucket **"Descartadas"** — tarefas removidas manualmente, ordenadas por mais recente primeiro, com fluxo de pergunta quando há tarefas dependentes (RF-015, RN-004).
- Visão "todas as tarefas" separada da tela inicial, sem curadoria de 4 opções (lista completa).
- **Dependência entre tarefas, como feature mínima e barata**: uma dependência por tarefa, sem lógica de cadeia complexa. Efeito: a tarefa dependente fica de fora das 4 opções da matriz "Pra fazer" enquanto a dependência não é concluída, mas continua visível/acionável em todos os outros lugares (RNF-002 continua valendo sem exceção). Concluir uma tarefa com dependência pendente exige confirmação explícita citando a pendência (RF-014, RF-012, RN-003).
- Lembretes (informações avulsas, não são tarefas), em área separada dos buckets de tarefa (RF-007, RF-008, RF-009).
- Hábitos com registro diário de cumprimento (RF-010, RF-011).

## Out of scope (nesta fase — candidatos a MVP posterior ou fase de UI)

- Visão de calendário com distribuição automática de horários (RF-005, RF-006) — mecanismo ainda não está claro (ver `backlog.md`, seção "Perguntas em aberto").
- Hierarquia de tarefas em sub-tarefas/projetos (RF-013) — não confirmada como necessidade nesta rodada.
- Revisão espaçada / flashcards e protocolo fixo para "não sei o que fazer" — mencionados no relato, mas nunca formalizados como caso de uso.
- Múltiplos usuários / autenticação — sistema é de uso individual (RNF-003).
- Notificações (dentro do app ou do SO) para lembretes/hábitos pendentes — não decidido.
- Frequência configurável de hábitos (só diária confirmada por enquanto) — não decidido.
- Decisão local-only vs. acessível remotamente (RNF-004) — não decidido.
- Calibração numérica exata dos limiares de urgência/importância e dos parâmetros da curva exponencial — depende de uso real com exemplos concretos.
- Decisões de UI relacionadas à dependência: selo visual "depende de: X", validação de dependência circular, sinal visual de "quase atrasando" — adiadas conscientemente para a fase de desenho de UI, pois o usuário já tem um modelo mental da tela inicial no qual esses sinais não são estritamente necessários.

## Restrições de negócio

- Prazo: nenhum prazo externo — desenvolvimento pessoal, sem pressão contratual.
- Orçamento: nenhum orçamento formal — decisões de custo (ex.: hospedagem, se houver) ficam a critério do próprio usuário no Design.
- Equipe: 1 pessoa, acumulando os papéis de stakeholder e desenvolvedor.

## Restrições técnicas

- Infraestrutura existente: nenhuma — projeto novo, sem legado a manter compatibilidade.
- Compatibilidade com legado: não se aplica.
- Tecnologias já adotadas: nenhuma decisão de tecnologia foi tomada ainda — Design será refeito do zero (a base anterior, Laravel + SQLite + React PWA, está arquivada em `_archive/design-v1.md` só como referência histórica, sem obrigar reuso). O modelo de dados anterior (enum de status pendente/em_andamento/concluído) precisará ser revisto para acomodar os novos buckets (Atrasadas é derivado da data_final, não é status armazenado; Descartadas é status novo).

## Critérios de aceitação do escopo

- O usuário consegue criar uma tarefa em poucos passos e ela aparece corretamente posicionada (bucket + quadrante, quando aplicável) sem precisar decidir manualmente a ordem.
- A lista geral de tarefas nunca "esconde" uma tarefa pendente — o RNF-002 é validado na prática, inclusive para tarefas com dependência pendente (que saem só da matriz curada, não da lista geral).
- Uma tarefa com dependência pendente não pode ser concluída sem confirmação explícita citando a pendência.
- Lembretes e hábitos funcionam como funcionalidades separadas da lista de tarefas, sem se misturar na priorização.
