# ADR 0013 — Mantenedora autônoma de backlog/PR via skill `roadmap-todo`

**Status:** Accepted
**Date:** 2026-06-24 (America/New_York)
**Skill:** `skills/roadmap-todo.md` (canônica) + `.claude/skills/roadmap-todo/` (local)
**Supersede:** —

---

## Context and Problem Statement

A skill `roadmap-todo` (criada como `todo-roadmap-pilot` no PR #1, `3984ca1`) já
reconcilia `TODO.md`/`ROADMAP.md` contra o estado real do repo. O pedido seguinte
foi promovê-la a **mantenedora autônoma**: além de reconciliar docs, ela deve
manter a `main` limpa como fonte oficial — revisar/validar/mergear PRs seguros,
escolher e executar o próximo trabalho com evidência, validar, abrir PR, **mergear
quando os critérios passam**, sincronizar a `main` e atualizar o backlog; tomar
decisões arquiteturais quando há evidência (registrando ADR); e autoevoluir as
próprias regras.

**Pergunta:** Quanto poder de auto-merge e de decisão dar à skill **sem** abrir
risco de quebrar produção, vazar segredo ou perder trabalho?

## Decision Drivers

- Velocidade: o backlog tem fila contínua (slices de API, health runs); revisão
  humana por PR é gargalo para mudanças triviais/reversíveis.
- Segurança: `main` é a fonte oficial; precisa ficar sempre verde e revertível.
- Reversibilidade: preferir ações que se desfazem com `git revert`/`restore`.
- Não-fabricação: nenhuma afirmação de status sem evidência real.
- Respeito a trabalho em andamento (ex.: branch `security/p0-webhook-signatures`).

## Considered Options

1. **Manual-gate total** — a skill só propõe; humano mergeia tudo.
   - ✅ Risco mínimo. ❌ Não atende ao pedido de autonomia; gargalo em mudanças triviais.
2. **Autonomia total** — auto-merge de qualquer PR com checks verdes.
   - ✅ Máxima velocidade. ❌ Inaceitável: mudanças sensíveis/produtivas poderiam
     passar; sem distinção de escopo/causa de falha.
3. **Autonomia com guardas (ESCOLHIDA)** — auto-merge apenas de PRs do próprio
   ciclo, com escopo declarado, sem conflito, sem mudança sensível, e com checks
   verdes **ou** reds comprovadamente pré-existentes (análise de causa). Decisões
   arquiteturais exigem ADR; ações sensíveis (DNS/D1/OAuth/Gmail/Calendar/n8n/
   secrets/integrações) exigem autorização explícita registrada no TODO/ROADMAP.
   - ✅ Velocidade em mudanças seguras + trava dura no que é arriscado.
   - ❌ Exige análise de causa de checks (mais lógica na skill).

## Decision Outcome

Escolhida a **opção 3**. Critérios de auto-merge, análise de causa de checks
vermelhos (pré-existente vs introduzido pelo diff), regra **merge = deploy** (só
valida-antes-e-depois quando a base dispara deploy; aqui `deploy.yml` só roda em
tags `v*`/manual, então merge ≠ deploy) e autoevolução estão codificados em
`skills/roadmap-todo.md`. Limite de **3 mudanças coesas por rodada** (anti-loop).

### Plano de rollback

- Reverter um merge ruim: `git revert <merge-sha>` via PR (a `main` não dispara
  deploy em push, então o revert é suficiente; produção só muda em tag).
- Reverter a política: este ADR pode ser superseded por um ADR que restaure o
  manual-gate; a skill é versionada e revert-ável como qualquer arquivo.
- Desligar a autonomia imediatamente: invocar a skill só em `dry-run`/`--no-merge`.

### Impacto em produção

- **Nenhum impacto direto.** Produção (imagem Docker) só é publicada por
  `deploy.yml` em **tags `v*`** ou `workflow_dispatch`. Merges em `main` por esta
  skill **não** disparam deploy. Caso isso mude no futuro, a regra "merge = deploy"
  da skill passa a exigir validação antes/depois + rollback/hotfix/incidente.

## Consequences

- **Positivas:** main fica limpa e sem PR seguro pendente; backlog reflete trabalho
  real; decisões ficam rastreáveis (ADR); mudanças triviais não gargalam em review.
- **Negativas:** a skill assume mais responsabilidade; um bug na análise de causa
  poderia mergear um red legítimo — mitigado por "nunca mergear red causado pelo
  diff" + escopo restrito a PRs do próprio ciclo + reversibilidade.
- **Neutras:** reds pré-existentes (ex.: `cargo fmt` drift, `cargo audit` advisory
  na `main`) viram itens de TODO em vez de bloquear merges não relacionados.

## Links

- PR #1 (`3984ca1`) — criação da skill + reconciliação inicial do backlog.
- `skills/roadmap-todo.md` — política operacional (auto-merge, loop, autoevolução).
- `docs/adr/README.md` — convenção de ADR (MADR simplificado).
