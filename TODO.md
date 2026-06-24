# TODO

Fila operacional curta do GarraIA/GarraRUST. Complementa `ROADMAP.md` (direção de
produto): aqui ficam o que foi concluído (com evidência), o que está parcial,
bloqueado ou adiado, e os próximos passos curtos da próxima sessão.

**Atualizado:** 2026-06-24 ~14:00 (America/New_York) — rodada do loop horário da
`roadmap-todo` (mantenedora autônoma) sobre `origin/main` @ `08d07c4`.

> **Fonte de evidência:** histórico de commits de `origin/main`. Os PRs vivem no
> upstream `michelbr84`; `origin` (`mercercaio-ship-it`) sincroniza via merge —
> os números `#NNN` referenciam PRs do upstream e servem de rastreabilidade.

---

## ✅ Concluído — rodadas da mantenedora autônoma (2026-06-24)

- **Rodada loop ~14:00 — reconciliação §3.6/§3.8 + status Docs Tier 2.**
  Verificado que os checkboxes de §3.6 (chat) e §3.8 (Docs Tier 2: schema
  migrations 026–029 + endpoints `/v1/doc-pages*` + busca) **já estão `[x]` com
  plan/GAR** — nada a reconciliar. **Docs Tier 2 NÃO fecha**: faltam colaboração
  em tempo real (CRDT/WS, ADR 0008), embeds (task/file/chat `/garra`) e o critério
  de aceite Tier 2. Status registrado em `ROADMAP.md` §3.8. (Sem mudança de código;
  docs-only, reversível.)
- **PR #1 mergeado na `main`** (`3984ca1`) — skill operacional (canônica em
  `skills/` + registro `CLAUDE.md`) + reconciliação do backlog. Validação:
  Coverage (build+test) verde + Clippy/MSRV/cargo-deny/CodeQL/gitleaks verdes;
  reds pré-existentes registrados como dívida de CI (ver Próximos passos). Merge
  **não dispara deploy** (`deploy.yml` só em tags `v*`/manual).
- **Skill promovida a mantenedora autônoma e renomeada → `roadmap-todo`**
  (`skills/roadmap-todo.md`); decisão registrada em **[ADR 0013](docs/adr/0013-autonomous-maintainer-skill.md)**
  (autonomia com guardas). Entrega desta rodada (autoevolução da skill).

## ✅ Concluído desde 2026-06-10 (verificado em `origin/main`)

Reconciliação dos **82 commits** mergeados após GAR-835 (PR #706, boundary do
TODO anterior). Agrupado por tema; commits de `docs(tracking)`/`docs(plans)` e as
notas de health run contam como **evidência**, não como entregas separadas.

