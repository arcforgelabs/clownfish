---
repo: arcforgelabs/dictate
cluster_id: gitcrawl-2-live-preview-of-dictated-text-that-follows-the-focused-field-def
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
  - "#152"
candidates:
  - "#150"
  - "#151"
  - "#152"
cluster_refs:
  - "#150"
  - "#151"
  - "#152"
overlap_policy: "skip-any"
security_policy: central_security_only
import_security_policy: "skip-full"
security_sensitive: false
canonical_hint: "gitcrawl representative #152 is open; worker must verify it is still the best live canonical."
notes: "Generated from gitcrawl run cluster 2 on 2026-10-14."
---

# Gitcrawl Cluster 2

Generated from local gitcrawl run cluster 2 for `arcforgelabs/dictate`.

Display title:

> Live preview of dictated text that follows the focused field (deferred)

Cluster shape from gitcrawl:

- total members: 3
- issues: 3
- pull requests: 0
- open candidates in local store: 3
- representative: #152, currently open in local store
- latest member update: 2026-10-14T03:31:08.678601381Z

## Goal

Classify the open candidate issues and PRs in read-only plan mode. Do not close anything. If the representative is closed, report whether another open item should become the live canonical. If the cluster contains multiple root causes, split them in the action matrix instead of forcing a single duplicate family.

## Member Inventory

Closed context refs:

- none

Open candidates:

- #150 Recording overlay: show a 'recording' pill whenever Dictate is listening
- #151 Don't paste when no text field is focused; keep the dictation and say so
- #152 Live preview of dictated text that follows the focused field (deferred)
