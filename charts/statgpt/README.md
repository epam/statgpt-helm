# statgpt

![Version: 1.0.5](https://img.shields.io/badge/Version-1.0.5-informational?style=flat-square) ![AppVersion: 1.0.0](https://img.shields.io/badge/AppVersion-1.0.0-informational?style=flat-square)

Umbrella chart for StatGPT solution

## Prerequisites

- Helm 3.8.0+
- PV provisioner support in the underlying infrastructure (optional)
- Ingress controller support in the underlying infrastructure (optional)

## Requirements

Kubernetes: `>=1.23.0-0`

| Repository | Name | Version |
|------------|------|---------|
| https://charts.bitnami.com/bitnami | common | 2.14.1 |
| https://charts.bitnami.com/bitnami | elasticsearch(elasticsearch) | 21.3.2 |
| https://charts.bitnami.com/bitnami | pgvector(postgresql) | 15.5.5 |
| https://charts.epam-rail.com | chat-backend(dial-extension) | 1.3.2 |
| https://charts.epam-rail.com | admin-backend(dial-extension) | 1.3.2 |
| https://charts.epam-rail.com | admin-frontend(dial-extension) | 1.3.2 |
| https://charts.epam-rail.com | portal-frontend(dial-extension) | 1.3.2 |

## Validating the Chart

You can preview the generated manifests before installation.

Example for the chart with the release name `my-release` and namespace `my-namespace`:

```console
helm template my-release . --namespace my-namespace --values values.yaml
```

With environment-specific values:

```console
helm template my-release . --namespace my-namespace --values values.yaml --values envs/statgpt/dev/values.yaml
```

## Installing the Chart

First, update dependencies:

```console
helm dependency update
```

To install the chart with the release name `my-release` into namespace `my-namespace`:

```console
helm install my-release . --namespace my-namespace --values values.yaml
```

## Using Environment-Specific Values

If you have environment-specific configuration files, you can merge them with the base values.

Example for development environment:

```console
helm install my-release . --namespace my-namespace --values values.yaml --values envs/statgpt/dev/values.yaml
```

Example for production environment:

```console
helm install my-release . --namespace my-namespace --values values.yaml --values envs/statgpt/prod/values.yaml
```

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment from namespace `my-namespace`:

```console
helm delete my-release --namespace my-namespace
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

**NOTE**: Persistent Volumes created by StatefulSets won't be deleted automatically

## Parameters

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```console
helm install my-release . --namespace my-namespace --values values.yaml --set admin-frontend.image.tag=latest
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| _admin_frontend_version | string | `"0.1.0"` | Admin Frontend version is used for the admin-frontend image tag |
| _backend_version | string | `"0.1.0"` | Backend version is used for both chat-backend and admin-backend image tags (must be the same for both) |
| _elasticsearch_version | string | `"8.14.3-debian-12-r0"` | Elasticsearch version is used for the elasticsearch image tag |
| _pgvector_version | string | `"v0.8.1"` | PGVector extension version |
| _postgresql_version | string | `"16.3.0-debian-12-r14"` | PostgreSQL version is used for the postgresql image tag |
| admin-backend.commonLabels."app.kubernetes.io/component" | string | `"application"` | Kubernetes label to identify the component as an application |
| admin-backend.containerPorts.http | int | `8000` | HTTP port for the application |
| admin-backend.enabled | bool | `false` | Indicates whether the admin-backend service is enabled |
| admin-backend.env.ADMIN_MODE | string | `"APP"` | Application mode for admin settings |
| admin-backend.env.ADMIN_ROLES_CLAIM | string | `"environment-specific"` | Claim used to identify user roles |
| admin-backend.env.ADMIN_ROLES_VALUES | string | `"environment-specific"` | Values within the role claim that signify admin access |
| admin-backend.env.ADMIN_SCOPE_CLAIM_VALIDATION_ENABLED | string | `"environment-specific"` | If "true", the admin portal will check for scopes in the OIDC token, otherwise this check will be skipped |
| admin-backend.env.DIAL_URL | string | `"environment-specific"` | URL for DIAL application |
| admin-backend.env.ELASTIC_CONNECTION_STRING | string | `"environment-specific"` | Connection string for Elasticsearch |
| admin-backend.env.ELASTIC_INDICATORS_INDEX | string | `"environment-specific"` | Index for Elasticsearch indicators |
| admin-backend.env.ELASTIC_MATCHING_INDEX | string | `"environment-specific"` | Index for Elasticsearch matching |
| admin-backend.env.OIDC_AUTH_ENABLED | string | `"environment-specific"` | Enable or disable OIDC authentication |
| admin-backend.env.OIDC_CLIENT_ID | string | `"environment-specific"` | Client ID for OIDC authentication |
| admin-backend.env.OIDC_CONFIGURATION_ENDPOINT | string | `"environment-specific"` | URL to fetch OIDC configuration |
| admin-backend.env.OIDC_ISSUER | string | `"environment-specific"` | OIDC issuer URL |
| admin-backend.env.OIDC_USERNAME_CLAIM | string | `"environment-specific"` | Specify claim used for the username extraction |
| admin-backend.env.PGVECTOR_DATABASE | string | `"environment-specific"` | Database name for PGVector |
| admin-backend.env.PGVECTOR_HOST | string | `"environment-specific"` | Host for PGVector database |
| admin-backend.env.PGVECTOR_PORT | string | `"environment-specific"` | Port for PGVector database |
| admin-backend.env.WEB_CONCURRENCY | string | `"1"` | Number of concurrent web processes |
| admin-backend.image.pullPolicy | string | `"Always"` | Image pull policy |
| admin-backend.image.registry | string | `"docker.io"` | Docker registry URL |
| admin-backend.image.repository | string | `"epam/statgpt-admin-backend"` | Image repository name |
| admin-backend.image.tag | string | `"0.1.0"` | Image tag or version |
| admin-backend.ingress.annotations | object | `{"nginx.ingress.kubernetes.io/proxy-connect-timeout":"600","nginx.ingress.kubernetes.io/proxy-read-timeout":"600","nginx.ingress.kubernetes.io/proxy-send-timeout":"600"}` | NGINX annotations for proxy configuration |
| admin-backend.ingress.enabled | bool | `false` | Enable Ingress resource |
| admin-backend.ingress.ingressClassName | string | `"nginx"` | Specify the Ingress class name |
| admin-backend.ingress.path | string | `"/admin/api"` | Path for the Ingress resource |
| admin-backend.initContainers[0].command[0] | string | `"sh"` |  |
| admin-backend.initContainers[0].command[1] | string | `"-c"` |  |
| admin-backend.initContainers[0].command[2] | string | `"alembic -c $APP_HOME/alembic.ini upgrade head"` |  |
| admin-backend.initContainers[0].env[0].name | string | `"APP_HOME"` |  |
| admin-backend.initContainers[0].env[0].value | string | `"/home/app"` |  |
| admin-backend.initContainers[0].env[1].name | string | `"PGVECTOR_HOST"` |  |
| admin-backend.initContainers[0].env[1].value | string | `"{{ .Values.env.PGVECTOR_HOST}}"` |  |
| admin-backend.initContainers[0].env[2].name | string | `"PGVECTOR_PORT"` |  |
| admin-backend.initContainers[0].env[2].value | string | `"{{ .Values.env.PGVECTOR_PORT }}"` |  |
| admin-backend.initContainers[0].env[3].name | string | `"PGVECTOR_DATABASE"` |  |
| admin-backend.initContainers[0].env[3].value | string | `"{{ .Values.env.PGVECTOR_DATABASE }}"` |  |
| admin-backend.initContainers[0].env[4].name | string | `"PGVECTOR_USER"` |  |
| admin-backend.initContainers[0].env[4].value | string | `"{{ .Values.secrets.PGVECTOR_USER }}"` |  |
| admin-backend.initContainers[0].env[5].name | string | `"PGVECTOR_PASSWORD"` |  |
| admin-backend.initContainers[0].env[5].value | string | `"{{ .Values.secrets.PGVECTOR_PASSWORD }}"` |  |
| admin-backend.initContainers[0].image | string | `"{{ .Values.image.registry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}"` |  |
| admin-backend.initContainers[0].imagePullPolicy | string | `"IfNotPresent"` |  |
| admin-backend.initContainers[0].name | string | `"alembic"` |  |
| admin-backend.livenessProbe.enabled | bool | `true` | Enable livenessProbe |
| admin-backend.metrics.enabled | bool | `false` | Enable metrics collection |
| admin-backend.metrics.serviceMonitor.enabled | bool | `false` | Enable Prometheus ServiceMonitor for metrics |
| admin-backend.readinessProbe.enabled | bool | `true` | Enable readinessProbe |
| admin-backend.resources.limits.cpu | string | `"1000m"` | Maximum CPU limit for the container |
| admin-backend.resources.limits.memory | string | `"4Gi"` | Maximum memory limit for the container |
| admin-backend.resources.requests.cpu | string | `"100m"` | Minimum CPU request for resource scheduling |
| admin-backend.resources.requests.memory | string | `"2Gi"` | Minimum memory request for resource scheduling |
| admin-frontend.commonLabels."app.kubernetes.io/component" | string | `"application"` | Kubernetes label to identify the component as an application |
| admin-frontend.containerPorts.http | int | `3000` | HTTP port for the application |
| admin-frontend.enabled | bool | `false` | Indicates whether the admin-frontend service is enabled |
| admin-frontend.env.API_URL | string | `"environment-specific"` | API URL for admin backend |
| admin-frontend.env.DIAL_API_URL | string | `"environment-specific"` | DIAL API URL |
| admin-frontend.env.NEXTAUTH_URL | string | `"environment-specific"` | URL for NextAuth service |
| admin-frontend.image.pullPolicy | string | `"Always"` | Image pull policy |
| admin-frontend.image.registry | string | `"docker.io"` | Docker registry URL |
| admin-frontend.image.repository | string | `"epam/statgpt-admin-frontend"` | Image repository name |
| admin-frontend.image.tag | string | `"0.1.0"` | Image tag or version |
| admin-frontend.ingress.annotations | object | `{"nginx.ingress.kubernetes.io/proxy-connect-timeout":"600","nginx.ingress.kubernetes.io/proxy-read-timeout":"600","nginx.ingress.kubernetes.io/proxy-send-timeout":"600"}` | NGINX annotations for proxy configuration |
| admin-frontend.ingress.enabled | bool | `false` | Enable Ingress resource |
| admin-frontend.ingress.ingressClassName | string | `"nginx"` | Specify the Ingress class name |
| admin-frontend.ingress.path | string | `"/"` | Path for the Ingress resource |
| admin-frontend.livenessProbe.enabled | bool | `true` | Enable livenessProbe |
| admin-frontend.livenessProbe.httpGet | object | `{"path":"/api/health"}` | HTTP GET request configuration for liveness probe |
| admin-frontend.livenessProbe.httpGet.path | string | `"/api/health"` | Health check endpoint path |
| admin-frontend.metrics.enabled | bool | `false` | Enable metrics collection |
| admin-frontend.metrics.serviceMonitor.enabled | bool | `false` | Enable Prometheus ServiceMonitor for metrics |
| admin-frontend.readinessProbe.enabled | bool | `true` | Enable readinessProbe |
| admin-frontend.readinessProbe.httpGet | object | `{"path":"/api/health"}` | HTTP GET request configuration for liveness probe |
| admin-frontend.readinessProbe.httpGet.path | string | `"/api/health"` | Health check endpoint path |
| admin-frontend.resources.limits.cpu | string | `"1000m"` | Maximum CPU limit for the container |
| admin-frontend.resources.limits.memory | string | `"2Gi"` | Maximum memory limit for the container |
| admin-frontend.resources.requests.cpu | string | `"500m"` | Minimum CPU request for resource scheduling |
| admin-frontend.resources.requests.memory | string | `"0.5Gi"` | Minimum memory request for resource scheduling |
| chat-backend.commonLabels."app.kubernetes.io/component" | string | `"application"` | Kubernetes label to identify the component as an application |
| chat-backend.containerPorts.http | int | `5000` | HTTP port for the application |
| chat-backend.enabled | bool | `false` | Indicates whether the chat-backend service is enabled |
| chat-backend.env.DIAL_APP_NAME | string | `"StatGPT"` | Name of the DIAL app for OpenTelemetry |
| chat-backend.env.DIAL_AUTH_MODE | string | `"user_token"` | Authentication mode for DIAL |
| chat-backend.env.DIAL_SHOW_DEBUG_STAGES | string | `"false"` | Show debug stages in DIAL |
| chat-backend.env.DIAL_SHOW_STAGE_SECONDS | string | `"false"` | Show stage seconds in DIAL |
| chat-backend.env.DIAL_URL | string | `"environment-specific"` | URL for DIAL application |
| chat-backend.env.ELASTIC_CONNECTION_STRING | string | `"environment-specific"` | Connection string for Elasticsearch |
| chat-backend.env.ELASTIC_INDICATORS_INDEX | string | `"environment-specific"` | Index for Elasticsearch indicators |
| chat-backend.env.ELASTIC_MATCHING_INDEX | string | `"environment-specific"` | Index for Elasticsearch matching |
| chat-backend.env.LANGCHAIN_DEBUG | string | `"false"` | Enable debug logging for Langchain |
| chat-backend.env.LANGCHAIN_DEFAULT_SEED | string | `"820288"` | Default seed for Langchain operations |
| chat-backend.env.LANGCHAIN_VERBOSE | string | `"false"` | Enable verbose logging for Langchain |
| chat-backend.env.PGVECTOR_DATABASE | string | `"environment-specific"` | Database name for PGVector |
| chat-backend.env.PGVECTOR_HOST | string | `"environment-specific"` | Host for PGVector database |
| chat-backend.env.PGVECTOR_PORT | string | `"environment-specific"` | Port for PGVector database |
| chat-backend.env.WEB_CONCURRENCY | string | `"1"` | Number of concurrent web processes |
| chat-backend.image.pullPolicy | string | `"Always"` | Image pull policy |
| chat-backend.image.registry | string | `"docker.io"` | Docker registry URL |
| chat-backend.image.repository | string | `"epam/statgpt-chat-backend"` | Image repository name |
| chat-backend.image.tag | string | `"0.1.0"` | Image tag or version |
| chat-backend.livenessProbe.enabled | bool | `true` | Enable livenessProbe |
| chat-backend.metrics.enabled | bool | `false` | Enable metrics collection |
| chat-backend.metrics.serviceMonitor.enabled | bool | `false` | Enable Prometheus ServiceMonitor for metrics |
| chat-backend.podSecurityContext.enabled | bool | `true` | Enable security context for the pod |
| chat-backend.podSecurityContext.runAsUser | int | `5678` | User ID under which the pod runs |
| chat-backend.readinessProbe.enabled | bool | `true` | Enable readinessProbe |
| chat-backend.resources.limits.cpu | string | `"1000m"` | Maximum CPU limit for the container |
| chat-backend.resources.limits.memory | string | `"4Gi"` | Maximum memory limit for the container |
| chat-backend.resources.requests.cpu | string | `"100m"` | Minimum CPU request for resource scheduling |
| chat-backend.resources.requests.memory | string | `"2Gi"` | Minimum memory request for resource scheduling |
| elasticsearch.coordinating.replicaCount | int | `0` | Number of coordinating node replicas |
| elasticsearch.data.replicaCount | int | `0` | Number of data node replicas |
| elasticsearch.enabled | bool | `false` | Indicates whether the elasticsearch service is enabled |
| elasticsearch.image.registry | string | `"docker.io"` | Docker registry URL |
| elasticsearch.image.repository | string | `"bitnamilegacy/elasticsearch"` | Image repository name |
| elasticsearch.image.tag | string | `"8.14.3-debian-12-r0"` | Image tag or version |
| elasticsearch.ingest.replicaCount | int | `0` | Number of ingest node replicas |
| elasticsearch.kibanaEnabled | bool | `false` | Indicates whether Kibana is enabled |
| elasticsearch.master.masterOnly | bool | `false` | Specifies if master nodes are master-only |
| elasticsearch.master.replicaCount | int | `1` | Number of master replicas |
| elasticsearch.master.resources.limits.cpu | string | `"1000m"` | Maximum CPU limit for the container |
| elasticsearch.master.resources.limits.memory | string | `"1Gi"` | Maximum memory limit for the container |
| elasticsearch.master.resources.requests.cpu | string | `"500m"` | Minimum CPU request for resource scheduling |
| elasticsearch.master.resources.requests.memory | string | `"0.5Gi"` | Minimum memory request for resource scheduling |
| elasticsearch.metrics.image.repository | string | `"bitnamilegacy/elasticsearch-exporter"` | Fix: override deprecated repository with the updated one |
| elasticsearch.security.enabled | bool | `false` | Enable security features |
| elasticsearch.security.tls | object | `{"autoGenerated":true}` | Auto-generate TLS certificates |
| elasticsearch.sysctlImage.repository | string | `"bitnamilegacy/os-shell"` | Fix: override deprecated repository with the updated one |
| elasticsearch.volumePermissions.image.repository | string | `"bitnamilegacy/os-shell"` | Fix: override deprecated repository with the updated one |
| pgvector.auth.database | string | `"statgpt"` | Database name |
| pgvector.enabled | bool | `false` | Indicates whether the pgvector service is enabled |
| pgvector.image.registry | string | `"docker.io"` | Docker registry URL |
| pgvector.image.repository | string | `"bitnamilegacy/postgresql"` | Image repository name |
| pgvector.image.tag | string | `"16.3.0-debian-12-r14"` | Image tag or version |
| pgvector.metrics.image.repository | string | `"bitnamilegacy/postgres-exporter"` | Fix: override deprecated repository with the updated one |
| pgvector.pgvectorVersion | string | `"v0.8.1"` | PGVector extension version to build and install |
| pgvector.postgresqlSharedPreloadLibraries | string | `"pgaudit,vector"` | PostgreSQL shared preload libraries including pgaudit for auditing and vector for pgvector extension |
| pgvector.primary.extraVolumeMounts | list | `[{"mountPath":"/opt/bitnami/postgresql/lib","name":"postgresql-files","subPath":"postgresql-files/lib"},{"mountPath":"/opt/bitnami/postgresql/share","name":"postgresql-files","subPath":"postgresql-files/share"}]` | Mount complete PostgreSQL directories (lib and share) with pgvector extension included |
| pgvector.primary.extraVolumes | list | `[{"emptyDir":{},"name":"postgresql-files"}]` | Shared volume for storing complete PostgreSQL directories with pgvector extension |
| pgvector.primary.initContainers | list | `[{"command":["/bin/bash","-c","set -e\necho \"=== Setting up PostgreSQL with pgvector extension ===\"\n\nPGVECTOR_VERSION=\"{{ .Values.pgvectorVersion }}\"\necho \"Installing pgvector version: $PGVECTOR_VERSION\"\n\n# Create directory structure in shared volume\nmkdir -p /shared/postgresql-files/lib /shared/postgresql-files/share\n\n# Copy entire existing PostgreSQL directories to preserve all libraries and extensions\necho \"Copying existing PostgreSQL directories...\"\ncp -r /opt/bitnami/postgresql/lib/* /shared/postgresql-files/lib/\ncp -r /opt/bitnami/postgresql/share/* /shared/postgresql-files/share/\n\n# Install build dependencies for compiling pgvector\necho \"Installing build dependencies...\"\ninstall_packages git build-essential\n\n# Clone and build pgvector from source\necho \"Building pgvector extension version $PGVECTOR_VERSION...\"\ncd /tmp\ngit clone --branch \"$PGVECTOR_VERSION\" https://github.com/pgvector/pgvector.git\ncd pgvector\nexport PG_CONFIG=/opt/bitnami/postgresql/bin/pg_config\nmake clean && make\n\n# Add compiled pgvector files to the copied PostgreSQL directories\necho \"Installing pgvector files...\"\ncp vector.so /shared/postgresql-files/lib/\ncp sql/vector--*.sql /shared/postgresql-files/share/extension/\ncp vector.control /shared/postgresql-files/share/extension/\n\n# Set proper ownership for Bitnami PostgreSQL user\nchown -R 1001:1001 /shared\n\n# Verify installation\necho \"Verifying pgvector installation...\"\nls -la /shared/postgresql-files/lib/vector.so\nls -la /shared/postgresql-files/share/extension/vector.control\n\necho \"=== pgvector $PGVECTOR_VERSION setup completed successfully ===\"\n"],"image":"{{ .Values.image.registry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}","name":"setup-pgvector","securityContext":{"runAsNonRoot":false,"runAsUser":0},"volumeMounts":[{"mountPath":"/shared","name":"postgresql-files"}]}]` | Build and install pgvector extension by copying existing PostgreSQL directories and adding pgvector files |
| pgvector.primary.initdb | object | `{"scripts":{"01_create_extension.sh":"#!/bin/sh\nset -e\n\nexport PGPASSWORD=$POSTGRES_POSTGRES_PASSWORD\n\necho \"=== Creating the vector extension in the database ===\"\npsql -U postgres -d $POSTGRES_DATABASE -c \"CREATE EXTENSION IF NOT EXISTS vector;\"\n\necho \"Currently installed extensions:\"\npsql -U postgres -d $POSTGRES_DATABASE -c \"\\dx\"\n\necho \"=== The vector extension has been successfully created in the database ===\"\n"}}` | Database initialization scripts to create the vector extension after PostgreSQL startup |
| pgvector.primary.resources.limits.cpu | string | `"4000m"` | Maximum CPU limit for the container |
| pgvector.primary.resources.limits.memory | string | `"4Gi"` | Maximum memory limit for the container |
| pgvector.primary.resources.requests.cpu | string | `"2000m"` | Minimum CPU request for resource scheduling |
| pgvector.primary.resources.requests.memory | string | `"2Gi"` | Minimum memory request for resource scheduling |
| pgvector.volumePermissions.image.repository | string | `"bitnamilegacy/os-shell"` | Fix: override deprecated repository with the updated one |
| portal-frontend.commonLabels."app.kubernetes.io/component" | string | `"application"` | Kubernetes label to identify the component as an application |
| portal-frontend.containerPorts.http | int | `3000` | HTTP port for the application |
| portal-frontend.enabled | bool | `false` | Indicates whether the portal-frontend service is enabled |
| portal-frontend.env.DEFAULT_MODEL | string | `"environment-specific"` | Default model |
| portal-frontend.env.DIAL_API_URL | string | `"environment-specific"` | DIAL API URL |
| portal-frontend.env.DIAL_API_VERSION | string | `"environment-specific"` | DIAL API Version |
| portal-frontend.env.SDMX_API_URL | string | `"environment-specific"` | SDMX API URL |
| portal-frontend.image.pullPolicy | string | `"Always"` | Image pull policy |
| portal-frontend.image.registry | string | `"environment-specific"` | Docker registry URL |
| portal-frontend.image.repository | string | `"environment-specific"` | Image repository name |
| portal-frontend.image.tag | string | `"environment-specific"` | Image tag or version |
| portal-frontend.ingress.annotations | object | `{"nginx.ingress.kubernetes.io/proxy-connect-timeout":"600","nginx.ingress.kubernetes.io/proxy-read-timeout":"600","nginx.ingress.kubernetes.io/proxy-send-timeout":"600"}` | NGINX annotations for proxy configuration |
| portal-frontend.ingress.enabled | bool | `false` | Enable Ingress resource |
| portal-frontend.ingress.ingressClassName | string | `"nginx"` | Specify the Ingress class name |
| portal-frontend.ingress.path | string | `"/"` | Path for the Ingress resource |
| portal-frontend.livenessProbe.enabled | bool | `true` | Enable livenessProbe |
| portal-frontend.livenessProbe.httpGet | object | `{"path":"/api/health"}` | HTTP GET request configuration for liveness probe |
| portal-frontend.livenessProbe.httpGet.path | string | `"/api/health"` | Health check endpoint path |
| portal-frontend.metrics.enabled | bool | `false` | Enable metrics collection |
| portal-frontend.metrics.serviceMonitor.enabled | bool | `false` | Enable Prometheus ServiceMonitor for metrics |
| portal-frontend.readinessProbe.enabled | bool | `true` | Enable readinessProbe |
| portal-frontend.readinessProbe.httpGet | object | `{"path":"/api/health"}` | HTTP GET request configuration for liveness probe |
| portal-frontend.readinessProbe.httpGet.path | string | `"/api/health"` | Health check endpoint path |
| portal-frontend.resources.limits.cpu | string | `"1000m"` | Maximum CPU limit for the container |
| portal-frontend.resources.limits.memory | string | `"4Gi"` | Maximum memory limit for the container |
| portal-frontend.resources.requests.cpu | string | `"100m"` | Minimum CPU request for resource scheduling |
| portal-frontend.resources.requests.memory | string | `"0.5Gi"` | Minimum memory request for resource scheduling |

--------------------------------------------------
Generated from chart metadata using [helm-docs](https://github.com/norwoodj/helm-docs): `docker run --rm --volume "$(pwd):/helm-docs" -u $(id -u) jnorwood/helm-docs:latest`
