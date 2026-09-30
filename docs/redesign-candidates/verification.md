# Runtime evidence for structural review

The initial design review made no repository source/config/test/docs changes. Its proposals, rubric and evidence are now persisted under docs/redesign-candidates/ as documentation, not applied refactors. Read-only scouts returned exact current module evidence; reviewed source takes precedence over stale architectural comments.

Observed executions:
- `bun cli/omp-web.ts --version`: exit 0, stdout `0.2.1`. Exercises actual installed-command source dispatcher/version branch.
- Isolated attempt to start actual `startFleet` with ephemeral loopback port, scratch state/config/workspace/stats paths: exit 1 before boot. Error: `Cannot find module '@oh-my-pi/pi-coding-agent/config/settings-ui' from '/mnt/sources/server/settings-model.ts'`. No dependency repair attempted; this is an environment prerequisite for full fleet/session/browser smoke, not a structure proposal defect. No claim that full fleet/daemon ran.
- Actual `createStatsApp` behind ephemeral `Bun.serve`: GET `/ctl/stats/health` returned 200 with ok=true, statsDb='missing', sessionsCount=0 and isolated absent data paths. Outside mount returned 404 through outer host. Server/app closed. Proves existing mountable read-only stats composition can run without fleet daemon boot.
- Shared `encodeSseEvent` + `parseSseUnits` executed with a frame and ping emitted through 3-byte stream chunks including multibyte UTF-8 payload. Parsed frame id='1024' and identical JSON; named ping had no id. Exit 0. Proves an actual reused portable transport seam, not a source-string assertion.
- Configured TypeScript LSP failed to start: `/usr/bin/env: 'node': No such file or directory`. Source evidence came from bounded import reads and scout investigation; no unavailable symbol results claimed.

No tests, typecheck, builds, lint, or browser/UI verification ran. This task is architectural investigation, not an implementation. No real provider/Kubernetes/proxy lifecycle evidence claimed.
