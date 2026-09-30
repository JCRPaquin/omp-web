# Repository source structure review

## Recommendation
Candidate B is the preferred long-term contribution-first structure: one source root, explicitly named cli/fleet/session/runtime/web products, frontend feature colocation, and portable contracts separated from host utilities. Keep a single package/lock/release. Candidate A is the lower-risk migration alternative and offers useful initial extractions without moving entrypoints. Candidate C is approved only for sustained separate ownership or demonstrated second consumers needing workspace exports, not for directory neatness.

## Current architecture
- cli/omp-web.ts is the single installed command dispatcher. Most fleet command handling is fleet/cli.ts; session selection dynamically imports server/index.ts, which boots on evaluation. Preserve deferred imports/stdout contract.
- fleet is the browser/control server, persistent roster/process supervisor, privileged clone lifecycle owner and durable transcript mirror. Direct daemon connector/supervisor and outbound clone callback transport have distinct authorities.
- server is the single live SDK AgentSession daemon, with SDK/session/dialog/subagent/HTTP/SSE/collaboration integration and durable session logs.
- runtime has provider process execution, real adapters at runtime/providers, workspace preparation/baseline and store verification, plus runtime image/resources. Some session-tree verification here is actually shared with the daemon and fleet.
- src is the Solid browser. Live shared state uses one createStore and extracted domain actions; state.ts still coordinates transport/session/frame ingress. Historical analysis at src/tx has its own local signals/resources and typed HTTP client. Do not force those local view resources into new global stores.
- shared mixes portable session/stats/provider/archive/callback contracts and transport with host-backed locks, filesystem materialization, process identity, hashing and test-only helpers.
- scripts owns dev/test/build/install/release workflows; docs owns user documentation and approved design ledgers. The package ships one CLI plus provider/image/UI assets in dist-bundle.

Browser -> POST/SSE fleet edge -> direct session daemon -> in-process SDK. Managed clone daemons initiate authenticated callback up/down/bulk channels to fleet. CLI -> fleet control routes or selected session entry. Fleet lifecycle -> provider-runtime operations/executables. Historical browser client -> fleet stats/stored read APIs -> stats DB/session files or mirrored logs, without waking compute.

## Contribution friction
1. Ambiguous server/ versus fleet/server.ts and src/ meaning browser-only.
2. Browser features split among components/, store/, helper feature dirs and styles/.
3. Large composition/coordinator modules: server/index, fleet/server, edge, workspace-lifecycle and src/state. Split adapter/helpers, not invariants across competing owners.
4. Existing cross-product reuse lives under owners: settings model/coercion and loopback in server; archive planning/verification in runtime.
5. shared is not a portable-only leaf. Whole-file moves into contracts would import host code; actual split required.
6. Root test/ is specifically fleet stats tests, while other suites are owner-colocated. Tests and fixtures should become discoverable beside owners.
7. Architecture docs' narrow import claims do not match source: fleet statically imports SDK settings values and session implementation, plus runtime utilities. Zero agent state is not zero SDK runtime dependency.

## Alternatives
Full reviewed trees, concrete mappings and migration details:
- [Candidate A v2: grouped product roots](candidate-a-grouped-roots.md). Keep cli/server/fleet/runtime/src roots and existing composition entrypoints; group backend folders, stratify shared, rename tx to transcripts, keep existing frontend domain/component convention.
- [Candidate B v2: unified product/feature source](candidate-b-unified-source.md). src/{cli,fleet,session,runtime,web,contracts,transport,network,platform,sdk-settings,testing}. Web features group components/actions/logic; model core has one store, separate ingress/reset coordinator; preserve global CSS import ordering.
- [Candidate C v2: private workspace monorepo](candidate-c-workspaces.md). apps/{cli,fleet,session,web}; packages/{contracts,sse,host-platform,sdk-settings,provider-runtime,test-support}. Private source export maps and declared dependency graph; one root lock/version/build/release. Network policy is a contracts subpath, not a tiny extra package. Stats remains fleet-owned.

