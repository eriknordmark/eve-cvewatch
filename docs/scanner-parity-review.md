# Scanner parity and CI review

This follows PR #4's upstream-base selection fix. The standalone scanner was
compared with CVEWatch's scanner and its Go-version/fallback regression suite.
Service, UI, alerting, and reachability code are outside this repository's scope.

## Findings addressed

| Area | Previous failure | Change |
| --- | --- | --- |
| Fork build isolation | PR make commands could modify the scanner or baseline on the same runner | Separate PR build, baseline build, and trusted scan jobs; data-only handoff |
| Workflow version | Reusable workflow checked out the tools repository's default branch | Check out the defining workflow's exact repository/SHA |
| Workflow events | `workflow_run` has no `pull_request`; empty fields selected wrong/default input | Resolve the associated PR, reject stale/ambiguous runs, validate output and repository inputs |
| Go | `go1.x.y` could query an incorrect advisory set | Normalize toolchain semver; isolate cache keys; preserve reported binary versions |
| Package identity | Unscoped Alpine/binary names and scoped npm parsing could miss the right package | Container/release/source identity; preserve npm scopes and Go module paths |
| False suppression | Flat Alpine fix IDs suppressed findings regardless of installed revision/ecosystem | Remove the unsafe suppression; retain version-aware distro checks |
| Advisory integrity | Failed later pages could return partial success; retries did not reach swallowed errors | Retry at HTTP boundary and reject incomplete/schema-invalid/pagination cycles |
| Cache lifecycle | Whole date directories were deleted while other workers might use them | Stable cache, eight-hour per-entry expiry, atomic archive/report/result replacement |
| Kernel | Output labels such as `pr` were treated as source refs; disjoint intervals could erase earlier matches | Explicit source-pinned kernel metadata; test each numeric affected interval |
| Build failure | Bash `if` disabled errexit and allowed a stale build output | Explicit failure propagation and exact `sbom_info`; no arbitrary dist-file fallback |
| Baseline consistency | Image, metadata, and displayed commit could come from different master tips | Resolve base once; isolate worktree and use the same commit for image and metadata |
| Historical identity | Cached SBOM was relabeled using the newest commit in its month | Persist actual extracted commit; require matching image identity before reuse |
| Scan/upload failures | Old successful files could be uploaded after failure; failed uploads were deleted | Invalidate completed output on rescan, separate partial diagnostics, upload successful workers only, retain failed uploads, propagate nonzero status |
| Diff identity | Same display name hid unrelated packages; alias/provider changes appeared new | Compare package PURL identities and CVE/advisory aliases |
| Fallback | Upstream CI missed CVEWatch's conservative supplemental discovery | Port evidence-gated Alpine/exact-commit fallback and its regression suite |

## Validation and limits

The regression suite covers real local Git fetch/worktree operations, actual
shell orchestration with fake build/image/upload commands, malformed API pages,
rate limits/timeouts, cache expiry and migration, a synchronized concurrent
coverage-report write, failed archive refresh, kernel interval boundaries,
fixed/backported Alpine revisions, workflow input injection, and alias-aware
comparison. Builds remain arbitrary source execution within their isolated
runner. Their SBOM contents and ignore policy still require source review.

Bash syntax checks and `git diff --check` pass. Actionlint 1.7.12 does not yet
recognize GitHub.com's documented `job.workflow_repository`/`job.workflow_sha`
properties. With only those two property warnings excluded, workflow lint
passes. The implementation uses the official GitHub context definitions, not
an inferred caller ref. Full EVE image builds and live vulnerability-feed scans
have not been run for this change.
