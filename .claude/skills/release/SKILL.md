---
name: release
description: Prepare a StatGPT helm chart release — collect new component versions from the source repos, confirm targets with the user, open the release PR (version anchors, Chart.yaml, regenerated README), and attach a release-notes draft. Use whenever the user wants to cut/create/prepare a chart release or patch, bump component versions, check whether a release is due, or draft chart release notes — even if they just say "backend 0.X is out, update the chart".
---

# StatGPT Chart Release

Prepares a release of the `statgpt` umbrella chart. The chart pins application
versions via YAML anchors in `charts/statgpt/values.yaml`; releasing means bumping
those anchors + the chart version, regenerating docs, and opening a PR. Publishing
itself is automated: when the PR merges to `main`, chart-releaser packages any chart
whose `version` changed and creates the GitHub release. Release notes are managed by
CI/humans after that — this skill only produces the **draft** and attaches it to the PR.

## Component → source repo mapping

| values.yaml anchor | Source repo (releases + notes) | Chart components using it |
|---|---|---|
| `_backend_version` | `epam/statgpt-backend` | chat-backend, admin-backend |
| `_admin_frontend_version` | `epam/statgpt-admin-frontend` | admin-frontend |
| `_portal-frontend_version` | `epam/statgpt-global-trusted-data-commons` | portal-frontend |
| `_sdmx_proxy_version` | `epam/statgpt-sdmx-proxy` | sdmx-proxy, sdmx-proxy-config-server |

Paired components (backend×2, sdmx-proxy×2) must stay on the same version — that is
why the anchors exist. Always edit the anchor at the top of values.yaml, never the
individual `image.tag` fields.

## Step 1 — Collect versions

1. Read the current anchors from the top of `charts/statgpt/values.yaml` and the
   current chart `version` from `charts/statgpt/Chart.yaml`.
2. For each source repo: `gh release list -R epam/<repo> --limit 10`.
3. For every component that is behind, fetch the release notes of **every
   intermediate version**, not just the latest
   (`gh release view <tag> -R epam/<repo> --json body -q '.body'`).
   A chart that skips from 0.11.0 to 0.13.0 must aggregate the changelogs of 0.12.0
   *and* 0.13.0 — deployment changes in a skipped version still apply to upgraders.
4. From each release body, extract:
   - notable Features/Fixes bullets (for "What's Changed"),
   - the "Deployment Changes" section (some repos spell it "Deployment changes").
     Anything other than "no deployment changes" means the chart's `values.yaml`
     env/secret schema likely needs edits (renames, new vars, removed vars) — these
     are the changes release 1.12 made (`NEXTAUTH_URL`→`AUTH_URL` etc.).

## Step 2 — Confirm with the user

Propose the target chart version:
- **Minor** (`X.(Y+1).0`) when any component bump brings features or deployment
  changes — PR/commit title `feat: create release X.Y`, branch `feat/release-X.Y`.
- **Patch** (`X.Y.(Z+1)`) when only fixes/patch bumps — title
  `chore: create patch X.Y.Z`, branch `chore/patch-X.Y.Z`.

Present a table (component | current | proposed | deployment changes?) and confirm
with AskUserQuestion: the target chart version and each component version (the user
may want to hold a component back). Also surface any deployment changes verbatim and
agree on how they map to values.yaml edits before touching files.

## Step 3 — Prepare the PR

1. Create the branch from up-to-date `main`.
2. Edit the version anchors in `charts/statgpt/values.yaml`.
3. Apply values schema changes implied by the components' deployment changes
   (env/secret renames, new vars with sensible defaults, removals). Keep the `# --`
   helm-docs comment on every new/renamed key — the README is generated from them.
   Check whether `charts/statgpt/ci/default-values.yaml` and
   `charts/statgpt/examples/*/values.yaml` reference renamed/removed vars and update
   them too.
4. Bump `version` in `charts/statgpt/Chart.yaml` (leave `appVersion` alone unless
   the user asks).
