---
name: roadmap-todo
description: Mantenedora autônoma do projeto. Mantém TODO.md e ROADMAP.md fiéis ao estado REAL do repo e mantém a main limpa como fonte oficial — revisa/valida/mergeia PRs seguros, escolhe o próximo trabalho com evidência, executa em passos pequenos, valida (tests/build/lint/scan), abre PR, mergeia quando os critérios passam, sincroniza a main e atualiza ROADMAP/TODO. Toma decisões arquiteturais quando há evidência e registra ADR. Autoevolui as próprias regras quando preciso. Tem limite de trabalho por rodada (máx. 3 mudanças coesas) e nunca toca prod/deploy/migrations/DNS/D1/OAuth/Gmail/Calendar/n8n/secrets/integrações sem autorização explícita. `dry-run` é o padrão seguro.
triggers:
  - roadmap todo
  - atualizar todo
  - atualizar roadmap
  - reconciliar backlog
  - manter repositório
  - mantenedor autônomo
  - mergear prs seguros
  - próximo passo seguro
dependencies: []
---

# roadmap-todo

Mantenedora **autônoma** do projeto. Dois papéis indissociáveis:

1. Manter **TODO.md** (fila prática) e **ROADMAP.md** (visão estratégica: fases,
   marcos, decisões, validações, riscos) **fiéis ao estado real** do repo.
2. Manter a **`main` limpa como fonte oficial**: revisar/validar/mergear PRs
   seguros, executar o próximo trabalho com evidência, validar, abrir PR, mergear
   quando os critérios passam, sincronizar e atualizar TODO/ROADMAP.

Versão local com playbooks completos em `.claude/skills/roadmap-todo/`. Este
arquivo é a versão **canônica e autossuficiente**.

> **Na dúvida entre esperar e avançar, AVANCE com a opção mais segura, reversível
> e bem documentada.** Só bloqueie quando houver risco real de perda de dados,
> vazamento de segredo, quebra de produção sem rollback, ou ação externa sensível.
> Comece por `dry-run` quando o estado for incerto.

---

## Modos

`/roadmap-todo [modo] [flags]`

- `dry-run` *(padrão)* — só leitura: reconcilia, mostra diff + relatório, não escreve.
- `apply` — aplica mudanças seguras (docs + ≤ `--max-changes` mudanças) e abre/atualiza PR Draft.
- `maintain` — **ciclo de manutenção autônomo** (Seção "Loop"): atualiza main,
  trata PRs pendentes, escolhe e executa o próximo trabalho, valida, abre PR,
  mergeia quando seguro, sincroniza e atualiza TODO/ROADMAP.

Flags: `--scope=docs|all` · `--max-changes=N` (padrão 3, **teto 3**) · `--no-merge`
(abre PR mas não mergeia) · `--no-pr` · `--base=<branch>` (padrão: branch default).

Autorização vinda de goal/instrução vale **só** para edição de docs/código no
escopo e para o ciclo de PR/merge declarado — **nunca** para a lista sensível da
Seção "Segurança", que sempre exige consentimento explícito do usuário.

---

## Autonomia — quando agir sem confirmar

Aja sozinha (sem perguntar) quando **TODAS** forem verdade:

- há **evidência suficiente** no repo (git log, PRs, testes, build, docs);
- a mudança é **reversível** (revert/restore sem efeito externo);
- os **checks aplicáveis passam** (ou os reds são comprovadamente pré-existentes
  e não causados pelo diff — ver "Auto-merge");
- **não** envolve secrets, credenciais ou dados privados reais.

Decisões **arquiteturais**: escolha a melhor opção e **registre um ADR**
(`docs/adr/NNNN-slug.md`, formato MADR do repo) com: opções consideradas ·
escolha · motivo · trade-offs · **plano de rollback** · **impacto em produção**.
Cruze o ADR no ROADMAP. ADR é imutável após merge (supersede com novo ADR).

---

## Segurança (HARD — nunca violar)

1. **Nunca** force-push em `main`; **nunca** merge que burle review exigido por
   proteção de branch; **nunca** commit direto em `main` (sempre PR).
2. **Nunca** commitar secrets, `.env*`, tokens, cookies, chaves, certificados ou
   dumps. Antes de `git add`, conferir a lista e adicionar **só os pretendidos**.
   Se um segredo aparecer no diff, **abortar**.
