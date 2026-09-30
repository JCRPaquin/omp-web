# Repository structure review rubric

Goal: improve ease of contribution without changing product or wire behavior. Score each criterion from 1 to 5. Weighted score = sum(weight * score / 5). Scores are design judgments, not measured contributor performance.

| Criterion | Weight | Questions and evidence |
|---|---:|---|
| Component ownership | 10 | Q1: product composition roots and major responsibilities have unambiguous owners; CLI, fleet, session daemon, browser, provider runtime, docs/tooling are accounted for. |
| Interaction and trust boundaries | 15 | Q2: allowed dependency direction is explicit; browser/runtime/SDK separation, transports, authoritative state and privileged operations stay intact. |
| Reuse placement | 10 | Q3: extract only demonstrated shared capabilities; distinguish portable contracts/transport from host-only helpers and test support; avoid generic shared dumping grounds. |
| Test placement and isolation | 10 | Q4: tests have one predictable placement policy, ownership, discovery and fixtures; heavy tests and global state stay isolated; no product imports of test support. |
| Valuable behavioral verification | 10 | Q5: cover plausible consumer-visible failures at boundaries, state transitions, security, persistence and distribution; avoid wiring/source-text/mock-echo tests. |
| Discoverability and contribution flow | 20 | Q6: newcomer can locate implementation and tests for a user-visible feature; names match products; shallow navigable paths, documentation map, one obvious command entrypoint. |
| Cohesion and modularity | 15 | Q7: concrete split/merge seams and ownership; preserve lifecycle/state authority; no interface/class for every file or gratuitous abstraction. |
| Migration and tooling practicality | 10 | Common practice: bounded migration, no unnecessary new release units; build/dev/Vite/CLI/runtime image/resource/test paths accounted for; incremental checkpoints, no legacy shims, synchronized SDK pins. |

Approval requires weighted score >= 80/100, every criterion >= 3/5, and no blocking defect. A conditional-use candidate may be approved only as an alternative for its stated use case, not asserted to be appropriate now. Reviewer must identify the condition explicitly.

Hard blockers: broken/bumped wire protocol just to reorganize; changed command/URL/file-format behavior; duplicate workspace lifecycle owner or frontend store; test/host-only/privileged imports reachable in browser; lost replay/dedup/origin/generation safeguards; settings policy split from parity without explicit owners; inaccurate claims that fleet is statically SDK-free; package/runtime assets breaking single installable CLI; missing source move/test/entrypoint cleanup plan.

Reviewer deliverable: version reviewed; eight scores and weighted total; approval or blocking issues; exact required changes; optional nonblocking comments. Evaluate candidly. Do not waive blockers to finish. Read only; do not run build/lint/tests/formatters. Review proposed design rather than implementation that has not occurred.
