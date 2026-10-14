---
repo: arcforgelabs/dictate
cluster_id: gitcrawl-4-proposed-retire-hotword-biasing-keep-spelling-replacements-parak
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
  - "#120"
candidates:
  - "#120"
  - "#121"
cluster_refs:
  - "#120"
  - "#121"
overlap_policy: "skip-any"
security_policy: central_security_only
import_security_policy: "skip-full"
security_sensitive: false
canonical_hint: "gitcrawl representative #120 is open; worker must verify it is still the best live canonical."
notes: "Generated from gitcrawl run cluster 4 on 2026-10-14."
---

# Gitcrawl Cluster 4

Generated from local gitcrawl run cluster 4 for `arcforgelabs/dictate`.

Display title:

> Proposed: retire hotword biasing, keep spelling replacements (Parakeet can't use hotwords)

Cluster shape from gitcrawl:

- total members: 2
- issues: 2
- pull requests: 0
- open candidates in local store: 2
- representative: #120, currently open in local store
- latest member update: 2026-10-14T03:31:08.686417499Z

## Goal

Classify the open candidate issues and PRs in read-only plan mode. Do not close anything. If the representative is closed, report whether another open item should become the live canonical. If the cluster contains multiple root causes, split them in the action matrix instead of forcing a single duplicate family.

## Member Inventory

Closed context refs:

- none

Open candidates:

- #120 Proposed: retire hotword biasing, keep spelling replacements (Parakeet can't use hotwords)
- #121 Proposed: if hotword post-correction stays, tighten matching before defaulting to hybrid