## Reuse and split/merge decisions
- Common settings presentation/coercion becomes host-only sdk-settings; daemon live side effects and fleet persist-only behavior remain distinct adapters. Wire SettingsModel remains portable.
- Loopback classifier becomes portable network policy, not an import from session config.
- runtime/export-sessions capability becomes host session-files shared by daemon quiesce, runtime verification and fleet lifecycle. Preserve traversal bounds, JSONL checks and typed errors.
- shared/provider-protocol host /proc/pidfile helpers become provider-owned identity; pure protocol remains contracts.
- archive export-id hashing becomes host archive-id; portable manifest/path validation remains contracts.
- wake materialization planner/emitter becomes fleet/history/materialize-transfer; receiver remains daemon-owned; request/record types/validation remain contracts.
- callback stream codecs become transport, importing contracts without reverse dependency.
- FleetEdge pipe/replay/origin/generation bookkeeping stays one owner; feature command adapters and public projections are separable.
- WorkspaceLifecycle remains the one persisted/serializing create/ensure/stop/delete authority. Extract evidence/provider adapters only.
- Frontend one-store core imports neither feature actions nor transport; transport and feature actions use core; coordinator invokes handlers and owns guards/resets. Components import actions directly, not a cyclic all-actions facade.
- Group roster UI/actions/helpers into roster; historical tx into transcripts; model-picker helpers belong to settings/model selection, not usage accounting.
- Keep stats and stored history neighboring fleet history modules because they already reuse log-store/HTTP/JSONL internals, not an invented independently released stats package. Do not merge the distinct persistence authorities of live SDK logs, mirrored logs and stats.db.

## Tests
Unit and owner-scoped integration tests co-locate as *.test.ts or owner __tests__ for multi-file suites. Heavy tests do not automatically belong at root. Root tests/system is only cross-product executable scenarios. Move current test/ stats suite and its fixtures/generator into fleet history/stats/__tests__. Owner helpers remain local *.testkit.ts; shared tempDir goes into testing/test-support and each importing test retains top-level afterAll(cleanupTempDirs). Script-based release gates retain public scripts commands even when invoking relocated scenario support.

Split large state suite by transport/replay, streaming/history, attach/reset and fleet/auth behaviors only with singleton/global/timer isolation. Do not create filename/source text/wiring assertions to validate the move.

Valuable consumer-visible coverage to preserve/add where absent: transport chunking/proto/backpressure/replay and same-process at-most-once/restart refusal; cross-browser unicast; stale attach/session frames and pending-call reset; durable lock/registry/log ack boundaries and byte-identical wake; failed deletion preserving source/history and generation stop proof; download/symlink/credential/CSRF/trusted-proxy/callback/sandbox/Git guards; actual browser roster/chat/transcripts/settings/fleet-required behavior; offline packed install/serve/spawn/update and provider/image resource lookup. Real provider/proxy tests are prerequisite-gated, never require a live model in normal suite.

## Reviewer rubric and iterations
Rubric weights: component ownership 10; interactions/trust 15; reuse 10; test placement/isolation 10; behavioral verification 10; discoverability/contribution flow 20; cohesion/modularity 15; migration/tooling 10. Five-point scores, >=80 total, every score >=3, no unresolved blocker. Scores are qualitative design judgments.

- A: 75 -> 82/100, approved after explicit host/protocol split and consumer/test mapping.
- B: 85 -> 90/100, approved after shared host session-files extraction and cycle-free frontend core/coordinator clarification.
- C: 78 -> 83/100, approved for its conditional use case after shared archive placement and concrete packaging/container/root-typecheck plan.

[Full scores and reviewer findings](review-results.md). [Rubric](review-rubric.md). [Common constraints](common-guidelines.md).

## Migration
First clarify contributor task-to-path map and dependency diagram. Extract demonstrated shared capabilities with all callers/tests in each atomic change. Then group/move products independently, keeping extraction and relocation in separate reviewable commits. Move UI feature ownership only after core/coordinator boundaries are explicit. Move stats tests with fixture generator and paths, and multi-product scenarios without duplicate discovery. Remove old paths and aliases.

Update package scripts, CLI deferred targets, dev spawn templates, Vite/index.html, type/lint/format paths, build embedded-dist generation/finally restoration, provider/image/container roots, version/package lookup, benchmark/fixture/onboarding/release paths and docs links. Preserve URL/CLI/wire/durable data/volume/artifact contracts, one SDK pin set. No protocol bump just for source relocation.

Each implemented step needs targeted owner tests + root types, full suite for shared/daemon/edge changes, actual runtime/browser smoke and offline distribution gate whenever entrypoints/assets/build move. Folder/package exports do not alone enforce trust boundaries; review/lint dependency rules still matter.

## Observed verification and limits
CLI source version branch returned 0.2.1. Actual mounted stats HTTP health returned 200 with explicit missing data; outside route 404. Actual SSE frame/ping parsed correctly across three-byte UTF-8 stream chunks. Full fleet boot attempt blocked at SDK module import: @oh-my-pi/pi-coding-agent/config/settings-ui missing from server/settings-model.ts. LSP also unavailable because node is missing. No dependency/environment repair, tests/build/typecheck or UI proof claimed. [Runtime evidence](verification.md).

These files document reviewed proposals, not an implemented refactor. Only documentation under docs/redesign-candidates/ was added when persisting this review.
