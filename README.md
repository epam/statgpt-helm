# StatGPT Helm Repository

[![License](https://img.shields.io/github/license/epam/statgpt-helm?color=blue&labelColor=2B3137)](https://github.com/epam/statgpt-helm/blob/main/LICENSE)
[![GitHub Workflow Status (Release)](https://img.shields.io/github/actions/workflow/status/epam/statgpt-helm/release.yaml?logo=github&label=Release%20Charts&logoColor=959DA5&labelColor=2B3137&color=30C151)](https://github.com/epam/statgpt-helm/actions/workflows/release.yaml)
[![GitHub all releases](https://img.shields.io/github/downloads/epam/statgpt-helm/total?logo=github&label=Chart%20Downloads&logoColor=959DA5&labelColor=2B3137&color=30C151)](https://github.com/epam/statgpt-helm/releases)

## Usage

[Helm](https://helm.sh) must be installed to use the charts. Please refer to Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

```console
helm repo add statgpt https://epam.github.io/statgpt-helm
```

If you had already added this repo earlier, run `helm repo update` to retrieve the latest versions of the packages. You can then run `helm search repo statgpt` to see the charts.

To install the `statgpt` chart:

```console
helm install my-release statgpt/statgpt
```

To uninstall the chart:

```console
helm delete my-release
```

## Contributing

<!-- Keep full URL links to repo files because this README syncs from main to gh-pages.  -->
We'd love to have you contribute! Please refer to our [contribution guidelines](https://github.com/epam/statgpt-helm/blob/main/CONTRIBUTING.md) for details.

## License

<!-- Keep full URL links to repo files because this README syncs from main to gh-pages.  -->
[MIT License](https://github.com/epam/statgpt-helm/blob/main/LICENSE).
