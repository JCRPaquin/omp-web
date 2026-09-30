# Candidate B v2: unified source root, product ownership plus feature cohesion

Status: reviewed and approved as a design proposal, not implemented. Read [the common guidelines](common-guidelines.md) and [review results](review-results.md) with this candidate.

Use case: current repo wants a consistently named source tree and task-oriented navigation without paying workspace/package cost. RECOMMENDED design direction, contingent on reviewer approval. One package.json, bun.lock and installable artifact. src/ becomes all production TypeScript, not just frontend. Group by product first, feature second; do not mix frontend/backend implementations of a feature into one feature root.

```text
src/
  cli/                     # omp-web.ts single dispatch; update, version
  fleet/
    main.ts                # startFleet composition root
    cli.ts                 # fleet command implementation
    edge/                  # FleetEdge, browser pipes/origins/replay, embedded-dist
    daemons/               # supervisor, connector, selectors/fanout, activity
    registry/              # registry persistence and title/discovery probes
    projects/              # project registration and managed-worktree operations/routes
    workspaces/            # single clone lifecycle + profiles/control routes
    transport/             # fleet callback enrollment/up/down/bulk
    auth/                  # browser auth/CSRF/trusted proxy
    settings/              # fleet settings persistence
    history/
      log-store.ts, stored-sessions.ts, wake-materialize.ts
      stats/               # StatsApp, routes/lib/config, __tests__/fixtures
  session/
    main.ts                # daemon boot/composition, not an importable service barrel
    agent/                 # SessionEntry, METHODS, SDK event integration, subagent mirror
    http/                  # authorization, commands/dedup, download jail
    transport/             # SSE delivery, callback client/control
    history/               # lineage tailing/materialization/flush/quiesce evidence
    settings/              # live session side effects
    collab/                # WS relay/host/session/CLI
  runtime/
    providers/             # bwrap/kubernetes executable adapters
    workspace/             # prepare, baseline, provider volume/store verification/resources
    image/                 # Dockerfile + helpers, owned by runtime
  web/
    main.tsx, App.tsx
    model/                 # one createStore, types, guarded ingress/reset coordinator
    transport/             # existing browser call/retry/auth mechanics
    features/
      chat/                # components, actions, tool rendering/grouping
      prompt/              # components, slash commands/history/images/autocomplete
      roster/              # sidebar/components, projects/workspaces/activity/unread/actions
      transcripts/         # current tx view/API/utils, tx.css (rename stylesheet only deliberately)
      settings/            # panel/model picker/theme/preferences/actions
      usage/               # modal/sidebar/provider usage mirror/helpers
      subagents/           # subagent strip/drill-down/actions
    ui/                    # genuinely reused Icon/dialog/view primitives, sprite engine
    text/                  # safe markdown/diff/clipboard/image utilities
    styles.css, styles/    # theme/global/feature style files keep cascade order
  contracts/               # portable wire/types/constants/pure validation, no host or stream imports
  transport/               # shared portable SSE and callback stream codecs
  network/                 # pure cross-product loopback classification
  platform/                # host file-lock, session-files archive verification, archive-id hashing
  sdk-settings/            # shared host-only settings model/coercion; not browser importable
  testing/                 # tempDir and cross-product *.testkit helpers, never production imports
tests/system/              # true cross-product process/wire/distribution scenarios
scripts/                   # dev/build/test/release/install and thin owner fixture invocation
docs/                      # docs site and frozen clone design ledgers stay put
index.html                 # Vite points script to src/web/main.tsx
```

All common constraints/test policies apply. Tests co-locate with implementation or owner __tests__, including existing heavy owner integration tests. No flattened src/services, src/classes or root unit-test mirror. Current root test/ becomes src/fleet/history/stats/__tests__. Tests genuinely exercising fleet+daemon/process products live tests/system. Helpers and fixtures stay with their owning suite unless multiple owners demonstrably use them.

