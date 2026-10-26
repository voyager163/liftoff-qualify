# Review guide

Start every review with the number of findings. When there is nothing to
fix, say "No findings" and stop. Edit this file to say what matters most in
this project; reviewers read it on every pull request.

## What failure looks like here

- **It breaks the API contract**: a changed response shape, status code or
  error format that existing clients rely on, without a version change.
- **It loses or corrupts data**: a migration that cannot be rolled back, a
  delete without a guard, or a transaction that leaves partial state.
- **It trusts input**: missing validation, injection, or data from one user
  reachable by another.
- **It lies about health**: `/health/ready` reporting ready while a
  dependency is down, or `/health/live` failing because of a dependency.
- **It weakens a check**: skipped tests, lowered coverage, or a check that
  passes without running anything.

## Important

A finding is Important when the change introduces or exposes one of the
failures above, or any other defect a user would hit. Give the file and line,
the input and state that trigger it, and the wrong result.

## Nits

At most three per review. Skip anything the linter or formatter reports.

## Do not report

- Generated files in `.github/liftoff/` and the rendered skills and agents.
- Lockfile content, beyond whether it matches the manifest.

## Always check

- Every migration has a working downgrade.
- New endpoints have tests, including the failure cases.
- Secrets never appear in code, logs or configuration examples.
