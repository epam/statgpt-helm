# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **Helm chart repository** (not application source code). It publishes a single umbrella chart, `statgpt`, that deploys the StatGPT solution — an AI DIAL–based conversational analytics application over SDMX statistics data. Chart packages are released to GitHub Pages (`https://epam.github.io/statgpt-helm`) and consumed with `helm repo add statgpt ...`. The actual application code lives in separate repos (`epam/statgpt`, DIAL charts at `https://charts.dialx.ai`).

Everything of substance lives under `charts/statgpt/`.

## Architecture

The umbrella chart has almost no templates of its own. Its behavior is composed from subchart dependencies declared in `charts/statgpt/Chart.yaml`:

- **Six application components are all aliases of the same `dial-extension` chart** (packaged repo `https://charts.dialx.ai`; source at [`github.com/epam/ai-dial-helm`](https://github.com/epam/ai-dial-helm)): `chat-backend`, `admin-backend`, `admin-frontend`, `portal-frontend`, `sdmx-proxy`, `sdmx-proxy-config-server`. Each is gated by a `<name>.enabled` condition. Because they share one chart, they all expose the **same value schema** (`image`, `env`, `secrets`, `resources`, `livenessProbe`/`readinessProbe`, `serviceAccount`, `podSecurityContext`, `extraVolumes`, `extraDeploy`, etc.). The Deployment, Service, Secret creation, and env/secret mounting all come from `dial-extension` — not from this repo. To understand how a component renders, read the `dial-extension` chart source in `epam/ai-dial-helm`, not `charts/statgpt/templates/`.
- **Stateful dependencies** come from Bitnami: `pgvector` (a `postgresql` alias with the pgvector extension), `elasticsearch`, and `common` (the Bitnami helper library used by the local templates).

The only first-party templates in `charts/statgpt/templates/`:
- `admin-backend-auto-update-cronjob.yaml` — a CronJob that runs the admin-backend image with `ADMIN_MODE=AUTO_UPDATE`, gated on `admin-backend.autoUpdateCronJob.enabled`. It re-derives env/secret wiring from `admin-backend.*` values.
- `extra-list.yaml` — renders arbitrary manifests from the top-level `extraDeploy` list.
- `_helpers.tpl` is intentionally empty; helpers come from the Bitnami `common` chart.

### values.yaml conventions

`charts/statgpt/values.yaml` is the single source of truth for configuration and is heavily commented.

- **Version anchors** at the top of the file pin image tags via YAML anchors (`&backend_version`, `&sdmx_proxy_version`, etc.). Constraints encoded there: `chat-backend` and `admin-backend` **must share** `backend_version`; `sdmx-proxy` and `sdmx-proxy-config-server` **must share** `sdmx_proxy_version`. Change the anchor, not individual tags.
- Shipped defaults have components `enabled: false` and sensitive/site-specific values set to `"environment-specific"` placeholders. Real deployments layer an env-specific values file on top (see `charts/statgpt/examples/`).
- `charts/statgpt/ci/default-values.yaml` is a separate, self-contained values file used only by CI to make an installable release; it uses its own anchors to share `env`/`secrets` (DIAL, pgvector, elastic credentials) across components.

## Common commands

Run chart commands from `charts/statgpt/` unless noted.

```sh
# Fetch/refresh subchart dependencies (required before template/install)
helm dependency update

# Preview rendered manifests
helm template my-release . --namespace my-namespace --values values.yaml
# ...layered with an environment file:
helm template my-release . --namespace my-namespace --values values.yaml --values examples/generic/values.yaml

helm install my-release . --namespace my-namespace --values values.yaml
```

### Lint & test (mirrors CI, run from repo root)

CI uses [chart-testing](https://github.com/helm/chart-testing) (`ct`) configured by `ct.yaml`.

```sh
ct list-changed --config ct.yaml   # which charts changed vs main
ct lint --config ct.yaml           # lint changed charts (needs Python: yamale + yamllint)
ct install --config ct.yaml        # install changed charts into a kind cluster
```

`.pre-commit-config.yaml` runs `check-yaml` (which **excludes** `charts/statgpt/templates/*.yaml|yml` because they contain Helm templating), plus end-of-file / whitespace / line-ending fixers.

### Regenerating docs — important

`charts/statgpt/README.md` is **generated** by [helm-docs](https://github.com/norwoodj/helm-docs) from `values.yaml` comments plus `README.md.gotmpl`. **Never hand-edit `charts/statgpt/README.md`.** To change documented parameters, edit the `# --` annotation comments in `values.yaml`, then regenerate:

```sh
# from repo root
docker run --rm --volume "$(pwd):/helm-docs" -u "$(id -u)" jnorwood/helm-docs:v1.11.3 --chart-search-root=charts/statgpt
```

The `lint-test` workflow fails if `README.md` is out of date relative to `values.yaml`.

## Release & contribution workflow

- **Releasing is automated.** On push to `main`, `.github/workflows/release.yaml` runs `chart-releaser`, packages any chart whose version changed, and publishes it. Therefore a change is released only if you **bump `version` in `charts/statgpt/Chart.yaml`**. `appVersion` tracks the app, `version` tracks the chart. Release PRs by convention are titled `feat: create release X.Y` / `chore: create patch X.Y.Z` and bump `Chart.yaml` version + regenerate `README.md` together.
- **PR titles must follow [Conventional Commits](https://www.conventionalcommits.org/)** — enforced by `pr-title-check.yml` (delegates to `epam/ai-dial-ci`).
- The root `README.md` is auto-synced to the `gh-pages` branch on change (`sync-readme.yaml`); keep links in it as full `https://github.com/...` URLs since it is served from gh-pages.

## Notes

- Cross-component env wiring assumes the `fullnameOverride` naming shown in `ci/default-values.yaml` (e.g. `PGVECTOR_HOST: statgpt-pgvector`, `ELASTIC_CONNECTION_STRING: http://statgpt-elasticsearch:9200`). Changing a component's `fullnameOverride` requires updating every reference to it.
- `charts/statgpt/examples/azure/` shows the production-oriented pattern: Azure Workload Identity + CSI `SecretProviderClass` (via `extraDeploy`) instead of plaintext `secrets`, and `PGVECTOR_USE_MSI` for passwordless Postgres. `examples/generic/` is a minimal, non-production plaintext-secret walkthrough.
