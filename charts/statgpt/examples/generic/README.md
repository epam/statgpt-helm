# StatGPT Generic Installation Simple Guide

- [StatGPT Generic Installation Simple Guide](#statgpt-generic-installation-simple-guide)
  - [Prerequisites](#prerequisites)
  - [Expected Outcome](#expected-outcome)
  - [Install](#install)
  - [Uninstall](#uninstall)
  - [StatGPT Configuration](#statgpt-configuration)
  - [DIAL Configuration](#dial-configuration)
  - [What's next?](#whats-next)

## Prerequisites

- Kubernetes cluster `1.24+`
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) installed and configured
- [Helm](https://helm.sh/docs/intro/install/) `3.8.0+` installed
- [Ingress-Nginx Controller](https://kubernetes.github.io/ingress-nginx/deploy/) installed in the cluster
- [cert-manager](https://cert-manager.io/docs/installation/) installed in the cluster (optional)
- [external-dns](https://github.com/kubernetes-sigs/external-dns) installed in the cluster (optional)
- [Azure OpenAI](https://learn.microsoft.com/en-us/azure/ai-services/openai/overview) `gpt-4.1-2025-04-14` and `text-embedding-3-large` models [deployed](https://docs.dialx.ai/tutorials/devops/deployment/deployment-of-models/openai-model-deployment)
- [AI DIAL](https://github.com/epam/ai-dial-helm/blob/main/charts/dial/examples/generic/simple/README.md) system deployed with OpenAI adapter enabled and DIAL API Key generated

## Expected Outcome

By following the instructions in this guide, you will successfully install StatGPT and configure it to work with the AI DIAL system.\
Please note that this guide **does not use a persistent disk** for data storage.\
Please note that this guide represents a very basic deployment scenario, and **should never be used in production**.\
Configuring authentication provider, encrypted secrets, model usage limits, Ingress allowlisting and other security measures are **out of scope** of this guide.

## Install

1. Create Kubernetes namespace, e.g. `statgpt`

    **Command:**

    ```sh
    kubectl create namespace statgpt
    ```

    **Output:**

    ```console
    namespace/statgpt created
    ```

1. Add Helm chart repository

    **Command:**

    ```sh
    helm repo add statgpt https://epam.github.io/statgpt-helm
    ```

    **Output:**

    ```console
    "statgpt" has been added to your repositories
    ```

1. Copy [values.yaml](values.yaml) file to your working directory and fill in missing values:
    - Replace `%%NAMESPACE%%` with namespace created above, e.g. `statgpt`
    - Replace `%%RELEASE_NAME%%` with release name created above, e.g. `statgpt`
    - Replace `%%DOMAIN%%` with your domain name, e.g. `example.com`
    - Replace `%%DIAL_URL%%` with DIAL Core URL from [prerequisites](#prerequisites), e.g. `http://dial-core.dial.svc.cluster.local:80`
    - Replace `%%DIAL_API_KEY%%` with DIAL API Key value from [prerequisites](#prerequisites) (generated during DIAL configuration)
    - Replace `%%SDMX_PORTAL_URL%%` with SDMX Portal URL, e.g. `https://explorer.statgpt.dialx.ai/en/explorer`
    - Replace `%%NEXTAUTH_SECRET%%` with generated value (`openssl rand -base64 64`)
    - Replace `%%POSTGRES_ADMIN_PASSWORD%%` with generated value (`pwgen -s -1 32`)
    - Replace `%%POSTGRES_DB_USERNAME%%`, e.g. `statgpt`
    - Replace `%%POSTGRES_DB_PASSWORD%%` with generated value (`pwgen -s -1 32`)
    - Replace `%%ELASTIC_USERNAME%%`, e.g. `statgpt`
    - Replace `%%ELASTIC_PASSWORD%%` with generated value (`pwgen -s -1 32`)
    - It's assumed you've configured **external-dns** and **cert-manager** beforehand, so replace `%%CLUSTER_ISSUER%%` with your cluster issuer name, e.g. `letsencrypt-production`

1. Install `statgpt` helm chart in created namespace, applying custom values file:

    **Command:**

    ```sh
    helm install statgpt statgpt/statgpt -f values.yaml --namespace statgpt
    ```

    **Output:**

    ```console
    Release "statgpt" does not exist. Installing it now.
    NAME: statgpt
    LAST DEPLOYED: Thu Sep 31 15:44:21 2025
    NAMESPACE: statgpt
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    CHART NAME: statgpt
    CHART VERSION: 1.0.0
    APP VERSION: 1.0.0
    ** Please be patient while the chart is being deployed **
    ```

1. Now you can access:
    - StatGPT Admin by the following URL: `https://admin.%%DOMAIN%%/`, e.g. `https://admin.example.com/`

## Uninstall

1. Uninstall `statgpt` helm chart from created namespace

    **Command:**

    ```sh
    helm uninstall statgpt --namespace statgpt
    ```

    **Output:**

    ```console
    release "statgpt" uninstalled
    ```

1. Delete Kubernetes namespace, e.g. `statgpt`

    **Command:**

    ```sh
    kubectl delete namespace statgpt
    ```

    **Output:**

    ```console
    namespace "statgpt" deleted
    ```

## StatGPT Configuration

1. Access StatGPT Admin at the following URL: `https://admin.%%DOMAIN%%/`

1. Go to the Channels tab and click the Import button

1. Import the [test channel archive](LINK_HERE) and wait for it to load (the `statgpt-app` channel should appear on the Channels tab)

1. Now StatGPT is ready to be connected to the AI DIAL system

## DIAL Configuration

1. The DIAL API Key for StatGPT must have a role that provides access to all necessary Azure OpenAI models. Configuration example:

```
{
    "keys": {
        "statgpt_api_key": {
            "project": "StatGPT",
            "role": "statgpt"
        }
    },
    "roles": {
        "statgpt": {
            "limits": {
                "gpt-4.1-2025-04-14": {
                    "day": "10000000",
                    "minute": "640000"
                },
                "text-embedding-3-large": {
                    "day": "10000000",
                    "minute": "640000"
                }
            }
        }
    }
}
```

2. StatGPT must be added to the DIAL Core configuration as an application (actually, each StatGPT Channel is one DIAL Application). Additionally, the necessary roles must be granted access to the StatGPT application. Configuration example:

```
{
    "applications": {
        "statgpt-app": {
            "displayName": "StatGPT",
            "description": "StatGPT application.",
            "descriptionKeywords": [
                "Talk-To-Your-Data",
                "StatGPT"
            ],
            "endpoint": "http://statgpt-chat-backend.statgpt/openai/deployments/statgpt-app/chat/completions",
            "forwardAuthToken": true,
            "features": {
                "configurationEndpoint": "http://statgpt-chat-backend.statgpt/openai/deployments/statgpt-app/configuration"
            },
            "userRoles": [
                "statgpt-role"
            ]
        }
    },
    "roles": {
        "statgpt-role": {
            "limits": {
                "statgpt-app": {}
            }
        }
    }
}
```

3. Now StatGPT is completely ready to use

## What's next?

- [StatGPT Documentation](https://github.com/epam/statgpt-documentation)
