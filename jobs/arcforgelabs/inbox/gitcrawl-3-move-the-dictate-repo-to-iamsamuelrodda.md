---
repo: arcforgelabs/dictate
cluster_id: gitcrawl-3-move-the-dictate-repo-to-iamsamuelrodda
mode: plan
allowed_actions:
  - comment
  - label
  - close
blocked_actions:
  - force_push
  - bypass_checks
  - merge
  - fix
  - raise_pr
require_human_for:
  - security_sensitive
  - failing_checks
  - conflicting_prs
  - unclear_canonical
  - broad_code_delta
canonical:
  - "#135"
candidates:
  - "#135"
  - "#164"
cluster_refs:
  - "#135"
  - "#164"
overlap_policy: "skip-any"
security_policy: central_security_only
import_security_policy: "skip-full"
security_sensitive: false
security_signal_refs:
  - "#135"
canonical_hint: "gitcrawl representative #135 is open; worker must verify it is still the best live canonical."
notes: "Generated from gitcrawl run cluster 3 on 2026-10-14. Security-signal refs #135 must be routed with route_security and must not block unrelated non-security work."
---

# Gitcrawl Cluster 3

Generated from local gitcrawl run cluster 3 for `arcforgelabs/dictate`.

Display title:

> Move the dictate repo to IAMSamuelRodda

Cluster shape from gitcrawl:

- total members: 2
- issues: 2
- pull requests: 0
- open candidates in local store: 2
- security-signal refs requiring route_security: #135
- representative: #135, currently open in local store
- latest member update: 2026-10-14T03:31:08.681438026Z

## Goal

Classify the open candidate issues and PRs in read-only plan mode. Do not close anything. If the representative is closed, report whether another open item should become the live canonical. If the cluster contains multiple root causes, split them in the action matrix instead of forcing a single duplicate family.

## Member Inventory

Closed context refs:

- none

Open candidates:

- #135 [security-signal] Move the dictate repo to IAMSamuelRodda
- #164 Move Dictate's public pages off arcforge.au and into this repo