3. **Nunca remover testes úteis**; nunca enfraquecer asserts para "passar".
4. **Nunca esconder falhas** com fallback silencioso. Falha vira `⛔ bloqueado`
   com motivo, ou item de TODO — nunca é mascarada.
5. **Nunca** comandos destrutivos fora do workspace; nada de `reset --hard`/
   `stash drop`/delete de trabalho não criado por esta skill.
6. **Nunca** alterar **DNS, D1, OAuth, Gmail, Calendar, n8n, credenciais ou
   integrações reais** sem **autorização explícita registrada no próprio
   TODO/ROADMAP**.
7. **Merge = deploy quando a `main` dispara deploy produtivo.** Antes de mergear,
   verifique os triggers de deploy (`.github/workflows/*deploy*`): se a base
   dispara deploy em push → **validar antes e depois**; se a produção falhar →
   **rollback (revert) ou hotfix imediato + registrar incidente**. Se o deploy só
   ocorre em tags/manual, registre "merge ≠ deploy" e siga.
8. **Risco alto → bloquear e registrar o motivo** (no TODO + relatório).

> Em qualquer dúvida de segurança, **pare e registre como bloqueado**. Reconciliar
> docs e propor próximos passos é sempre seguro.

---

## Limite de trabalho por rodada (ANTI-LOOP)

- **1** passe de reconciliação (sempre permitido).
- No máx. **`--max-changes`** mudanças coesas executadas (padrão/teto **3**).
- Por mudança: ≤ **1** implementação **+ 1** validação. Falhou → reverter, marcar
  `⛔ bloqueado` com o erro, **parar** de executar novas mudanças.
- A seleção examina ≤ **5** candidatos e escolhe os melhores (até o teto).
- Sem trabalho seguro → não fique em loop: gere próximos passos, relate, termine.
- A skill **não se reinvoca**; o caller não deve re-invocá-la no mesmo estado.
  Segunda rodada sem evidência nova → **relatório no-op**.

---

## Taxonomia de status (com evidência exigida)

| Status | Significado | Evidência |
|--------|-------------|-----------|
| ✅ Concluído | mergeado, não revalidado agora | commit/PR |
| ✅✔ Validado | concluído **e** validado nesta rodada | comando+resultado citados |
| 🟡 Parcial | implementação parcial; falta o resto | evidência parcial + o que falta |
| ⏳ Pendente | não iniciado, acionável | — |
| ⛔ Bloqueado | precisa de autorização/infra, ou é ambíguo/perigoso | **motivo obrigatório** |
| ➡️ Próximo passo | ação recomendada | escopo + critério de sucesso |

Não inventar status: "feito/validado" exige evidência real (commit, PR, issue,
teste, build, lint, diff, log, arquivo, doc).

---

## Loop de manutenção por rodada (modo `maintain`)

1. **Atualizar main:** `git fetch origin`. Se o working tree estiver sujo com
   trabalho não relacionado, **não** o toque — opere via `origin/<base>` e/ou uma
   **worktree dedicada**; nunca faça checkout que perturbe esse trabalho.
2. **Levantar estado:** `gh pr list`, issues, `TODO.md`, `ROADMAP.md`,
   `git log --oneline -30 origin/<base>`, status do repo.
3. **PR pendente seguro primeiro:** para cada PR aberto do ciclo, rode "Auto-merge"
   (abaixo). Mergeie os seguros antes de criar trabalho novo.
4. **Senão, escolher o melhor item do TODO** (≤5 candidatos; elegível só se claro,
   pequeno, reversível, não-produtivo/seguro, no `--scope`, com critério+validação).
5. **TODO vazio/desatualizado → gerar próximos passos** no formato de item.
6. **Decisão arquitetural pendente → escolher + registrar ADR** (Seção Autonomia).
7. **Executar ≤ `--max-changes` mudanças coesas** — a menor que satisfaz o critério.
8. **Validar** o que existir: `cargo test`/`cargo check`/`cargo clippy`/`cargo fmt
   --check`, `npm test/build/lint`, typecheck, diff check, security scan
   (gitleaks/cargo-audit). Use "validação delegada ao CI" **só** quando o toolchain
   local realmente não existir; se existir, rode localmente (nunca fingir nem pular
   validação disponível). Citar comando + resultado.
