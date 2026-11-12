# Incidents

Newest first.

## INC-001 — leak drill blocked for the wrong reason (2026-11-12)

- **What I broke:** Staged a fake GitHub token and ran `git commit` to prove the gitleaks hook blocks it.
- **Symptom:** Commit refused with "No .pre-commit-config.yaml file was found". No gitleaks output at all.
- **Diagnosis:** The config step had been skipped. pre-commit fails closed without a config, so pre-commit itself refused the commit; gitleaks never ran.
- **Fix:** Created .pre-commit-config.yaml and re-ran the drill. Output showed hook "gitleaks (staged changes)" Failed with RuleID github-pat.
- **Lesson:** YOUR LESSON
- **Time to fix:** 5 min

## INC-000 — check-ignore exit code lied about the negated rule (2026-11-12)

- **What I broke:** Verified .gitignore with `git check-ignore -v <paths>; echo "exit: $?"`, expecting exit 1 for paths that must NOT be ignored.
- **Symptom:** `exit: 0`, although neither `.env.example` nor `.terraform.lock.hcl` is ignored.
- **Diagnosis:** With `-v`, a path matching a negated rule (`!.env.example`) counts as a match for the exit code. Without `-v`, the exit code counts only paths that are actually ignored.
- **Fix:** Re-ran without `-v` → `exit: 1`.
- **Lesson:** Test the check on a known-yes and a known-no case before trusting its exit code in a script.
- **Time to fix:** 5 min
