# Incidents

Newest first.

## INC-002 — local main diverged from GitHub after a double reset (2026-11-12)

- **What I broke:** Ran `git reset HEAD~1` twice (the undo step appeared in two consecutive instructions). The first reset removed an unpushed drill commit; the second removed 57429e2, which was already on GitHub.
- **Symptom:** `git push` rejected as non-fast-forward; `git status` said local main and origin/main had diverged; docs/versions.md suddenly showed as modified. A follow-up `git pull` (rebase) stopped with a conflict in NEXT.md.
- **Diagnosis:** The second reset moved local main behind origin/main and turned 57429e2's changes into unstaged edits. New local commits then created two histories from the same parent; the rebase conflicted because both sides had edited NEXT.md.
- **Fix:** `git rebase --abort`, then `git diff origin/main` to see what a reset would discard (only one NEXT.md line), then `git reset --hard origin/main`, then re-applied that one line as a new commit and pushed.
- **Lesson:** When a command fails, stop and read the error before running anything else; every extra command added a layer. Diff against the target before any destructive reset. Amend or reset only before a push; after a push, add a new commit.
- **Time to fix:** 15 min

## INC-001 — leak drill blocked for the wrong reason (2026-11-12)

- **What I broke:** Staged a fake GitHub token and ran `git commit` to prove the gitleaks hook blocks it.
- **Symptom:** Commit refused with "No .pre-commit-config.yaml file was found". No gitleaks output at all.
- **Diagnosis:** The config step had been skipped. pre-commit fails closed without a config, so pre-commit itself refused the commit; gitleaks never ran.
- **Fix:** Created .pre-commit-config.yaml and re-ran the drill. Output showed hook "gitleaks (staged changes)" Failed with RuleID github-pat.
- **Lesson:** When a safety control fires, read why it fired before moving on. A block for the wrong reason, or a warning nobody reads, gives false confidence.
- **Time to fix:** 5 min

## INC-000 — check-ignore exit code lied about the negated rule (2026-11-12)

- **What I broke:** Verified .gitignore with `git check-ignore -v <paths>; echo "exit: $?"`, expecting exit 1 for paths that must NOT be ignored.
- **Symptom:** `exit: 0`, although neither `.env.example` nor `.terraform.lock.hcl` is ignored.
- **Diagnosis:** With `-v`, a path matching a negated rule (`!.env.example`) counts as a match for the exit code. Without `-v`, the exit code counts only paths that are actually ignored.
- **Fix:** Re-ran without `-v` → `exit: 1`.
- **Lesson:** Test the check on a known-yes and a known-no case before trusting its exit code in a script.
- **Time to fix:** 5 min
