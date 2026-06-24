---
name: todo-roadmap-pilot
description: Mantém TODO.md e ROADMAP.md fiéis ao estado REAL do repo e faz o projeto avançar um passo seguro por vez. Reconcilia o backlog contra evidência real (git, PRs, testes, build, CI), classifica cada item sem inventar status, executa só tarefas pequenas/reversíveis/não-produtivas, atualiza o ROADMAP quando uma fase fecha (com evidência), propõe próximos passos e emite relatório. Tem limite de trabalho por rodada (anti-loop) e nunca toca prod/deploy/migrations/DNS/D1/OAuth/Gmail/Calendar/n8n/secrets/integrações sem autorização explícita.
triggers:
  - atualizar todo
  - atualizar roadmap
  - reconciliar backlog
  - todo roadmap
  - backlog está em dia
  - próximo passo seguro
dependencies: []
---

# todo-roadmap-pilot

Piloto operacional para manter **TODO.md** (fila prática imediata) e **ROADMAP.md**
(visão estratégica: fases, marcos, evidências, riscos) sempre fiéis ao repo, e
avançar com segurança. Versão local com templates/playbook completos em
`.claude/skills/todo-roadmap-pilot/` (layout discoverable). Este arquivo é a
versão **canônica e autossuficiente**.

> **Comece sempre por `dry-run`** quando houver qualquer dúvida.

---

## Modos e argumentos

`/todo-roadmap-pilot [modo] [flags]`

- `dry-run` *(padrão)* — **só leitura**: reconcilia em memória, mostra o diff
  proposto e o relatório. Não escreve, não commita, não abre PR.
- `apply` — aplica mudanças **seguras** (docs + no máx. `--max-tasks` tarefas) e,
  se permitido, abre/atualiza um PR **em Draft**.

Flags: `--scope=docs` (padrão em apply; só TODO/ROADMAP/docs) · `--scope=all`
(permite código pequeno e seguro) · `--max-tasks=N` (padrão 1, **teto 3**) ·
`--no-pr` · `--base=<branch>` · `--phase-review`.

Sem modo indicado → **assuma `dry-run`**. Autorização vinda de goal/instrução vale
só para edição de docs/código **no escopo** — nunca para a lista sensível abaixo.

---

## Regras de segurança (HARD — nunca violar)

1. **Nunca** alterar produção, deploy, rodar migrations, ou mexer em **DNS, D1,
   OAuth, Gmail, Calendar, n8n, secrets ou integrações reais** sem **autorização
   explícita do usuário nesta conversa**.
2. **Nunca** imprimir/logar/commitar `.env*`, secrets, tokens, cookies,
   certificados, chaves privadas ou PII. Antes de `git add`, confira a lista e
   adicione **só os arquivos pretendidos** (evite `git add -A`).
3. **Nunca** force-push, push direto em `main`, ou merge. PR sempre **Draft**.
4. **Só** tarefas **pequenas, reversíveis, bem definidas, não-produtivas**.
   Ambígua/perigosa/produtiva/sensível → marcar `⛔ bloqueado` e **não executar**.
5. **Não inventar status.** "Feito/validado" exige evidência real (commit, PR,
   issue, teste, build, lint, diff, log, arquivo, doc). Sem evidência →
   `pendente`/`precisa de verificação`.
6. **Respeitar trabalho/convenções em andamento.** Working tree sujo com mudanças
   não relacionadas → isole (branch/worktree dedicada) e faça `add` seletivo.
   Respeite `.gitignore` (ex.: `.claude/skills/` é local; o canônico é `skills/`).

---

## Limite de trabalho por rodada (ANTI-LOOP)

- **1** passe de reconciliação (sempre permitido; leitura + edição de docs).
- No máx. **`--max-tasks`** tarefas acionáveis (padrão 1, teto 3).
- Por tarefa: máx. **1** implementação **+ 1** validação. Falhou → reverter,
  marcar `⛔ bloqueado` com o motivo, **parar** de executar novas tarefas.