- **Docs Tier 2 (Notion-like) — epic expandido (7 endpoints).**
  GAR-837 single-page CRUD (#709), GAR-840 blocks CRUD (#712), GAR-845 page
  versions (#717), GAR-847 page duplicate (#720), GAR-850 version restore (#723),
  GAR-853 single block fetch (#725), GAR-858 `doc_page_mentions` + inbox (#730).
- **Self-service `/v1/me/*` + Compliance LGPD/GDPR (10 endpoints).**
  GAR-866 list/revoke sessions (#742), GAR-869 revoke-all sessions (#747),
  GAR-871 api-keys CRUD (#751), GAR-874 update api-key (#755), GAR-876 password
  change (#757), GAR-881 personal audit trail (#766), GAR-884 `DELETE /v1/me`
  soft-delete (#771), **GAR-885 `GET /v1/me/export` — LGPD art. 20 / GDPR
  portabilidade (#774)**, **GAR-888 `POST /v1/me/anonymize` — LGPD art. 12 / GDPR
  art. 4(5) (#777)**, GAR-860 `GET /v1/me/doc-pages` inbox (#735).
- **Groups & chat members (2).** GAR-864 `GET` single chat member (#738),
  GAR-890 `DELETE /v1/groups/{id}` owner-only soft-delete (#779).
- **Busca unificada — slices 16+17.** GAR-856 `types=doc_pages` e
  `types=doc_blocks` (#728).
- **Qualidade / mutation testing.** GAR-891 Q6.15 — kill 4 mutants em
  `password.rs` + `audit_workspace.rs` (#780).
- **Deps / CI / segurança de dependências.** GAR-894 split sqlx + fix
  **RUSTSEC-2026-0182** (wasmtime-wasi 45.0.2) (#787), GAR-844 retry de CI em
  401 transitório (#716); chores: bump `@playwright/test` 1.60→1.61 (#781),
  branch-cleanup (#741).
- **Cadência de segurança — health runs 107–145.** Notas de status operacionais
  (GAR-836…GAR-893, ~31 entradas, PRs #707–#786). São registros de
  saúde/observabilidade, **não** correções de vulnerabilidade.

> ⚠️ **Não reivindicado** (lacunas na sequência, sem evidência em `origin/main`):
> GAR-851, GAR-862, GAR-875, GAR-887, GAR-889 — não marcar como concluído.

---

## ✅ Concluído em sessões anteriores (arquivo compacto)

Mantido só para rastreabilidade (detalhe nas PRs / `plans/`):

- GAR-806 GET task comment · GAR-800 PATCH task-label · GAR-798 GET thread ·
  GAR-795 PATCH task comment (#644) · GAR-794 POST accept invite ·
  GAR-777 GET `/v1/me/invites` (#621) · GAR-780 GET/DELETE invite revocation (#625) ·
  GAR-767 GET `/v1/me/files` · GAR-765 GET `/v1/me/chats` ·
  GAR-733 search slice 14 (groups, #561) · GAR-705 health run 30 (#508) ·
  GAR-467 Q6.5 mutation/audit (#509) · GAR-702 health run 28 (#504) ·
  GAR-703 search slice 5 (files, #505) · GAR-697 search slice 4 (has_attachment).
- GAR-835 Docs Tier 2 scaffold — migration 026 + POST/GET `/v1/.../doc-pages`
  (PR #706, `54f88bc`). *(boundary desta reconciliação)*

---

## 🟡 Parcialmente feito

- **GAR-603 — Runpod Load Balancer Serverless.**
  - Feito: `garra start` HTTP, bind `0.0.0.0`, rotas `/ping` e `/health`,
    `PORT`/`HOST`, Dockerfile sem REPL, docs de deployment.
  - Falta: smoke Docker local + smoke público
    `https://<ENDPOINT_ID>.api.runpod.ai/ping`; suporte a `PORT_HEALTH` separado
    de `PORT`. → ver bloqueio abaixo.

---

## ⛔ Bloqueado (precisa de autorização ou infra externa)

- **Smoke Docker/Runpod do GAR-603.** Motivo: exige runtime Docker e/ou endpoint
  Runpod público — ação operacional/infra fora do escopo seguro de docs. Destrava
  com autorização + ambiente Docker.
- **GAR-374 — validação object storage S3-compatible.** Motivo: depende de
  MinIO/S3/R2/GCS ou CI com serviço externo configurado.
- **GAR-504 — benchmark evidence run.** Motivo: depende de host dedicado.
- **GAR-410 — CredentialVault final.** Motivo: item crítico/amplo de segurança;
  exige toolchain Rust local + revisão profunda.

---

## ⏳ Adiado com justificativa

- **GAR-372 / Fase 2.1 — RAG embeddings.** Exige toolchain Rust + testes; adiar
  até ambiente com `cargo`/`rustc`.
- **Execução async/provider-backed das native skills (GarraMaxPower).** Slice
  próprio, após decidir o fechamento do épico.

---

## ➡️ Próximos passos recomendados

> Formato de item — ver `.claude/skills/roadmap-todo/references/templates.md`.

### [P1] Fechar Docs Tier 2 (§3.8): colaboração real-time + embeds — ⛔ bloqueado (código/toolchain)
- **Prioridade:** P1
- **Escopo:** colaboração em tempo real (CRDT/OT + WS `/v1/doc-pages/{id}/stream` com
  awareness, ADR [0008](docs/adr/0008-doc-collaboration.md)) e embeds (task/file/chat
  `/garra`) — o que falta para satisfazer o critério de aceite Tier 2.
- **Motivo:** single-editor (CRUD/versions/mentions/search/inbox) já entregue (PRs
  #706–#735); isto é o restante para fechar Docs Tier 2.
- **Arquivos prováveis:** `crates/garraia-gateway` (rotas WS), `garraia-db` (sync),
  `docs/adr/0008-doc-collaboration.md`.
- **Critério de sucesso:** 2 usuários editam a mesma página sem perder input;
  embed de task atualiza < 1s; página 500 blocos < 500ms p95.
- **Validação esperada:** testes de integração WS + CI verde.
- **Riscos:** mudança de runtime ampla → fatiar em slices pequenos; precisa toolchain Rust.
- **Status:** ⛔ bloqueado — trabalho de código (precisa `cargo`/toolchain local); fatiar em slices.

### [P1] Corrigir drift de formatação Rust na `main` (Format Check vermelho) — ⛔ bloqueado (toolchain)
- **Prioridade:** P1
- **Escopo:** rodar `cargo fmt --all` e commitar só a reformatação (zero mudança de lógica).
- **Motivo:** `Format Check` (`cargo fmt --check --all`) está vermelho na `main` (baseline) — impede "tudo verde".
- **Arquivos prováveis:** crates Rust com drift (descobrir via `cargo fmt --check --all`).
- **Critério de sucesso:** `cargo fmt --check --all` verde no CI.
- **Validação esperada:** Format Check verde no PR.
- **Riscos:** baixo (só formatação); exige toolchain Rust.
- **Status:** ⛔ bloqueado — sem `cargo`/`rustfmt` nesta sessão; executar quando houver toolchain ou via runner com Rust.

### [P1] Resolver advisory de `cargo audit` na `main` (Security Audit vermelho) — ⏳ pendente
- **Prioridade:** P1
- **Escopo:** identificar o RUSTSEC apontado por `Security — cargo audit` e bumpar/ajustar a dependência (ou registrar ignore justificado em `deny.toml`/`audit.toml` + issue).
- **Motivo:** `Security Audit` está vermelho na `main` (baseline) — dívida de segurança real.
- **Arquivos prováveis:** `Cargo.toml` / `Cargo.lock`, `deny.toml`.
- **Critério de sucesso:** `cargo audit` verde (ou ignore com justificativa rastreada).
- **Validação esperada:** Security Audit verde no PR; `cargo deny check`.
- **Riscos:** bump pode quebrar build → mitigar validando no CI antes do merge.
- **Status:** ⏳ pendente (ler o advisory específico nos logs do CI).

### [P2] Continuar a cadência de health-run (segurança/observabilidade) — ⏳ pendente
- **Prioridade:** P2
- **Escopo:** próxima nota de health run (146+), só registro/doc.
- **Motivo:** manter a varredura contínua de superfícies.
- **Arquivos prováveis:** docs de tracking / ROADMAP §1.5
- **Critério de sucesso:** nota datada com superfícies verificadas.
- **Validação esperada:** CI verde no PR.
- **Riscos:** baixo (docs).
- **Status:** ⏳ pendente

### [P2] Preparar toolchain Rust local — ⛔ bloqueado (depende do ambiente)
- **Motivo do bloqueio:** sem `cargo`/`rustc`/`rustfmt` local, a validação de
  runtime depende do CI no PR. Instalar a toolchain destrava GAR-372/GAR-410 e
  mudanças de código mais ambiciosas.

---

## Notas de reconciliação (constantes)

- Validação de runtime Rust depende do CI quando a toolchain não está local
  (ambiente desta sessão não tinha `cargo`/`rustc`/`rustfmt`/`gitleaks`/`markdownlint`).
- A evidência canônica de "concluído" é o histórico de `origin/main`; PRs no
  upstream `michelbr84`.
