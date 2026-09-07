# Claude Code handoff: feature/reconcile PR split

## Goal
Split the current `feature/reconcile` work into smaller draft PRs, with the frontend event-contract / field-mapping change submitted late.

## Important current caveat
In this current clone snapshot, only `copilot/featurereconcile` is visible locally, currently at:
- `ee147034f0edb3704a5655233590e6e547d61cd2` — `feat: add the reconcile profile (OpenRefine Reconciliation Service API)`

In the earlier working session, additional local staging branches were created and validated, but they are not present in the current visible branch list anymore. Treat the branch split below as the authoritative plan to recreate.

## Recommended PR split

### Draft PR A — safety / hardening / forms bootstrapping
Cherry-picks:
- `b6063034` — `fix: stop building innerHTML from document-editable text via template literals`
- `43b9f0d5` — `fix(annotate): FOTY0013 on commit save when user/status left as raw nodes`
- `6952c936` — `fix(annotate): backtick-brace in a doc comment broke Jinks template expansion`
- `47af6259` — `fix(annotate,forms): annotate-link doctype + dormant fore.js loader`
- `385880a6` — `docs: fix stale modules/annotation-config.xqm and annotate.html references`

Focus:
- XSS fix
- commit-save atomization/FOTY0013 fix
- annotate-link doctype fix
- forms feature bootstrapping
- directly related docs

### Draft PR B — annotate panel / UI / CSS repair
Cherry-picks:
- `ddbf8987` — `fix(annotate): inline the toolbar icon sprite, base10's loader is absent`
- `21ddec2d` — `fix(annotate): CSS specificity bug broke the whole Fore-based editor panel`
- `f633d90e` — `fix(annotate): oversized card corners + add a close/cancel button`
- `2497c053` — `fix(annotate): panel close button didn't work; header icons squashed`
- `9f1181de` — `fix(annotate): panel legibility, empty status dropdown, id-prefix bug`

Important note:
- `9f1181de` is mixed. When reconstructing PR B, keep only the UI/panel/status-option parts and exclude the annotate-tei person/reconcile hunk.

### Draft PR C — annotate-side ReconciliationService hookup (bridge PR)
Cherry-picks:
- `3c5be9b3` — `feat(annotate): wire ReconciliationService into live person authority lookup`
- `7e7376c6` — `fix(annotate): list ReconciliationService before GND for person matches`
- `0b746ad4` — `docs: warn about the connector-name typo trap; document type/limit attributes`

Important notes:
- This is bridge work between annotate and reconcile.
- `7e7376c6` may need manual conflict resolution if later field-mapping work is absent.
- `0b746ad4` is mixed; for this PR keep the `annotations.xml` part, not `reconcile.xml`.

### Draft PR D — late PR: person field-mapping / event-contract stack
Cherry-picks:
- `8a13573b` — `feat(annotate): fan out a matched authority's fields to multiple attributes`
- `da6dfbda` — `fix(annotate): real @ref-vs-@key bug behind "entity not found"/shallow info`
- `fd681a53` — `fix(annotate): fix "Entity not found" for field-mapped entities; contain preview overflow`
- `c3f0b8c6` — `fix(annotate): anno:occurrences must also match @ref, not just @key`
- `7b608f36` — `docs(annotate): clarify person's gnd=extend:gnd field mapping`
- `0b754d0a` — `docs: update reconciliation docs for current keyMap/getId fallback, add screenshots`
- `bd613fa1` — `docs(comments): audit annotate profile wiring code comments`
- `5f089b68` — `fix(annotate): repair broken cy.uploadXml calls + stale template URL in tests`

Why late:
- This is the real frontend event-contract-changing stack.
- The practical break is the move to consuming `pb-authority-select` data via `ev.detail.properties`.
- Several commits after `8a13573b` are completion/fix layers for the same mechanism and are safest reviewed together.

Important notes:
- `7b608f36` may become empty after conflict resolution.
- `0b754d0a` is mixed; keep the annotate-related `annotations.xml` updates and the screenshots `reconcile-annotate-detail.png` and `reconcile-annotate-search.png`, but not `reconcile.xml` or `reconcile-manifest.png` in PR D.

### Draft PR E — late follow-up: extend field-mapping to organization / place / work
Cherry-picks:
- `2ad80fbf` — `feat(annotate): extend field-mapping + keyMap fix to organization/place/work`
- `bce2d5a2` — `feat(annotate): wire extend:link into organization/place/work's fields=`
- `6e259cca` — `docs(annotate): document the keyMap companion config for fields=`

Important notes:
- In earlier conflict resolution, `bce2d5a2` became effectively empty after adapting `2ad80fbf`.
- `6e259cca` may also become empty if PR D already carries the richer `keyMap`/fallback docs wording.

