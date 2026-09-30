# Candidate C v2: private workspace monorepo with explicit package boundaries

Status: reviewed and approved as a design proposal, not implemented. Read [the common guidelines](common-guidelines.md) and [review results](review-results.md) with this candidate.

Use case: multiple sustained owners or an actual second consumer of portable contracts/provider runtime needs package-level import boundaries. Not recommended for today's repo merely to tidy directories. One root orchestrator, one Bun lockfile, one synchronized SDK pin set, one public CLI version/release/artifact; internal workspaces private, not separately published. Product services are apps, not reusable packages.

```text
apps/
  cli/                     # private package: src/omp-web.ts, update/version
  fleet/                   # private package: src/main.ts, cli.ts,
    src/                   # edge/daemons/registry/projects/workspaces/transport/auth/settings/history
  session/                 # private package: src/main.ts, agent/http/transport/history/settings/collab
  web/                     # private package: index.html, vite.config,
    src/                   # model/transport/features/ui/text/styles, same ownership as B
packages/
  contracts/               # portable shapes/constants/validators; network-policy subpath
  sse/                     # portable SSE and callback stream codecs, explicit subpaths
  host-platform/           # file-lock, session-files traversal/JSONL checks, archive-id hashing
  sdk-settings/            # private host-only SDK-backed common model/coercion
  provider-runtime/        # private: src/providers, workspace, image/assets
  test-support/            # private dev-only tempDir/testkit, never shipped/imported by products
  ... no standalone stats package
tests/system/              # cross-app process/distribution scenarios
scripts/                   # root dev/build/test/install/release orchestration
docs/                      # existing docs site/design ledgers retained
package.json               # root workspaces, release version, scripts + exact SDK pin source
bun.lock                   # one graph, no package-specific lockfiles
```

Within apps, use B's product/feature ownership, common cohesive splits and common test policy. Fleet history/stats stays inside apps/fleet/src/history/stats with its __tests__/fixtures: its log-store/config coupling is not hidden behind a pretend independent package. Provider runtime is a library/tool package with CLI executables and image assets; there is no extra service or network hop. The small loopback helper is explicitly contracts/network-policy, not a separate one-file package.

## Allowed package graph and exports
- web -> contracts (including network-policy subpath) and sse only among packages; no host-platform, provider-runtime, sdk-settings or session/fleet imports. Runtime SDK imports remain prohibited; SDK type coupling in contracts stays declared and pinned.
- session -> contracts/sse/host-platform/sdk-settings. Fleet -> same plus provider-runtime. Provider runtime -> contracts/host-platform and only needed sse, never apps.
- cli -> explicitly exported fleet CLI and session boot entrypoints plus its updater. Preserve command-selected dynamic imports; no eager barrel that boots a daemon when importing the package.
- Contracts contains portable types/constants/validators and contracts/network-policy only; SDK imports are type-only and pinned. SSE/callback stream codecs live in sse and import contracts; contracts never imports sse, host modules or apps, preventing a contracts↔sse cycle.
- SDK-settings exports model/coercion for fleet/session, never session-only effects. Host-platform has narrow intentional file-lock, session-files and archive-id subpath exports, not a utilities barrel. Move runtime/export-sessions unchanged into host-platform/session-files, migrate quiesce evidence/runtime verification/fleet limits consumers and tests atomically. Keep quiesce coordination session-owned and lifecycle/deletion authority fleet-owned. Provider process helpers -> provider-runtime/providers/identity; stored-materialization planner/emitter -> fleet/history/materialize-transfer, wire validation -> contracts. Follow common v3's full consumer mapping.
- App imports use workspace exports, never ../../ across package internals. Package exports express availability, not authorization by consumer: workspace dependency declarations + lint/import discipline still enforce that web cannot add host-only dependencies. No claim that export maps by themselves enforce full architecture.
- test-support is development-only. Owner-specific mocks/fixtures stay local. Root tests depend explicitly on apps' public composition/testing seams; do not export every private module to make tests work.

## Package manifests and SDK pins
One committed bun.lock and a root source-of-truth for the synchronized exact SDK set. Each importing workspace declares its real dependencies, all @oh-my-pi dependencies exactly 18.2.6 as currently. Add a release/preflight consistency validation for manifests because a workspace-level mismatch is consumer-visible runtime skew. Do not rely on undeclared root dependencies or introduce local dependency patches. Internal package export maps point to source for development; the shipped root build bundles required internal modules and externalizes only current pinned upstream SDK dependencies, not private workspace package names.
- Root tsconfig includes every apps/** and packages/** source and owner test plus scripts/system tests. Typecheck resolves workspace source exports directly, with no declaration-build prerequisite. Scoped package checks may supplement, never replace, the complete root tsgo check. Internal source export resolution must work in Bun development and build, not merely editor aliases.

## Distribution and assets
Root build owns one dist-bundle/cli.js, one version stamp, embedded web output and provider/image resources. Every shipped provider executable also bundles all its private workspace imports; no CLI or provider relies on installed private package names. Retain bwrap source-versus-installed daemon launch selection and env overrides (currently runtime/bwrap-args.ts:409-453), updating source targets only. Resource resolution distinguishes repository development roots from installed dist-bundle roots. Root packed metadata declares the exact upstream external dependencies and contains no unresolved workspace/private package dependencies. Root dev starts same products; Vite root becomes apps/web; no independent releases.
- Container recipe explicitly changes install order: copy root manifest/lockfile and ALL workspace manifests preserving relative paths before frozen install, then copy the complete session/provider/contracts/host-platform/sdk-settings source and asset closure, not merely two relocated apps. Runtime stage retains required workspace sources, manifests, node_modules links/targets, pinned external dependencies and executable/image assets (or bundles internal code with equivalent complete pinned SDK closure). Verify a fresh image can execute session and provider entrypoints without the checkout or unavailable private npm packages. Image layout changes never alter workspace volume paths or protocol.

## Tests and migration
Tests beside package/app implementation, integration inside owning package __tests__, cross-app tests at tests/system. Stats suite moves to apps/fleet/src/history/stats/__tests__ including generator/fixtures; state harness owns isolated globals/timers. Each workspace may offer scoped check commands, but root bun run test/check:types/build remains contributor/release interface, with physical-core worker policy preserved and no duplicated execution.

First implement/review module separations resembling B, then package actual boundaries one at a time. Update every dependency, import, script, TS/lint path, asset lookup, Vite/build generation and onboarding probe, remove old source paths and shims. Full root typecheck catches cross-package calls; targeted tests, actual runtime/browser smoke and offline packed install prove resolution/asset behavior. Do not combine package extraction with new public APIs or rewrite business logic.

Strengths: strongest explicit API/dependency map; scoped owner checks; actual reusable boundaries can grow independently without imports into app guts. Weaknesses: highest migration and maintenance cost, manifests/exports/type configuration and bundle graph complexity, SDK pin duplication/validation, lower newcomer simplicity. The single network helper remains a portable subpath rather than adding a one-file package.

Approval should be conditional on demonstrated ownership/second-consumer need and working packed-artifact resolution, not treated as the default contribution improvement.
