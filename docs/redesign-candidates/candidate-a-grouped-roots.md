# Candidate A v2: grouped product roots, single package

Status: reviewed and approved as a design proposal, not implemented. Read [the common guidelines](common-guidelines.md) and [review results](review-results.md) with this candidate.

Use case: current small-team contribution workflow; improve local navigation with low migration risk. Keep current top-level product names and source entrypoints. Unlike the other candidates, no entire source-root move and no package/workspace boundaries.

```text
cli/                       # omp-web.ts, update.ts, version.ts unchanged
server/                    # existing name retained for migration stability
  index.ts                 # composition/boot entrypoint unchanged
  session/                 # session-entry, methods, SDK integration
  http/                    # auth, command admission, download
  transport/               # sse-delivery, fleet-callback, daemon-control
  history/                 # log-tailer, session-materialize, writer-flush, quiesce evidence
  settings/                # session effects; shared presentation is elsewhere
  collab/                  # host, relay, session, CLI
fleet/
  server.ts                # composition root unchanged
  cli.ts                   # fleet CLI implementation unchanged
  edge/                    # FleetEdge, browser pipes, public projections, command adapters
  daemons/                 # connector, supervisor, spawn parsing, activity aggregation
  registry/                # durable Registry, discovery, title probes
  workspaces/              # projects/worktrees; clone lifecycle, profile/control adapters
  transport/               # callback registry and trust-specific adapters
  history/                 # log store, stored sessions, wake materialization
    stats/                 # existing stats API + __tests__/ stats suite
  auth/                    # browser sessions, gate, trusted proxies
  settings/                # fleet persistence adapter
runtime/                   # existing provider runtime root retained
  providers/               # executable adapters plus provider process-identity helper
  workspace/               # prepare, baseline, provider volume/store verification/resources
  image/                   # existing assets
shared/
  contracts/               # portable protocol shapes/constants/validators, not host helpers
  transport/               # portable SSE and callback stream codecs
  network/                 # portable numeric-host loopback classification
  host/                    # file-lock, session-files verification, archive-id hashing
  sdk-settings/            # SDK-backed shared model/coercion, NEVER browser
src/                       # existing Vite frontend root retained
  index.tsx, App.tsx
  state.ts                 # single-store facade/guarded ingress
  store/                   # current domain actions
  components/              # existing component groups retained
  transcripts/             # rename tx, API/util/view remain together
  prompt/, chat/, text/, usage/, prefs/, sprites/, fleet-ui/
  styles.css, styles/      # current CSS cascade retained
  models/                  # model-selection helpers removed from usage/
testing/                   # shared tempDir support, reusable *.testkit.ts
  contracts/               # only truly multi-product contract harnesses if needed
tests/system/              # cross-product executable scenarios only
scripts/                   # runner/build/release/dev; fixture generator delegates to owner
  ...
docs/                      # existing documentation site/design ledger retained
```

All tests follow [the common guidelines](common-guidelines.md). Tests adjacent to files move with owners. Current test/ stats suite moves to fleet/history/stats/__tests__ and generated fixtures beside it; no new global tests/unit hierarchy. Existing fleet integration and collaboration scenarios that actually span products move to tests/system; narrower owner tests stay owner-co-located. Component/domain split is intentionally retained to avoid making every UI move at once.

## Concrete contribution routes
- FleetEdge -> fleet/edge/edge.ts. Existing server.ts owns mounting/wiring only.
- Registry -> fleet/registry/registry.ts.
- SpawnSupervisor/DaemonConnector -> fleet/daemons/supervisor.ts and connector.ts.
- WorkspaceLifecycle -> fleet/workspaces/clone-lifecycle.ts, still one authority.
- FleetLogStore -> fleet/history/log-store.ts; StatsApp -> fleet/history/stats/index.ts.
- SettingsModel building/coercion -> shared/sdk-settings/model.ts; session effects -> server/settings/effects.ts; fleet persistence -> fleet/settings/settings.ts.
- Commands/frames -> shared/contracts/protocol.ts; SSE ring -> shared/transport/sse.ts.
- One live AgentSession wiring -> server/session/; one browser store still src/state.ts.
- Split current mixed shared modules before moves: provider protocol filesystem/process helpers -> runtime/providers/identity.ts; manifest computeExportId -> shared/host/archive-id.ts; wake-materialize planner/emitter -> fleet/history/materialize-transfer.ts, retaining request/record types and validators under shared/contracts. Callback SSE/NDJSON codec functions -> shared/transport/callback.ts while contract shapes/constants/validators remain portable. runtime/export-sessions -> shared/host/session-files.ts for the demonstrated daemon/runtime/fleet archive reuse. Follow common v3's exact producer/consumer and test migration, no aliases or duplicated validation.

## Modularity decisions
Apply common splits only when they shrink a composition root or remove demonstrated cross-product imports. Do not split large lifecycle methods merely to equalize file sizes. registry/discovery remain sibling modules, not a merged class. Existing collab CLI script target must point to new server/collab/cli.ts. Avoid creating index.ts barrels in every folder.

## Migration and costs
Cost low to medium: many import updates, but existing server/index.ts, fleet/server.ts, fleet/cli.ts, cli/omp-web.ts, src/index.tsx and Vite root remain. Structural changes still affect build embedding (fleet/embedded-dist becomes fleet/edge/embedded-dist.ts), provider relative resources, fixture imports and runtime/workspace scripts, so common verification remains required. Keep embedded-dist in fleet/edge because the edge owns UI serving and build script generates/restores it.

Steps: fix contributor map first; split mixed contract/host modules and extract real shared loopback/settings/session-files seams, atomically migrating each capability's producers/consumers and tests; group one backend subsystem per atomic move; relocate stats tests with fixtures/generator; regroup session files; rename frontend tx and relocate model helpers last. Preserve runtime volume layouts and artifact output paths, not accidental source-relative lookups.

Strengths: little toolchain disruption; reviewable file moves; easy product ownership. Weaknesses: ambiguous server/ and src/ names remain; frontend components and logic still require two predictable locations; shared/ remains mixed but explicitly separates portable versus privileged subtrees. Dependency enforcement comes from reviewed import rules and existing lint/type tools, not package exports.

Best fit now if the priority is lowest-risk contribution improvements over consistent source-root vocabulary.