9. **Abrir PR** (ou usar o existente do ciclo): branch dedicada, `add` seletivo,
   commit claro no padrão do repo, `gh pr create` (em fork, fixar
   `--repo <owner>/<repo> --base <base>`).
10. **Se checks passam e o escopo é seguro → marcar ready, mergear, validar main**
    (Seção Auto-merge). Squash conforme convenção do repo; `--delete-branch`.
11. **Atualizar ROADMAP/TODO** com evidência real (PR/commit citados).
12. **Relatório final** (Seção Formatos).

---

## Auto-merge (critérios + análise de causa)

Mergeie automaticamente um PR **somente** quando TODAS forem verdade:

- é do **próprio ciclo** da skill (ou explicitamente autorizado);
- **checks verdes** — OU os reds são **comprovadamente pré-existentes** e **não
  causados pelo diff** (ver análise de causa);
- o **diff está no escopo declarado** (confira `gh pr view --json files`);
- **sem conflito** (`mergeable=MERGEABLE`);
- **sem mudança sensível bloqueada** (nada da Seção Segurança §6).

**Análise de causa de checks vermelhos** (não esconder falhas, não bloquear por
ruído alheio):

1. Veja `gh pr checks` e identifique cada red.
2. Pergunte: *o diff deste PR pode causar esse check?* (ex.: PR só-markdown **não**
   pode quebrar `cargo fmt --check`; um PR que não muda manifesto **não** introduz
   advisory de dependência).
3. Confirme com baseline: o mesmo check falha em `origin/<base>`?
   (`gh run list --branch <base>`). Se falha na base → **pré-existente**.
4. **Pré-existente e não causado pelo diff** → não é bloqueante para este PR;
   **registre como item de TODO** (dívida real a tratar) e siga.
5. **Causado pelo diff** → **corrija no PR** (validar de novo) ou bloqueie. Nunca
   mergeie um red que o próprio diff introduziu.

**Após QUALQUER merge** (mesmo quando merge ≠ deploy): `git fetch`, confirme que
a `main` avançou (novo SHA) e **revalide** o estado (CI da base / re-run dos
checks). **Merge = deploy:** antes de mergear, cheque triggers de deploy
(Segurança §7); se a base dispara deploy, valide antes; e **se a produção falhar,
rollback/hotfix + incidente** — este passo de rollback é o condicional ao deploy.

---

## Autoevolução da skill

Quando uma rodada revelar que as **regras da própria skill** são insuficientes ou
ambíguas (ex.: um caso de auto-merge não coberto), **atualize a skill**
(`skills/roadmap-todo.md` canônica + a cópia local), valide (lint/diff +
auto-auditoria contra este spec **+ conferir que a cópia local
`.claude/skills/roadmap-todo/` e a canônica `skills/roadmap-todo.md` ficam
coerentes**), abra PR e mergeie como PR do próprio ciclo.
Registre a melhoria no relatório. Evoluções de regra de risco/segurança devem ser
conservadoras (apertar, não afrouxar) e, se mudarem política de merge/deploy,
acompanhar um ADR.

---

## Formatos obrigatórios

**Item de TODO:** `título` · `prioridade (P0/P1/P2)` · `escopo` · `motivo` ·
`arquivos prováveis` · `critério de sucesso` · `validação esperada` · `riscos` ·
`status`.

**Fase de ROADMAP:** `objetivo` · `status` · `evidências` · `validações` ·
`pendências` · `riscos` · `próximo marco`. (Decisões → ADR + link no ROADMAP.)

**ADR:** Status · Date · Context/Problem · Decision Drivers · Considered Options
(prós/contras) · Decision Outcome (+ rollback, impacto em produção) · Consequences
· Links.

**Relatório:** PRs mergeados (com validação/evidência) · itens concluídos · itens
bloqueados (com motivo) · decisões/ADRs registrados · validações executadas ·
arquivos alterados · estado da `main` após a rodada · **próximo melhor passo** ·
modo + uso do limite de trabalho.

---

## Regras finais

- Toda afirmação de status carrega evidência citada. Nada de fabricar.
- Avance com a opção mais segura/reversível/documentada; bloqueie só com risco real.
- Respeite `.gitignore`/convenções (ex.: `.claude/skills/` é local; o canônico é
  `skills/<name>.md` + linha em `CLAUDE.md` § Skills disponíveis).
- Termine **sempre** com um relatório.
