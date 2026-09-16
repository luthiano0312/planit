# Plano de implementação — Acréscimo de justificativas e gatilhos à documentação do Planit

## Contexto para o agente

O projeto Planit tem um arquivo `audit.md` (registro de uma sessão de revisão crítica `/grill-me`) que já foi incorporado aos demais documentos, mas algumas **racionalizações de calibração e gatilhos de reabertura** ficaram registrados apenas no audit. Esta tarefa copia esses conteúdos para os documentos definitivos, sem alterar nenhuma decisão, valor numérico, regra de negócio ou escopo. É apenas acréscimo de texto justificativo.

Diretório de trabalho informado pelo usuário: `G:\Meu Drive\projetos\planit\docs_planit` (localizar os arquivos pelo nome, pois as referências cruzadas usam caminhos relativos inconsistentes: `01-requisitos/algoritmo-priorizacao.md` e `02-design/algoritmo-priorizacao.md`).

**Arquivos a modificar:** `algoritmo-priorizacao.md`, `backlog.md`, `escopo.md`.
**Arquivos que NÃO devem ser tocados:** `novas_decisoes.md` (documento histórico), `perguntas-em-aberto.md`, `stakeholders.md`, `audit.md`.

---

## Edição 1 — `algoritmo-priorizacao.md` §2 (Etapa 1)

**Localizar** o bullet do limiar do piso, que está neste trecho exato:

```
  - `limiar_piso` (dias absolutos): dias_restantes ≤ 3 → "urgente" (avaliado antes do percentual, ver item 1 acima)
```

**Ação:** substituir esse bullet pela versão com a justificativa de calibração acrescentada:

```
  - `limiar_piso` (dias absolutos): dias_restantes ≤ 3 → "urgente" (avaliado antes do percentual, ver item 1 acima). **Calibração do valor 3:** escolhido pela capacidade real de execução do usuário — a maioria das tarefas cabe em ~3 dias dada sua disponibilidade média. Falso positivos (sinalizar como urgente algo que na prática é rápido) são aceitos conscientemente como custo baixo, preferíveis ao risco de uma tarefa passar despercebida (ver `audit.md` item 4).
```

---

## Edição 2 — `backlog.md`, tabela de RF (linha RF-013)

**Localizar** a célula "Origem / rastreabilidade" da linha RF-013, que contém este trecho exato:

```
**[Rationale revisado]** confirmado fora do MVP por decisão explícita de escopo (priorizar velocidade de validação da hipótese central do sistema), não por falta de dados de uso. Mitigado por RF-014 (dependência simples, encadeando etapas de um projeto como tarefas separadas). Ver `audit.md` item 10.
```

**Ação:** substituir por:

```
**[Rationale revisado]** confirmado fora do MVP por decisão explícita de escopo (priorizar velocidade de validação da hipótese central do sistema), não por falta de dados de uso. Mitigado por RF-014 (dependência simples, encadeando etapas de um projeto como tarefas separadas). **Limitação aceita conscientemente:** isso não é hierarquia de verdade — não há entidade "projeto" agrupando tarefas nem visão de progresso agregado; só resolve o efeito prático de "não deixar a etapa errada aparecer antes da hora". **Gatilho para reabrir:** se o cadastro manual de cada etapa como tarefa separada se mostrar ele mesmo um atrito relevante, ou se a falta de visão agregada de progresso se tornar um problema sentido na prática. Ver `audit.md` item 10.
```

---

## Edição 3 — `escopo.md`, seção "Out of scope"

**Localizar** o bullet:

```
- Hierarquia de tarefas em sub-tarefas/projetos (RF-013) — confirmada fora do MVP por decisão explícita de escopo (velocidade de validação da hipótese central do sistema), mitigada por RF-014 (dependência simples); ver `audit.md` item 10.
```

**Ação:** substituir por (apenas acrescenta o apontamento ao fim):

```
- Hierarquia de tarefas em sub-tarefas/projetos (RF-013) — confirmada fora do MVP por decisão explícita de escopo (velocidade de validação da hipótese central do sistema), mitigada por RF-014 (dependência simples); ver `audit.md` item 10 (limitação da mitigação e gatilho de reabertura registrados na rationale de RF-013 no `backlog.md`).
```

---

## Edição 4 — `escopo.md`, seção "Decisões conscientemente descartadas"

**Localizar** o bullet do Machine Learning:

```
- **Machine Learning para aprender ou sugerir a prioridade de tarefas**: inviável no ritmo de uso pessoal do sistema (~5-6 tarefas/semana); juntar dados suficientes levaria anos, muito mais que o tempo necessário para uma fórmula determinística já funcionar bem desde o dia 1 (`audit.md` item 1). Pode ser reaberto no futuro se o sistema seguir em uso por anos e o problema ainda incomodar.
```

**Ação:** substituir por:

```
- **Machine Learning para aprender ou sugerir a prioridade de tarefas**: o sinal de treino (aceitar/corrigir score sugerido) é teoricamente válido — a inviabilidade é apenas prática: a ~5-6 tarefas/semana, juntar as centenas/milhares de exemplos necessários levaria de 1 a 3+ anos (estimativa otimista), muito mais que o tempo necessário para uma fórmula determinística já funcionar bem desde o dia 1 (`audit.md` item 1). Pode ser reaberto no futuro se o sistema seguir em uso por anos e o problema ainda incomodar.
```

---

## Regras e verificação

1. Não alterar nada além dos quatro trechos acima — nenhum valor numérico, limiar, fórmula, prioridade MoSCoW ou status de escopo muda.
2. Preservar exatamente o estilo dos documentos: português, formatação markdown existente (bullets com `-`, negrito com `**`, referências cruzadas em crases).
3. Se alguma âncora não for encontrada exatamente como escrita, **parar e reportar** em vez de adivinhar — indica que o documento divergiu do esperado.
4. Após as edições, conferir que: (a) `audit.md` continua citado e legível como rastro; (b) não há duplicação do mesmo conteúdo em mais de um lugar além do planejado (escopo.md aponta para backlog.md em vez de repetir a limitação de RF-013); (c) as referências a `audit.md` itens 1, 4 e 10 seguem válidas.

**Fonte dos conteúdos inseridos:** `audit.md` — item 4 (Calibração do limiar = 3 dias), item 10 (limitação da mitigação de RF-013 e gatilho de reabertura), item 1 (raciocínio do descarte de ML). O agente pode confrontar o texto inserido com esses itens para validar fidelidade.
