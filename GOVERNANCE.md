# Governance

How changes reach `develop` and `main` in liftoff-qualify. Liftoff generated
this as a starting point under the `single-maintainer-gitflow` profile: No required approvals; the tier sets automated review depth.

## Branches

| Change | From | Into | Merge |
| --- | --- | --- | --- |
| Feature, fix, chore, docs, CI | `feature/…`, `fix/…`, `chore/…`, `docs/…`, `ci/…` | `develop` | Squash |
| Release | `release/X.Y.Z` from a healthy `develop` commit | `main` | Merge commit |
| Hotfix | `hotfix/X.Y.Z` from `main`, with an incident reference | `main` | Merge commit |
| Back-merge after a release | `sync/…` from `develop` with `main` merged in | `develop` | Merge commit |

Dependabot's branches, and those of the coding agents this project uses, may
also open pull requests into `develop`. A release branch is an exact
`develop` commit whose CI passed, so what merges into `main` is what was
tested. CI's `branch-policy` job checks all of this, and its
`release-lineage` job checks that a pull request into `main` contains the
current `main`, so merging leaves exactly the tree CI qualified.

Never push directly to `develop` or `main`, force-push them or squash away
history being synchronized.

## Review

1. Before opening a pull request, run the checks in [AGENTS.md](AGENTS.md)
   and show their output.
2. CI must pass: `ci-gate` covers lint, tests with coverage, dependency
   review and the branch checks; CodeQL blocks merging on high or critical
   alerts.
3. `review-policy` sets the tier from what the change touches
   ([.github/review-policy.yml](.github/review-policy.yml)) and checks the
   required approvals: low 0, medium 0,
   high 0.
4. Every review thread is resolved before merging: fixed, or answered with a
   reason.

## Releases

The Build workflow builds each image once, from every commit on `develop` and
on hotfix branches. It scans the image with Trivy, which fails on fixable high
and critical vulnerabilities, and with Grype for information. It publishes the
image to GitHub Container Registry, tagged with its commit, and attests how it
was built and its SBOM. Nothing is rebuilt for a release.

1. Run the Release workflow from `develop` with the version. It cuts
   `release/X.Y.Z` at a `develop` commit whose CI and build passed, and
   opens its pull request into `main`.
2. Merge it with a merge commit once its checks pass.
3. The After release workflow checks `main` has exactly the release's tree,
   attests the release manifest (the commit, its tree and each image by
   digest) for every image, tags `vX.Y.Z`, publishes the GitHub release and
   opens the `sync/…` pull request that brings `main` back into
   `develop`.

A hotfix works the same way, from `hotfix/X.Y.Z` branched from `main`.

GitHub starts no workflows for pull requests opened with the workflow token.
With a GitHub App for releases (the `RELEASE_APP_CLIENT_ID` variable and the
`RELEASE_APP_PRIVATE_KEY` secret), release and sync pull requests start their
checks like any other. Without one, the workflows start CI and the review
policy for them directly.

## Branch rules

The intended rules are in [.github/rulesets/](.github/rulesets/). They are
not active until you apply them, and they should only be applied after every
check they require (`ci-gate` and `review-policy`) has run successfully on a
real pull request, so a new rule never waits on a check that does not exist.

`liftoff governance assess` shows how far that has got, and
`liftoff governance apply-next` takes the next step Liftoff can take, such as
applying the rulesets, after you agree. Both use your GitHub CLI login.
`liftoff governance verify --prove` then checks that GitHub refuses direct,
forced and deleting pushes. To apply a ruleset yourself instead:

```bash
gh api --method POST repos/OWNER/REPO/rulesets --input .github/rulesets/develop.json
```

The Governance drift workflow then reads back what GitHub enforces and fails
if it differs from the committed rulesets.