### PR 2 — reconcile profile proper
Cherry-picks:
- `ee147034` — `feat: add the reconcile profile (OpenRefine Reconciliation Service API)`
- `eb622759` — `docs: add reconcile profile documentation, fix stale Custom-connector caveat`
- `d9752572` — `docs(reconcile): registers profile is now optional, not a hard dependency`
- `ff5fa3db` — `docs(reconcile): mention fuzzy matching, name variants, and Lucene pre-filtering`
- `d66e5b54` — `docs(reconcile): mention property/id query-condition matching`
- `6a4f1cee` — `Document production-hardening options for the reconcile profile`
- `93944b89` — `docs(reconcile): recommend ?version=0.2 for OpenRefine's data-extension UI`
- `66ece1b3` — `docs(reconcile): document computing a property from outside the entity`
- `c581036a` — `docs(reconcile): document the fixed preview - absolute links, property list, images`

## Recommended submission order
1. Draft PR A
2. Draft PR B
3. Draft PR C
4. Draft PR D
5. Draft PR E
6. PR 2 (reconcile profile)

## Compatibility / dependency conclusions from the research
- The field-mapping change is backward-compatible at the authority-response level if connectors provide normalized `id`, `label`, and optional `extend:*` properties.
- It is **not** backward-compatible with an older frontend event payload that only exposes `detail.ref`; the late field-mapping stack presupposes `ev.detail.properties`.
- Leaving `fields` unset preserves older behavior.
- Existing saved annotations keep working because later commits add `keyMap`, `@ref` occurrence matching, and fallback handling.

## Files that matter most
- `profiles/annotate/resources/scripts/annotations/annotations.js`
- `profiles/annotate/modules/annotations/tei-annotation-config.tpl.xqm`
- `profiles/annotate/templates/pages/annotate-tei.html`
- `profiles/annotate/templates/pages/annotate.html`
- `profiles/annotate/resources/css/annotate.css`
- `profiles/annotate/templates/annotation-blocks.html`
- `profiles/forms/config.json`
- `profiles/docs/data/doc/annotations.xml`
- `profiles/docs/data/doc/reconcile.xml`
- `profiles/annotate/config.json`

## Previously observed mixed-commit trouble spots
- `9f1181de` mixes UI fixes with person/reconcile/id-prefix behavior.
- `7e7376c6` assumes later field-mapping context in `annotate-tei.html`.
- `0b746ad4` mixes `annotations.xml` and `reconcile.xml`.
- `0b754d0a` mixes `annotations.xml`, `reconcile.xml`, and screenshot assets.
- `2ad80fbf` / `bce2d5a2` were authored on top of later field-mapping state and may need manual adaptation.

## Validation / blocker notes from the prior session
- `npm ci` failed because Cypress download was blocked.
- `npm ci --ignore-scripts` worked well enough to run `npm run build`.
- `npm run build` succeeded, but rewrote `profiles/theme-base10/doc/README.md`; revert that generated file before committing.
- secret scanning passed on the changed files.
- remote push / PR creation failed with HTTP 403, so no remote draft PRs were created.

## Important review findings to preserve when reconstructing branches
A later validation run flagged regressions that should be fixed before finalizing annotate-side PRs:
- do not reintroduce `innerHTML` XSS in `profiles/annotate/resources/scripts/annotations/annotations.js`
- keep the `string()` atomization fix in commit-save properties in `profiles/annotate/templates/pages/annotate.html`
- keep the `<option data-i18n="...">` fix for status dropdown labels
- keep `force=""` on the relevant `fx-refresh` calls for panel closing reliability
- keep `.authority-info` as a class selector in CSS, not `#authority-info`
- keep the reliable annotate-link doctype computation in `profiles/annotate/templates/annotation-blocks.html`

## Suggested first prompt for Claude Code at home

"In `mpilhlt/tei-publisher-jinks`, reconstruct the PR split for `feature/reconcile` using the handoff file `CLAUDE-CODE-HANDOFF-feature-reconcile-pr-split.md`. Start by fetching full history and upstream `eeditiones/jinks`, verify the merge-base against main, then recreate Draft PRs A-E plus PR 2 as local branches. When cherry-picking mixed commits, preserve the scope notes in the handoff. Validate with `npm ci --ignore-scripts`, `npm run build`, revert generated `profiles/theme-base10/doc/README.md`, and avoid reintroducing the annotated XSS/string()/force-refresh/class-selector fixes called out in the handoff." 

## Minimal reconstruction checklist
1. Fetch full history and upstream refs.
2. Reconfirm merge-base with upstream/main or origin/main.
3. Recreate PR A from main.
4. Recreate PR B from main, splitting `9f1181de` manually.
5. Recreate PR C from main, splitting `0b746ad4` manually.
6. Recreate PR D late, likely on top of C for least pain.
7. Recreate PR E on top of D.
8. Recreate PR 2 from main.
9. Validate each branch.
10. Push and open draft PRs once credentials allow it.
