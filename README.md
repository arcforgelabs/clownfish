<img width="1584" height="672" alt="clownfish_banner" src="https://github.com/user-attachments/assets/6b2a0d0f-aca8-47e5-8a1f-eb266c760646" />

# 🐠 Clownfish

Clownfish is a conservative OpenClaw maintainer tool for one-cluster issue and PR cleanup.

It takes a curated GitHub issue/PR cluster, asks a Codex worker to classify the items, and applies only narrow, auditable cleanup actions when the evidence is strong. This compliments the pre-pass work on [clawsweeper](https://github.com/openclaw/clawsweeper) and act as the second-pass intent based cluster resolution.

Allowed automated close reasons:

- duplicate of a clear canonical thread
- superseded by a clear canonical thread
- fixed by a specific candidate fix

Manual backlog-cleanup jobs may also use
[`instructions/low-signal-prs.md`](instructions/low-signal-prs.md) for
drive-by PRs that are clearly blank-template, docs-only discoverability churn,
test-only coverage spam, refactor-only noise, third-party capabilities that
belong on ClawHub, risky unapproved infra, or dirty branches. This policy is
opt-in per job and should return `needs_human` for plausible bug fixes or
anything with active maintainer signal.

Everything else stays open or is escalated for maintainer review.

Security-sensitive reports are deliberately out of scope. Clownfish
routes those refs to central OpenClaw security handling and keeps processing
unrelated ordinary bugs, provider gaps, and duplicate cleanup in the same
cluster. It follows OpenClaw `SECURITY.md`: trusted-operator exec behavior,
provider gaps, feature gaps, and hardening-only parity drift are not treated as
vulnerabilities unless there is a real trust-boundary bypass.

Explicit `security_signal_refs` remain quarantined during planning, close and
merge application, fix execution, and post-flight closeout, even if the live
item has no security label or structured marker. `security_override_refs` can
clear that imported signal; they do not bypass live mutation guards for security
labels or structured markers, or an existing plan's security quarantine.
GitHub URLs in these lists apply only to the job's repository. Live title, body,
and comment prose do not independently expand the deterministic detector.

## Status

Clownfish is intentionally smaller than ClawSweeper. ClawSweeper scans the whole OpenClaw backlog on a cadence; Clownfish handles targeted clusters that were already grouped by a human, gitcrawl, or another dedupe tool.

Cluster discovery currently comes from [openclaw/gitcrawl](https://github.com/openclaw/gitcrawl).

<img width="3582" height="2160" alt="image" src="https://github.com/user-attachments/assets/20b816cc-72ab-479e-bc18-84f5b2b53745" />

The default workflow is proposal-first. It does not comment or close unless a job is explicitly promoted and the deterministic applicator confirms live GitHub state has not changed.

## Dashboard

Last dashboard update: Jun 16, 2026, 09:45 UTC

<!-- projectclownfish-dashboard:start -->
No results yet.
<!-- projectclownfish-dashboard:end -->

## How It Works

For a maintainer-facing architecture map of the automation lanes, see
[`docs/INTERNAL_FEATURES.md`](docs/INTERNAL_FEATURES.md).

For the ClawSweeper feedback loop that updates existing generated PRs, see
[`docs/auto-update-prs.md`](docs/auto-update-prs.md).

That loop is marker-driven. ClawSweeper comments use hidden
`clawsweeper-verdict:*` markers, and only actionable PR feedback includes
`clawsweeper-action:fix-required`. Clownfish skips stale head SHAs and caps
automatic repairs at five per PR and one per PR head SHA.

Maintainers can opt an existing Clownfish PR into the bounded merge loop with
`/clownfish automerge`. That adds `clownfish:automerge`, dispatches
ClawSweeper for the current head, lets Clownfish repair trusted
`needs-changes` findings for up to five rounds, and merges only after a trusted
pass verdict for the exact current head plus green checks, clean mergeability,
and explicit `CLOWNFISH_ALLOW_MERGE=1` and `CLOWNFISH_ALLOW_AUTOMERGE=1`
gates.

ClawSweeper commit findings have a separate intake lane. A
`clawsweeper_commit_finding` dispatch fetches the latest markdown commit report,
writes an audit record under `results/commit-findings/`, and only sends the
finding into the PR executor when the issue is narrow, non-security, and still
worth repairing on latest `main`.

Each cluster job:

1. Starts from one markdown job file under `jobs/`.
2. Hydrates the listed issue/PR refs and first-hop linked refs.
3. Builds a cluster plan and fix artifact for autonomous jobs.
4. Runs Codex with repo-local policy prompts and JSON output schema in a read-only sandbox.
5. Writes structured run artifacts under `.projectclownfish/runs/`.
6. Reviews the worker artifact with deterministic safety checks.
7. Executes credited fix artifacts through `scripts/execute-fix-artifact.mjs` when the fix gate is open: repair a maintainer-editable contributor branch first, otherwise raise a narrow replacement PR, add non-bot source PR authors as replacement co-authors, and close the uneditable source PR after the replacement push succeeds.
8. Applies guarded close/comment and explicit merge actions through `scripts/apply-result.mjs`.
9. Publishes a sanitized result ledger back to this repo under `results/`, `jobs/openclaw/closed/`, `apply-report.json`, and this README dashboard.

Codex does not receive a GitHub token during classification. The runner preflights GitHub state before model execution, then Codex receives those artifacts and returns JSON only. When a reviewed fix artifact is executed, Codex gets a temporary target checkout without GitHub credentials; the deterministic executor owns commit, push, PR creation, and source-PR closeout using `CLOWNFISH_GH_TOKEN`. Commit author metadata defaults to `projectclownfish` and can be overridden with `CLOWNFISH_GIT_USER_NAME` and `CLOWNFISH_GIT_USER_EMAIL`; this is separate from the GitHub token used to push. The applicator re-fetches the target item, checks `updated_at`, blocks unsafe closeouts, writes idempotent close comments, closes supported duplicate/superseded/fixed-by-candidate actions, and can squash-merge explicitly allowed clean PR actions.

Merge is deliberately harder than closeout. A merge action must include `merge_preflight` proving security clearance, resolved human comments, resolved review-bot findings, a passed Codex `/review`, addressed review findings, and clean validation commands. Worker-authored proof is planning input only; every executable merge goes through the deterministic exact-head preflight. Jobs with `require_external_merge_preflight: true` route planned or blocked merge actions there immediately. The fix executor runs an agentic edit/review loop before it writes a fix PR: edit, validate, Codex `/review`, address findings, revalidate, and resolve PR review threads when permitted. The applicator also checks live unresolved GitHub review threads immediately before merge.

Replacement fix work uses a recoverable target branch named `clownfish/<cluster-id>`. The executor resumes that branch if it already exists and pushes checkpoint commits after agent edits and review-fix edits, adding `Co-authored-by` trailers for non-bot source PR authors when a contributor PR is replaced. It then opens or updates the PR only after validation and Codex `/review` pass. If `/review` still blocks the merge after retries, the run writes a blocked fix report and leaves the checkpoint branch recoverable instead of losing the patch.

Runs for the same job path and mode are queued instead of running concurrently. The workflow uses Node 24, `blacksmith-4vcpu-ubuntu-2404` for cluster planning/review, and `blacksmith-16vcpu-ubuntu-2404` for fix/apply execution. Fix execution prepares the target checkout with Corepack and the target `pnpm` package manager before validation; the execution job caches Codex, npm, Corepack, and the target pnpm store. Fix validation is pinned to OpenClaw's fast changed-lane posture by default: `pnpm check:changed` plus diff checks are the hard local gate, and target validation commands normalize to `pnpm check:changed` unless `CLOWNFISH_TARGET_VALIDATION_MODE=strict` or `CLOWNFISH_STRICT_TARGET_VALIDATION=1` is explicitly set. Unrelated flaky main CI, broad `pnpm check`, full tests, live, docker, and e2e lanes do not block narrow ProjectClownfish fixes by default.

Full worker prompts, Codex transcripts, and raw artifacts stay in GitHub Actions. The committed ledger keeps only the cluster summary, run URL, action counts, apply outcomes, closed targets, and needs-human entries.

Worker prompts combine repeated comments and reviews by their GitHub identity,
with `maintainer` and `review_bot` classification tags. Distinct identities and
records without an identity remain separate. Compaction keeps the existing
maintainer and bot priority windows. The retained cluster plan keeps every
original evidence collection, including the public IDs and URLs.

Structured-result repair receives the current result, validator errors, full job
scope, worker and selected mode instructions, policies, and exact hydrated
identities and safety gates. Discussion
and history stay in the retained artifacts for targeted inspection; repair does
not replay the investigation prompt. Missing proof requires a non-mutating or
blocked outcome. If the complete repair prompt exceeds 96,000 characters, the
runner skips inference and leaves the original result to fail validation, with
an explicit diagnostic. It never truncates safety facts to fit the budget.

## Modes

- `plan`: produces recommendations only.
- `execute`: can apply reviewed safe close and explicit clean merge actions from structured JSON.
- `autonomous`: adds live cluster preflight and fix-artifact generation. It may recommend and drive a canonical fix path; direct mutation still goes through the fix executor and applicator gates.
- `route_security`: quarantines true security-sensitive refs without poisoning unrelated cluster work.
- `needs_human`: only product-direction, trust-boundary, canonical-choice, merge-path, or contributor-credit decisions that remain unclear after the hydrated artifact and single-item review/check/decide pass.
- Automated reviewer feedback must be cleared during autonomous PR work. Greptile, Codex, Asile, CodeRabbit, Copilot, and similar bot comments must be addressed, proven non-actionable, or escalated before any merge or post-merge closeout recommendation.
- Merge preflight: no PR can merge until `CLOWNFISH_ALLOW_MERGE=1`, security issues are cleared, comments are resolved, Codex `/review` has passed, findings are addressed, changed-surface validation is clean, and `openclaw/ci-gate` from the canonical GitHub Actions CI workflow succeeded on the exact PR head within 24 hours. Clownfish rebuilds the synthetic merge against current `main`, waits for GitHub's matching test merge, and authorizes that test-merge commit through the strict App-owned `clownfish/exact-merge` check. Bounded disjoint base drift does not trigger `update-branch` or write a merge commit into the contributor branch. With the merge gate closed, ProjectClownfish labels merge-ready targets for human review instead of merging.
- Repair ladder: make the useful contributor PR mergeable when its branch is maintainer-editable; otherwise replace draft, stale, unmergeable, uneditable, or unsafe branches with a narrow credited fix PR. When fix PR mode is enabled, "wait or replace" is already answered: replace, preserve credit, then supersede only the source PR that could not be safely updated.

## Maintainer Comment Commands

Clownfish can route maintainer comments from target repositories back into the
cloud repair workflow. It recognizes both command styles:

```text
/clownfish status
@openclaw-clownfish status
```

Do not use `@clownfish`; that is a separate GitHub user. The accepted mention is
`@openclaw-clownfish` or `@openclaw-clownfish[bot]`.

Only maintainers can trigger it. The router checks GitHub `author_association`
and accepts `OWNER`, `MEMBER`, and `COLLABORATOR` by default. Contributor and
unknown comments are ignored without a reply.

Supported commands:

```text
/clownfish status
/clownfish fix ci
/clownfish address review
/clownfish rebase
/clownfish explain
/clownfish stop
@openclaw-clownfish fix ci
```

`status` and `explain` post a short status reply. `fix ci`, `address review`,
and `rebase` dispatch the normal `cluster-worker.yml` repair path, but only for
existing Clownfish PRs identified by the `clownfish` label or `clownfish/*`
branch. `stop` labels the item for human review.

The router writes an idempotency marker into each reply and records processed
comments in `results/comment-router.json`. The scheduled workflow is dry by
default; set `CLOWNFISH_COMMENT_ROUTER_EXECUTE=1` to let scheduled runs post
replies and dispatch workers.

## Local Run

Requires Node 24.

```bash
# Validate all job files.
npm run validate

# Render a plan-mode prompt without running Codex.
npm run render -- jobs/openclaw/inbox/cluster-example.md --mode plan

# Dry-run a worker without calling Codex.
npm run worker -- jobs/openclaw/inbox/cluster-example.md --mode plan --dry-run

# Build an offline autonomous cluster/fix artifact.
npm run build-fix-artifact -- jobs/openclaw/inbox/autonomous-example.md --offline

# Stage low-signal PR sweep jobs from local gitcrawl data.
npm run import-gitcrawl-low-signal -- --limit 20 --batch-size 5 --mode autonomous --sort stale

# Stage the next largest active gitcrawl clusters, skipping already-imported and
# fully security-sensitive clusters by default. Mixed clusters can route security
# refs while continuing ordinary bug/dedupe work.
npm run import-gitcrawl -- --from-gitcrawl --limit 40 --mode autonomous --suffix autonomous-smoke --allow-instant-close --allow-merge --allow-fix-pr --allow-post-merge-close

# If the local gitcrawl cluster id space changed, ignore stale job cluster ids
# and dedupe only against published worker result refs.
npm run import-gitcrawl -- --from-gitcrawl --limit 80 --mode autonomous \
  --suffix autonomous-issue-wave \
  --existing-results-only \
  --overlap-policy exclude-existing \
  --allow-instant-close \
  --allow-merge \
  --allow-fix-pr \
  --allow-post-merge-close

# Dispatch reviewed jobs. Dispatch, requeue, and self-heal refuse to exceed
# 32 live cluster-worker runs by default; tune with CLOWNFISH_MAX_LIVE_WORKERS
# or --max-live-workers. With --wait-for-capacity, dispatch can drain a larger
# file list in capacity-sized waves instead of refusing the whole batch.
CLOWNFISH_MAX_LIVE_WORKERS=32 npm run dispatch -- jobs/openclaw/inbox/cluster-example.md \
  --mode autonomous \
  --runner blacksmith-4vcpu-ubuntu-2404 \
  --execution-runner blacksmith-16vcpu-ubuntu-2404

# Select a plan-only wave by total referenced issue/PR count without dispatching.
# This is useful for high-volume inventory shards where job count is less useful
# than the number of hydrated refs the wave will consume.
npm run queue-status -- \
  --attempt-filter unattempted \
  --plan-ref-limit 2500 \
  --write-missing-plan /tmp/clownfish-plan-wave.txt \
  --allow-app-token-auth

# Refresh only verified-open jobs that the live sweep marked requeueable.
# This is an explicit plan-only lane for stale historical results; it does not
# auto-dispatch or convert the selected jobs into write-mode work.
npm run sweep-openclaw-jobs -- --live --verify-target-refs-live --report /tmp/clownfish-sweep.json
npm run queue-status -- \
  --sweep-report /tmp/clownfish-sweep.json \
  --sweep-requeue-limit 30 \
  --write-sweep-requeue /tmp/clownfish-sweep-requeue.txt \
  --allow-app-token-auth
npm run dispatch -- --jobs-file /tmp/clownfish-sweep-requeue.txt --mode plan

# High-volume plan inventory waves should keep hydration lean. This hydrates
# only the listed PR refs and skips issue/review comments for first-pass
# classification; follow-up execute/autonomous jobs can rehydrate deeper.
npm run dispatch -- --jobs-file /tmp/clownfish-plan-wave-all.txt \
  --mode plan \
  --dispatch-event repository-batch \
  --allow-app-token-auth \
  --max-live-workers 32 \
  --batch-max-parallel 8 \
  --batch-matrix-limit 200 \
  --hydrate-comments 0 \
  --max-linked-refs 0 \
  --max-comments-per-item 0 \
  --max-review-comments-per-pr 0 \
  --runner blacksmith-4vcpu-ubuntu-2404 \
  --execution-runner blacksmith-16vcpu-ubuntu-2404

# Full open-PR inventory sweep from the local gitcrawl snapshot. Generate
# plan-only shards from currently open PRs, skipping refs already present in
# the active inbox. This is the high-volume lane; it performs classification,
# not writes.
npm run import-gitcrawl-pr-inventory -- \
  --limit all \
  --batch-size 40 \
  --existing-dir jobs/openclaw/inbox \
  --mode plan

# Dispatch that full plan lane with lean hydration. Repository-batch admission
# rejects waves whose aggregate matrix width exceeds 32 workers, so this
# high-volume example uses one worker per batch and stays inside the cap.
node scripts/queue-status.mjs \
  --plan-ref-limit 4000 \
  --write-missing-plan /tmp/clownfish-plan-wave-all.txt \
  --allow-app-token-auth
npm run dispatch -- --jobs-file /tmp/clownfish-plan-wave-all.txt \
  --mode plan \
  --dispatch-event repository-batch \
  --allow-app-token-auth \
  --max-live-workers 32 \
  --batch-max-parallel 1 \
  --batch-matrix-limit 200 \
  --hydrate-comments 0 \
  --max-linked-refs 0 \
  --max-comments-per-item 0 \
  --max-review-comments-per-pr 0 \
  --runner blacksmith-4vcpu-ubuntu-2404 \
  --execution-runner blacksmith-16vcpu-ubuntu-2404

# Write/apply dispatches are deliberately capped separately. The default
# write lane cap is 5 workers even if the plan lane runs wider. Override only
# after fresh canaries prove the App quota and applicator path are healthy.
npm run dispatch -- --jobs-file /tmp/clownfish-close-retry.txt \
  --mode execute \
  --allow-app-token-auth \
  --wait-for-capacity \
  --write-live-worker-cap 3 \
  --batch-size 1 \
  --batch-delay-ms 120000 \
  --dispatch-concurrency 1 \
  --runner blacksmith-4vcpu-ubuntu-2404 \
  --execution-runner blacksmith-16vcpu-ubuntu-2404

# Find explicitly recoverable, checkpointed active-inbox fix failures that have
# not been superseded by a later child result. A failed batch alone is not
# enough to requeue work.
npm run self-heal

# Resolve a job from a run id or job path and show the requeue plan.
npm run requeue -- 24947178021

# Requeue one reviewed job/run into the live queue. This briefly opens both
# write gates when the job is execute/autonomous, waits for the run to start,
# then closes the gates.
npm run requeue -- 24947178021 --execute --open-execute-window \
  --runner blacksmith-4vcpu-ubuntu-2404 \
  --execution-runner blacksmith-16vcpu-ubuntu-2404

# Execute a reviewed fix artifact locally. Requires both execution gates and a write token.
CLOWNFISH_ALLOW_EXECUTE=1 CLOWNFISH_ALLOW_FIX_PR=1 npm run execute-fix -- jobs/openclaw/inbox/cluster-example.md --latest --dry-run

# Rebuild the open Clownfish PR finalization report without mutating GitHub.
npm run finalize-open-prs -- --write-report

# Dry-run maintainer comment routing. Recognizes `/clownfish ...` and
# `@openclaw-clownfish ...` in recent issue/PR comments.
npm run comment-router -- --repo openclaw/openclaw --lookback-minutes 180

# Execute maintainer comment routing: post replies and dispatch repair workers
# for existing Clownfish PRs when maintainers ask for `fix ci`,
# `address review`, or `rebase`.
npm run comment-router -- --repo openclaw/openclaw --execute --wait-for-capacity

# Dry-run job hygiene: classify old smoke jobs, outbox-ready jobs, unprocessed
# jobs, and requeue candidates without deleting, moving, or dispatching.
npm run sweep-openclaw-jobs -- --live

# Apply reviewed job hygiene. This deletes old smoke jobs, moves finalized jobs
# to jobs/openclaw/outbox/finalized, and parks never-run backlog in
# jobs/openclaw/outbox/stuck; it never dispatches workers.
npm run sweep-openclaw-jobs -- --live --apply-delete-tests --apply-outbox --apply-stuck

# Dry-run a parked-backlog promotion from outbox/stuck back into inbox.
npm run promote-stuck-jobs -- --limit 20

# Promote the largest parked-backlog jobs into the active queue.
npm run promote-stuck-jobs -- --sort size --limit 20 --apply

# Promote every parked-backlog job, largest clusters first.
npm run promote-stuck-jobs -- --sort size --limit all --apply

# Dry-run the Clownfish label backfill. This verifies live GitHub state and
# reports the exact PRs/issues that would receive the "clownfish" label.
npm run tag-clownfish -- --live

# Apply the label backfill after reviewing the dry-run report.
CLOWNFISH_ALLOW_EXECUTE=1 npm run tag-clownfish -- --live --apply

# Retry failed jobs once. This briefly opens the execution gate, waits for the
# dispatched workers to start, records the self-heal ledger, and closes the gate.
npm run self-heal -- --execute --open-execute-window --max-jobs 5 \
  --max-live-workers 32 \
  --runner blacksmith-4vcpu-ubuntu-2404 \
  --execution-runner blacksmith-16vcpu-ubuntu-2404
```

## Checks

Requeue-by-run-id removes its temporary artifact directory after resolution,
including download and parse failures. Artifact downloads have a two-minute
deadline; set `CLOWNFISH_REQUEUE_DOWNLOAD_TIMEOUT_MS` to a positive integer
number of milliseconds for larger downloads. Invalid values use the default.

Fix execution removes its automatically created workspace when the process exits,
after copying report diagnostics. An explicit `--work-dir` remains owned by the
operator and is retained, as is a separately supplied `--target-dir`.

GitHub CLI inspection, dispatch, routing, apply, and finalization calls, along
with the backlog probe's Git commands, have a two-minute child deadline.
`CLOWNFISH_GH_EXEC_TIMEOUT_MS` sets a positive integer duration in milliseconds
(up to 2,147,483,647); invalid values use the default. Capacity queries share
the remaining wait deadline, and backlog waiting includes its initial probe.
A backlog timeout blocks dispatch. Read-only secret/variable inspection reports
unavailable metadata through the existing warning path.

Timed-out writes are not automatically retried. Dispatch, router, and finalizer ledgers
retain unknown outcomes; verify GitHub state before submitting a new command
or using the finalizer's explicit `--allow-repeat` option.
Finalizer workflows serialize ledger owners, read current main, and publish their ledger even after
a failed dispatch step. Router publication replay retains unknown outcomes.
Nested backlog probes and their helpers share the outer cancellation boundary.

A successful merge command can mean GitHub only queued the PR. Post-flight and
automerge require the reviewed head, live merge timestamp and full merge commit SHA before recording
success; unconfirmed merges do not announce completion or close linked issues.
On replay, automerge records GitHub's confirmed merge only for the reviewed head,
without submitting another merge or claiming Clownfish performed the earlier merge.
Pending merge confirmations stay in the ledger beyond the recent-comment lookback
and terminal history limit. The router re-fetches those commands by ID, retires
edited or deleted commands, and never resubmits a merge while confirmation is pending.
With a one-comment scan limit, persisted per-repository scheduling alternates
pending observations and fresh commands while preserving the one-command limit.
Post-flight checkpoints a submitted request in its result report before fetching
confirmation, so retries observe that request instead of submitting it again.
A fully bound preflight for a new head can supersede the old checkpoint; all live
merge gates still apply to that new request.
External merge preflight and ready-PR inventory also block failing or pending legacy
commit statuses and unfinished checks, while preserving their ignored-check exceptions.

```bash
npm run validate
for f in scripts/*.mjs; do node --check "$f" || exit 1; done
npm run review-results -- .projectclownfish/runs
npm run publish-result -- .projectclownfish/runs
git diff --check
```

## GitHub Actions Setup

The workflow needs:

- Codex/OpenAI authentication for model execution
- a read-only GitHub token for worker inspection
- separate least-privilege GitHub App tokens for deterministic mutation, exact-check writes, and ruleset inspection. Ruleset inspection needs `Administration: write`; GitHub otherwise omits bypass actors, and Clownfish fails closed.
- execution gates that default closed: set `CLOWNFISH_ALLOW_EXECUTE=1` and `CLOWNFISH_ALLOW_FIX_PR=1` only for an intentional execution window; otherwise execute/autonomous dispatches render plan-only output and skip mutation steps
- merge is separately gated by `CLOWNFISH_ALLOW_MERGE`; automerge additionally requires `CLOWNFISH_ALLOW_AUTOMERGE`; both default to `0`, and merge-ready PRs are labeled `clownfish:human-review` and `clownfish:merge-ready` for a maintainer to merge manually
- optional `CLOWNFISH_CODEX_CLI_VERSION` variable to pin and refresh the cached Codex CLI
- optional `CLOWNFISH_MODEL` override for dispatch scripts; default Codex model is `gpt-5.5`
- optional `CLOWNFISH_MAX_LIVE_WORKERS` variable for dispatch/requeue/self-heal worker fan-out; default is `32`
- optional `CLOWNFISH_EXTERNAL_PREFLIGHT_CONCURRENCY` variable for parallel read-only external merge validation inside each cluster worker; default is `3`. Guarded merge application remains sequential so concurrent preflights cannot race mutations against a moving base.
- optional `CLOWNFISH_EXTERNAL_PREFLIGHT_CODEX_SANDBOX` and `CLOWNFISH_EXTERNAL_PREFLIGHT_CODEX_USE_LEGACY_LANDLOCK` variables for the exact-merge Codex review. The sandbox defaults to `read-only`; GitHub Actions enables Codex's legacy Landlock backend by default because hosted Ubuntu runners can reject bubblewrap network namespace setup. Set the legacy Landlock variable to `0` after the hosted runner supports the default backend. The review subprocess receives no GitHub token, runs on an ephemeral checkout, and is followed by tracked-byte, tree, parent, and Git-config integrity verification.
- optional `CLOWNFISH_EXTERNAL_PREFLIGHT_MAX_DISJOINT_BASE_COMMITS`, `CLOWNFISH_EXTERNAL_PREFLIGHT_MAX_DISJOINT_BASE_FILES`, and `CLOWNFISH_EXTERNAL_PREFLIGHT_MAX_BASE_REVALIDATIONS` variables for stabilizing an exact-head review when `main` advances; defaults are `20` commits, `128` files, and `2` validation reruns. Reuse requires linear ancestry, bounded files, no changed-path overlap, and an identical effective merge fingerprint; Clownfish then rebuilds the synthetic merge and reruns validation on refreshed `main`. Validation-control or same-area drift also reruns Codex review. Nonlinear, overlapping, larger, or continuously moving drift blocks the merge.
- optional `CLOWNFISH_REQUIRED_CI_GATE_NAME`, `CLOWNFISH_REQUIRED_CI_GATE_APP_ID`, `CLOWNFISH_REQUIRED_CI_GATE_MAX_AGE_HOURS`, and `CLOWNFISH_REQUIRED_CI_WORKFLOW_PATH` variables for the exact-head final gate; defaults are `openclaw/ci-gate`, GitHub Actions App `15368`, `24`, and `.github/workflows/ci.yml`. Clownfish verifies the check's Actions job, run, workflow, PR, and exact head; PRs changing the canonical CI workflow require maintainer handling. The repository ruleset must strictly require `clownfish/exact-merge` from the Clownfish App. `CLOWNFISH_RULES_GH_TOKEN` supplies the dedicated ruleset-write credential used to prove that App is not a bypass actor. The same non-bypassed App token publishes the authorization and performs the merge. Clownfish writes authorization to GitHub's current two-parent test-merge commit, so any new `main` commit invalidates it without a contributor-branch refresh or an App bypass.
- optional `CLOWNFISH_EXTERNAL_PREFLIGHT_HEARTBEAT_MS` variable for progress output while buffered external preflight children run; default is `60000` ms. Each child logs its start, elapsed heartbeat, and finish without exposing captured command output or credentials.
- optional `CLOWNFISH_MAX_ACTIVE_PRS_PER_AREA` variable for replacement PR backpressure; default is `50` open Clownfish PRs per touched area, `0` disables the area cap, and common changelog/release-note files are ignored for this check
- ClawSweeper commit-finding repair PRs are labeled `clownfish:commit-finding`
- optional `CLOWNFISH_CODEX_TIMEOUT_MS`, `CLOWNFISH_FIX_CODEX_TIMEOUT_MS`, and `CLOWNFISH_FIX_STEP_TIMEOUT_MS` variables; worker planning defaults to 30 minutes, while fix execution defaults to a 20 minute Codex budget inside a 40 minute executor step so timeout artifacts can be written
- optional `CLOWNFISH_PLANNER_TIMEOUT_MS` and `CLOWNFISH_REVIEW_TIMEOUT_MS` variables bound each cluster-planner and result-validation subprocess to 10 minutes by default, including offline autonomous planning and repair/recovery validation. A timeout stops the worker with exit code 0 and an action-free blocked result so artifact reporting can continue; any existing result is preserved as `result.before-timeout.json`. Empty or invalid values use the default.
- optional `CLOWNFISH_REBASE_ONLY_FIX_STEP_TIMEOUT_MS` and `CLOWNFISH_REBASE_ONLY_REVIEW_TIMEOUT_MS` variables; explicit `rebase_only` jobs default to 25 minutes with a five minute review cap, refresh current main before review to avoid reviewing a stale rebase, emit stage timing to the workflow log, and defer the Codex write preflight unless an actual rebase conflict needs a write worker. The review never mutates GitHub and may make only the minimum read-only `gh` query required by target-repo instructions.
- optional `CLOWNFISH_CODEX_REVIEW_ATTEMPTS` and `CLOWNFISH_RESOLVE_REVIEW_THREADS` variables for agentic merge-prep review loops
- optional `CLOWNFISH_CLAWSWEEPER_MAX_REPAIRS_PER_PR` and
  `CLOWNFISH_CLAWSWEEPER_MAX_REPAIRS_PER_HEAD` variables for trusted
  ClawSweeper review feedback; defaults are `5` automatic repair iterations per
  PR and `1` repair per PR head SHA. The per-PR cap is total across changing
  head SHAs and stops the automatic review/repair loop.
- optional `CLOWNFISH_COMMENT_ROUTER_EXECUTE=1` to let the scheduled comment
  router respond to maintainer-only `/clownfish ...` and
  `@openclaw-clownfish ...` commands. Without it, scheduled runs only write a
  dry report.

Keep exact secret names, token scopes, and execution-window procedures in private operations docs or repository settings notes. Do not put token values or live operational credentials in job files.