## Imports and state ownership
- src/web imports portable contracts/transport/network only outside web. It may retain SDK type imports as today, but never SDK runtime imports.
- src/fleet may import runtime/platform/sdk-settings/contracts/transport/network; it never imports session implementation.
- src/session may import platform/sdk-settings/contracts/transport/network, not fleet implementation.
- src/runtime may import platform/contracts/transport/network, not fleet/session composition or browser.
- shared SDK settings presentation/coercion -> sdk-settings; session side effects and fleet persistence remain separate. loopback logic leaves session/config.
- feature actions import model/core.ts and transport API. model/core owns only types, the single store construction and store-local pure selectors; it imports neither feature actions nor transport. Transport imports core, not the coordinator. A guarded ingress/reset coordinator imports domain handlers; transport-dependent selectors such as hasLiveSession live outside core. Components import feature actions directly. The old src/state.ts compatibility path is removed; use a narrow model API only if needed, never an all-actions barrel. Coordinator remains the attach/reset and guard-order authority.
- Existing DOMPurify, CSS order, in-place streaming, localStorage, model-role parity, dedup/origin guards and lifecycle serialization remain unchanged.

## Concrete mappings
- server/index.ts -> src/session/main.ts. fleet/server.ts -> src/fleet/main.ts. cli/omp-web.ts -> src/cli/omp-web.ts.
- fleet/edge.ts -> src/fleet/edge/edge.ts, with route/project-command adapters separated, while pipe/replay/generation/origin bookkeeping stays under one FleetEdge owner.
- fleet/workspace-lifecycle.ts -> src/fleet/workspaces/lifecycle.ts, with evidence/provider helpers extracted but no competing owner.
- src/state.ts -> src/web/model/core.ts + guarded ingress/coordinator, only along observed existing store seams. src/store domain actions join corresponding features or web/transport.
- src/components/roster + src/fleet-ui + src/store/projects/roster -> src/web/features/roster, but files remain distinct cohesive modules.
- src/tx -> src/web/features/transcripts; frontend API client is not merged with backend stats routes. model-picker helpers -> web/features/settings.
- shared/protocol.ts -> src/contracts/protocol.ts initially. Splitting command/frame/settings subtypes is optional only after reference/cycle inspection, not prerequisite to relocation.
- runtime/export-sessions.ts -> src/platform/session-files.ts with its traversal bounds, structural JSONL checks, typed errors and tests unchanged. Atomically migrate daemon quiesce evidence, runtime verification/resources and fleet lifecycle consumers. Quiesce orchestration stays session-owned, verified deletion stays fleet-owned, and provider volume/store verification stays runtime-owned. Filesystem helpers never enter portable contracts.
- Split mixed shared modules per common v3: provider process helpers -> src/runtime/providers/identity.ts; archive computeExportId -> src/platform/archive-id.ts; stored materialization planner/emitter -> src/fleet/history/materialize-transfer.ts; portable request/record validation -> src/contracts. Callback stream codec functions -> src/transport/callback.ts, removing contracts-to-transport imports. Migrate every producer/consumer and relevant test before whole-module moves; no browser host imports or legacy aliases.

## Migration and cost
Cost medium: more entrypoint/build/dev/resource updates than A, but no workspace manifests/export maps or package publishing. Preserve root scripts and public commands while replacing their source targets. Keep dist/ and dist-bundle/ unchanged; index.html Vite script target changes. Update docs/architecture.md's import diagram to actual architecture.

Sequence: (1) separate current cross-product reusable loopback, SDK-settings and host session-files ownership, migrating every archive consumer and relevant tests together without changing evidence semantics, (2) relocate contracts/platform/testing and update all consumers, (3) move session/fleet/runtime/CLI roots atomically per product with source-target updates and packaged runtime checks, (4) move browser root/Vite entry, (5) collocate feature logic/components without changing state ownership, (6) move stats tests/fixtures and system scenarios with helper isolation. Behavior extractions and folder moves are separate commits. Every intermediate step runs the relevant checks and is usable; no old-path aliases remain after each step.

Use current tsgo/oxlint and reviewed import rules to disallow cross-product imports. Browser dependency graph/bundle inspection can supplement boundary review during migration but is not a permanent test asserting source strings. Do not add an architecture framework only to enforce folder names.

Strengths: source paths say session/fleet/web; contributors find a feature's components/actions/tests together; platform and contracts are visibly different; single-package workflow preserved. Weaknesses: larger initial move; feature coupling can reintroduce model cycles if core/coordinator boundaries are ignored; folders alone do not provide package-level exports enforcement. The deliberate global CSS organization is an exception to full feature colocation.
