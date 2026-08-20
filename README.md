# myth-mobile-apps

MYTH Technologies mobile application products.

Generic monorepo: shared Dart packages (`packages/`) and mobile apps as Git
submodules (`apps/`) — each app remains an independent repository (own
history, CI, releases); this repository only stores a commit pointer to each
of them.

Dev policy: [`MYTH_DEV_POLICY.md`](MYTH_DEV_POLICY.md) (pointer to the canonical
`myth-operations` doc, see §12 for versioning & release). Changelog:
[`CHANGELOG.md`](CHANGELOG.md).

## Backlog & module placement — blocking rule

- Describes **a shared package itself** (its API, behavior, evolution) → backlog
  item, doc and code live **here** (`backlogs/`, `docs/`, `packages/<name>/`).
- Describes **how an app consumes/wires** a shared package, or an **app-specific
  feature** with no shared equivalent → backlog item, doc and code live in
  **that app's own repo**, never here.
- Reverse also holds: **no generic/reusable code in an app repo** (extract it to
  `packages/` instead); **no app-specific code in `packages/`** (colors, icons,
  business taxonomies, concrete backend endpoints — anything that varies per app
  stays local, even when several apps share the underlying logic: the package
  exposes parameters/callbacks, never a hardcoded value or backend contract).
- Example: an item about `myth_fuel_ev`'s own behavior → here; an item about its
  wiring inside VTC Copilot (imports, providers, screens that use it) →
  `vtc-copilot-mobile`.
- Enforced automatically: `policy-guard` (in each repo) only sees that repo's own
  `backlogs/` — an item placed in the wrong repo is invisible to the check and
  blocks the PR there with `[backlog-first]`. Full rule, with the backlog
  lifecycle it plugs into: `myth-operations/governance/MYTH_DEV_POLICY.md` §1.

## Shared packages (`packages/`)

> Full inventory, consumer matrix and the `git:` + `ref:` SHA dependency model:
> [`packages/README.md`](packages/README.md). Test architecture: [`TESTS.md`](TESTS.md).

- `myth_location_map` — base map + camera state
- `myth_core` — geometry (Haversine), errors/`Either`, `UseCase`, prefs
- `enseignes_carburant` — fuel-station brand icons (2 visual variants, one
  per app — see the package README)
- `myth_fuel_ev` — fuel/EV models (`ServiceStation`, `ChargingStation`),
  catalogs, ranking/filtering and the savings-analysis calculator, shared by
  VTC Copilot and Mon Plein Copilot. App-specific rendering (markers,
  colors, carousels) stays local to each app.
- `myth_field_logger` — field-test journal (`FieldTestLogger`): console +
  in-memory ring buffer + persistent local file, survives an offline kill,
  file rotation (`maxRetainedFiles`). App name/file prefix set once at
  bootstrap (`FieldTestLogger.instance..appName = '…'`).
- `myth_analytics` — generic buffer/throttle telemetry engine
  (`AnalyticsService`: batched POST to an app-supplied `eventsEndpoint`,
  error dedup/rate-limit), device/session runtime (`AnalyticsRuntime`) and
  navigation instrumentation (`AnalyticsNavObserver`, app-supplied
  route→event map). Event taxonomy and business context (tier, theme…)
  stay per-app — each app talks to its own backend (driverpro-backend for
  VTC Copilot; a separate backend for Mon Plein Copilot and siblings).
- `myth_itinerary` — pure route-replay math (`ReplayTrack`/`replayStateAt`:
  cursor-driven interpolation along a precomputed track) and orphan-trace
  recovery (`buildRecoveredTrace`: rebuilds a trip from a persisted GPS
  route + stops after an offline app kill). Returns generic shapes
  (`RecoveredTrace`, `TraceStop`) — each app maps them to its own
  activity/trip model.
- `myth_qa_lab` — in-app QA Lab shell: typed scenario catalog per module,
  sequential runner (progress + live logs), on-device user simulation
  (`QaUiDriver`), map hooks (`QaMapHook`), polished UI. Mounted behind each
  app's `kQaToolsEnabled` const gate → tree-shaken out of release binaries.
- `myth_test_kit` — shared TEST infrastructure (dev_dependency): pump
  harness, Riverpod container helpers, GWT wrappers, builders, JSON fixtures,
  fake clock/GPS tracks, custom matchers, stable `QaKeys`, golden policy.
  See `myth-operations/governance/testing/TEST_POLICY.md`.
- `lucide_icons` — vendored copy of the open-source Lucide icon set for
  Flutter (v0.257.0, lucide.dev). Generic icon library usable by any app;
  consume it through a `dependency_overrides` git entry pinned by SHA.

## Apps (`apps/`, submodules)

- `vtc-copilot-mobile`
- `monplein-copilot-mobile`
- `carburant-copilot-mobile`, `borne-recharge-copilot-mobile`,
  `zonevigilance-copilot-mobile` — repositories created, to be populated

## Cloning this repository

**If you have access to all apps**:
```bash
git clone --recurse-submodules https://github.com/mythtechno/myth-mobile-apps.git
```

**If you only have access to some apps** (the normal case for an external
collaborator or someone focused on a single app): `--recurse-submodules`
fails as a whole as soon as ONE submodule is inaccessible. Use instead:
```bash
git clone https://github.com/mythtechno/myth-mobile-apps.git
cd myth-mobile-apps
./scripts/setup-submodules.sh
```
This script initializes each submodule independently and **cleanly skips**
the ones you don't have access to (instead of blocking everything), with a
summary at the end.

## Access rights per submodule

Each submodule has its **own GitHub permissions, independent from
myth-mobile-apps** — granting access to this repository does NOT
automatically grant access to the apps it references. To add a collaborator
who should only see a subset of the apps:

1. Add them as a collaborator on **each app repository** individually
   (`Settings → Collaborators and teams`) — only the ones they need.
2. Also add them to `myth-mobile-apps` itself if they need to see the list
   of apps and the shared packages (`.gitmodules` lists all submodule URLs,
   but their content stays protected by their own permissions).
3. `./scripts/setup-submodules.sh` then fetches exactly what they are
   entitled to.

## App version tracking

The pinned commit of each submodule **is** the app version integrated by the
monorepo. Mechanism:

- **[APPS_VERSIONS.md](APPS_VERSIONS.md)** — generated manifest (app ·
  tracked branch · pinned commit · pubspec version). Do not edit by hand.
- **`./scripts/sync-apps.sh`** — moves each submodule to the head of its
  tracked branch (`.gitmodules`, `develop` everywhere) and regenerates the
  manifest; `--check` regenerates the manifest only.
- **CI `apps-sync-check`** — on every PR/push: each pointer must be
  reachable on its tracked branch (zero drift) and the manifest in sync.
- **CI `apps-bump`** (manual) — opens a bump PR targeting `develop`.

Flow: bump via PR on `develop` (staging); `main` (production) only moves
through a release, as per `myth-operations/governance/MYTH_DEV_POLICY.md`.
