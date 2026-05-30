# Activity Importer — Preserve Commit History Without the Code

This repository recreates my commit history from accounts and repos I've lost (or will lose) access
to, so my GitHub contribution graph survives. It records only commit **metadata** — message,
timestamp, and shortstat (files changed, insertions, deletions) — by appending one line per commit to
a `commits-<project>.txt` log and replaying it as a real commit dated to the original. **No source
code is ever transferred**, which keeps the process safe for proprietary / NDA codebases.

## Conventions

Refined over several imports. Follow these for every new project so the record stays consistent and
honest:

1. **Scope = default branch only.** Capture each repo's default/integration branch
   (`git symbolic-ref --short refs/remotes/origin/HEAD` → e.g. `main` / `master` / `staging` / `dmz`)
   — the work that actually merged and shipped, and what the contribution graph counted. Do **not**
   capture all branches: in a squash-merge workflow that double-counts (the squash commit *and* its
   pre-squash originals) and adds WIP/merge noise, overstating the real graph.
2. **All of your identities.** First enumerate authors
   (`git log --all --format='%ae|%an' | sort -u`) and select every email/name that was you — personal
   email, work email (even if revoked: match by the *historical* address, since it can no longer
   authenticate), and any `…@users.noreply.github.com` aliases.
3. **Every repo.** An employer/org usually spans many repos. Capture across all of them and dedupe by
   commit SHA so a commit reachable from more than one repo/branch is counted once.
4. **Drop stash artifacts.** Exclude commits whose subject starts with `WIP on`, `index on`, or
   `untracked files on` — transient `git stash` snapshots that never counted toward anything.
5. **Re-attribute on replay.** Author *and* committer = your current identity, so the replayed commits
   count toward *your* graph; preserve the original author/committer **dates** so they land on the
   right days.
6. **Mind stale sources.** Before trusting a clone, compare its newest commit date to your actual
   tenure end — a clone last pulled mid-tenure silently under-captures. If remote access is gone, work
   from a clean offline **archive** of the repos; the archive then *is* the source of truth.
7. **Metadata only.** Each replayed commit changes *only* the `commits-<project>.txt` log — never a
   source file. Verify with `git show --stat`.

## Mechanics

```bash
# 1. Extract — per repo, default branch, your identities, with shortstat:
git -C <repo> log <default-branch> -E -i \
  --author='you@personal|you@work|your-alias' \
  --pretty=format:'%H | %s | %ad' --date=iso --shortstat
#    → collapse each commit's shortstat onto its line; drop WIP/index/untracked subjects;
#      merge all repos; dedupe by SHA; sort oldest→newest.

# 2. Replay — one commit per line, original dates, current identity:
GIT_AUTHOR_DATE="$d" GIT_COMMITTER_DATE="$d" \
  git commit --date "$d" --author "Your Name <you@personal>" \
    -m "$subject -  $shortstat"
#    (prepend each line to commits-<project>.txt before committing.)

# 3. Push.
git push origin main
```

For multi-repo / multi-identity / stash handling, script steps 1–2 in Python rather than a shell
loop — it's far more robust than the original `while`-loop approach.

## Captured projects

| Project | Span | Source |
|---|---|---|
| EcoText | 2020–2022 | offline code archive (GitLab/Bitbucket; access lost) |
| EXO Freight | 2022–2024 | local repo clone |
| BlueCargo | 2024–2026 | offline clean archive (org access revoked) |

Each lives in its own `commits-<project>.txt`.