5. Regenerate the README — never hand-edit it (CI fails if it is stale):
   ```sh
   docker run --rm --volume "$(pwd):/helm-docs" -u "$(id -u)" jnorwood/helm-docs:v1.11.3 --chart-search-root=charts/statgpt
   ```
6. Sanity-check rendering before pushing:
   ```sh
   cd charts/statgpt && helm dependency update && \
   helm template statgpt . --values values.yaml --values ci/default-values.yaml >/dev/null
   ```
   Then confirm every component renders the intended tag:
   ```sh
   helm template statgpt . --values values.yaml --values ci/default-values.yaml \
     | grep -o 'image: "docker.io/epam/[^"]*"' | sort -u
   ```
   `helm dependency update` leaves untracked artifacts (`Chart.lock`,
   `charts/statgpt/charts/*.tgz`) — never commit them.
7. Commit **only** `Chart.yaml`, `values.yaml`, `README.md` (plus any
   ci/examples values files edited in step 3) with a conventional title matching
   the PR title, push, and open the PR using the repo PR template — fill
   "Description of changes" with the component bump summary, tick the Conventional
   Commits checkbox, drop the unused "fixes #" line.
8. Post the release-notes draft (Step 4) as a PR comment, prefixed with a line like
   `Release notes draft for statgpt-X.Y.Z — for use when editing the GitHub release
   after merge:`.

## Step 4 — Release-notes draft

Follow the established format exactly (see any recent release, e.g.
`gh release view statgpt-1.12.0`). Template:

```markdown
Umbrella chart for StatGPT solution

## Core Components

* `statgpt-backend`: [<v>](https://github.com/epam/statgpt-backend/releases/tag/<v>)
* `statgpt-admin-frontend`: [<v>](https://github.com/epam/statgpt-admin-frontend/releases/tag/<v>)
* `statgpt-global-trusted-data-commons`: [<v>](https://github.com/epam/statgpt-global-trusted-data-commons/releases/tag/<v>)
* `sdmx-proxy`: [<v>](https://github.com/epam/statgpt-sdmx-proxy/releases/tag/<v>)

## What's Changed

* The versions of the core components have been updated.
* <one bullet per other notable chart-level change, if any>

## Deployment Changes

<For each component with migration steps, a "### <component>" subsection with
concrete instructions (rename X→Y, add Z, remove W) aggregated across ALL
intermediate versions. If none:>
No deployment changes required for this release.
```

List **all four** components in Core Components with the versions the chart now
pins, including ones that didn't change. "Deployment Changes" is written for chart
*operators* — phrase entries as actions against their environment values files, not
as feature descriptions. Component notes sometimes deprecate *application/channel
config* fields (not values-file vars) — those are not deployment changes; if
relevant to operators, add them as a short blockquote note instead.

## Refreshing an open release PR

Components often release while the chart release PR is still open. In that case do
not start a new release — refresh the existing PR:

1. Find it: `gh pr list --state open --search "create release in:title"` (or the
   open `feat/release-*` / `chore/patch-*` branch) and check out its branch.
2. Re-run Step 1 for the newly released component(s) only; confirm the delta with
   the user (Step 2). Re-check the minor-vs-patch calculus — a new component
   version with features or deployment changes can upgrade a patch PR to a minor
   one, which means retitling the PR and renaming nothing (keep the branch; PRs are
   squash-merged under the PR title, so only the title must be correct).
3. Apply the additional bumps exactly as in Step 3 (anchor, deployment-change
   edits, helm-docs regeneration, render check) and push a follow-up commit — no
   need to amend or force-push.
4. Update the PR description's version table, and **edit** the existing
   release-notes draft comment rather than posting a second one:
   ```sh
   gh api repos/epam/statgpt-helm/issues/comments/<comment-id> -X PATCH -f body='...'
   ```
   (find the comment id via `gh api repos/epam/statgpt-helm/issues/<pr>/comments`).

## Cautions

- Confirm with the user before pushing the branch / opening the PR.
- If a component's deployment changes are ambiguous (e.g., a new var whose default
  is unclear), ask rather than guessing a default into values.yaml.
- Merging the PR triggers the actual release — never merge it yourself.