- A seleção examina no máx. **5** candidatos e escolhe **um**.
- Sem tarefa segura → não fique em loop: gere próximos passos, relate, termine.
- A skill **não se reinvoca**; o caller não deve re-invocá-la no mesmo estado.
  Segunda rodada sem evidência nova → **relatório no-op**.

---

## Taxonomia de status (com evidência exigida)

| Status | Significado | Evidência |
|--------|-------------|-----------|
| ✅ Concluído | entregue/mergeado, não revalidado agora | commit/PR |
| ✅✔ Validado | concluído **e** validado nesta rodada | comando+resultado citados |
| 🟡 Parcial | implementação parcial; falta o resto | evidência parcial + o que falta |
| ⏳ Pendente | não iniciado, acionável | — |
| ⛔ Bloqueado | precisa de autorização/infra, ou é ambíguo/perigoso | **motivo obrigatório** |
| ➡️ Próximo passo | ação recomendada | escopo + critério de sucesso |

---

## Procedimento (ordem fixa, com gates de parada)

1. Ler `ROADMAP.md`, `TODO.md`, `README` e docs relevantes (`plans/`, `docs/`).
2. Identificar a **fase atual** pelo ROADMAP.
3. Coletar **evidência real** (read-only): `git log --oneline -30`,
   `git status --short`, `git log origin/<base>..HEAD` e `HEAD..origin/<base>`,
   `gh pr list --state merged/open`, testes/CI. **Em forks**, a evidência canônica
   de "concluído" é o histórico de `origin/<base>` (PRs podem viver no upstream).
4. Comparar **TODO × realidade**; classificar cada item (taxonomia). Detectar:
   feitos sem marcar, "concluídos" sem evidência, merges recentes ausentes do
   TODO, entradas obsoletas, **headers/itens duplicados**.
5. Reconciliar o TODO (docs): mover/arquivar concluídos, deduplicar, marcar
   bloqueados **com motivo**, reordenar por prioridade, reescrever "Próximos
   passos" no **formato de item**.
6. Selecionar o próximo item seguro/útil (≤5 candidatos; elegível só se claro,
   pequeno, reversível, não-produtivo, no `--scope`, com critério+validação).
7. Executar (só `apply` e se elegível) — a menor mudança que satisfaz o critério.
8. Validar (teste/build/typecheck/lint/diff) e **citar o resultado**. Sem
   toolchain local → "validação delegada ao CI" / "N/A no ambiente" (não fingir).
9. Atualizar o TODO após cada item (status + evidência + data).
10. **Revisão de fase:** só vire o status de uma fase no ROADMAP se houver
    evidência de que **todas** as pendências fecharam (formato de fase abaixo).
    Senão, registre uma **nota de reconciliação datada** no changelog.
11. Sem item seguro pendente → criar 3–5 próximos passos no formato de item.
12. **PR Draft** (só `apply`, salvo `--no-pr`): isolar (branch/worktree), `add`
    seletivo, commit claro, `gh pr create --draft` (em fork, fixar
    `--repo <owner>/<repo> --base <base>`). Se já há PR apropriado, **atualize-o**.
13. Emitir o **relatório**.

---

## Formatos obrigatórios

**Item de TODO:** `título` · `prioridade (P0/P1/P2)` · `escopo` · `motivo` ·
`arquivos prováveis` · `critério de sucesso` · `validação esperada` · `riscos` ·
`status`.

**Fase de ROADMAP:** `objetivo` · `status` · `evidências` · `validações` ·
`pendências` · `riscos` · `próximo marco`.

**Relatório:** itens concluídos · itens bloqueados (com motivo) · validações
executadas · arquivos alterados · PR criado/atualizado · **próximo melhor passo**
· modo + uso do limite de trabalho.

---

## Regras

- Na dúvida de segurança, **pare e marque `⛔ bloqueado`** — reconciliar docs e
  propor próximos passos é sempre seguro.
- Toda afirmação de status carrega evidência citada. Nada de fabricar.
- Termine **sempre** com um relatório.
