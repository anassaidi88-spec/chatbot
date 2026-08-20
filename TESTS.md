# Test architecture — MYTH mobile

The unified test architecture is **delivered** (taxonomy everywhere, empty
allowlists, blocking R4 test-impact gate, `myth_test_kit` + `myth_qa_lab`, QA Lab
pilot `home_map` as the CI reference). This document records the **real** structure
so nobody re-invents a conflicting one.

## Where tests live

Tests live **per package** and **per app** — there is intentionally **no root-level
`test/`** in the monorepo. Each submodule app is a standalone repo whose CI runs its
own suite; each `packages/<pkg>` owns its `test/`.

## Taxonomy (real, per app — do not reshuffle)

Category folders under `apps/<app>/test/` describe the **nature** of each test. They
are richer than a generic `unit/integration` split and are matched by the QA
orchestrator + the R4 gate.

- **`vtc-copilot-mobile/test/`** — `community/`, `core/`, `features/`, `functional/`,
  `perf/`, `profile/`, `quality/`, `security/`, `support/`, `unit/`, `visual/`.
- **`monplein-copilot-mobile/test/`** — `functional/`, `quality/`, `security/`,
  `support/`, `unit/`, `widget/`.
- **`packages/<pkg>/test/`** — pure unit tests for the package's logic (e.g.
  `myth_qa_lab`, `myth_test_kit`, `myth_fuel_ev`, `enseignes_carburant`).

| Category | Scope |
|---|---|
| `unit` | pure classes, services, utils |
| `functional` | isolated widgets/modules |
| `features` | feature-slice behaviour |
| `widget` | widget rendering (monplein) |
| `visual` | golden / render checks |
| `perf` | performance guards |
| `security` | access / gating (client-side display gates) |
| `quality` | lint/analysis/taxonomy meta-checks |
| `support` | notifications/storage/support glue |
| `community` | community signal engine (vtc) |
| `profile` / `core` | profile & app-core suites (vtc) |

New categories are added **only when there are real tests to put in them** — never
create empty folders. Align monplein's set toward vtc's at need, not preemptively.

## Baselines (non-regression)

- `vtc` ≈ **857** tests · `monplein` ≈ **57** tests.
- Packages keep their own suites (e.g. `myth_audio` **35** cases, plugin-free via
  seams + `fake_async`; `myth_road_safety` **68** cases including the exact
  RAD-MSG-001 message templates — any wording change MUST break a test;
  `myth_support_chat` **61** cases so far, lots 1-6 of [[SUPPORT-SHARED-001]]).
- Any product rule that changes **must** break a test. Entitlement/gating matrices
  are anti-regression suites (e.g. `test/features/intelligence/ai_entitlements_test.dart`).

## Gates & CI (already in place — do not re-create)

- `.github/workflows/policy-guard.yml` — blocking MYTH_DEV_POLICY gate: **R1**
  backlog-first, **R2** done-integrity, **R3** readme_docs, **R4** test-impact
  (label `policy:no-test-impact` to waive with justification).
- `.github/workflows/apps-sync-check.yml` — submodule pointers vs `APPS_VERSIONS.md`.
- `.github/workflows/mobile-ci-reusable.yml` — reusable mobile pipeline.
- `nightly-qa` — stays inert until merged to `main` (release); activate at the next
  release.

## Shared test tooling

- **`myth_test_kit`** — shared fixtures/helpers for unit & integration tests.
- **`myth_qa_lab`** — in-app QA tools; the `home_map` pilot is the CI reference chain
  (migration + functional + security/perf + QA Lab + journey).

## Rules

- Run `flutter test` per touched app/package; keep `flutter analyze` clean
  (`unused_element`/`unused_import` is the authoritative dead-code signal, not grep).
- Code/comments/docs in **English** (§11); backlogs/issues in **French**.
- A package's tests live in its `packages/<pkg>/test/` (§1) — never in an app repo.

See also: [`packages/README.md`](packages/README.md), [`MYTH_DEV_POLICY.md`](MYTH_DEV_POLICY.md).
