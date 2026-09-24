# Katta Compose

[![Compose](https://github.com/shift7-ch/katta-compose/actions/workflows/compose.yml/badge.svg)](https://github.com/shift7-ch/katta-compose/actions/workflows/compose.yml)

> [Katta](https://katta.cloud/): transform your S3 storage into a secure, team-friendly workspace with client-side encryption.

Docker Compose environment to run [Katta Server](https://github.com/shift7-ch/katta-server) with Keycloak, PostgreSQL and MinIO.

## Usage

```bash
docker compose --profile demo up --wait
docker compose --profile demo down
```

> [!TIP]
> Open Katta Web at http://hub.localhost:8280 and log in with username `admin` and password `admin`.

> [!WARNING]
> This environment is for development, testing and demos only. It uses well-known passwords and client secrets,
> a committed TLS key for Keycloak, and runs Keycloak and Katta Server in development mode.

### Profiles

| Profile  | Description                                                                                                 |
|----------|-------------------------------------------------------------------------------------------------------------|
| `local`  | Katta Server, Keycloak, PostgreSQL and MinIO.                                                               |
| `demo`   | Same as `local`, and creates storage profiles for MinIO with static and STS storage access in Katta Server. |
| `hybrid` | Katta Server and PostgreSQL only, using an existing Keycloak and MinIO configured in the env file.          |

### Configuration

All variables are set in [`.env`](.env), which Compose loads automatically.
To provide your own values, pass a complete copy with `--env-file`, which replaces `.env`.

| Variable                                         | Default                                           | Description                                                                                                                                    |
|--------------------------------------------------|---------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| `KATTA_CHART`                                    | `oci://ghcr.io/shift7-ch/katta-helm/katta-server` | Helm chart of Katta Server to render the Keycloak realm from. See [Keycloak Realm](#keycloak-realm).                                           |
| `KATTA_CHART_VERSION`                            | `^1`                                              | Version or version range of the Helm chart. Keep at the version of `KATTA_SERVER_IMAGE`. Leave empty for the latest version.                   |
| `SETUP_DIR`                                      | `./setup`                                         | Directory with the MinIO policies, using the layout of [`setup`](setup).                                                                       |
| `KATTA_SERVER_IMAGE`                             | `ghcr.io/shift7-ch/katta-server:latest`           | Image to build Katta Server from.                                                                                                              || `KATTA_COMPOSE_VERSION`                          | `local`                                       | Tag of the images built from this project. Set to a released version to pull the published images.                                             |
| `HUB_INITIAL_LICENSE`, `HUB_INITIAL_ID`          |                                                   | License of Katta Server. [`.env`](.env) sets a test license.                                                                                   |
| `CSP_CONNECT_SRC_EXTRA`                          |                                                   | Additional `connect-src` sources for the Content-Security-Policy header of Katta Server, such as the S3 and STS endpoints of storage profiles. |

Relative paths resolve against the directory containing `compose.yaml`.
Use absolute paths to provide setup files from another project.

#### Keycloak Realm

The service `keycloak-realm` renders the realm with `helm template` from the realm template
`_realm.tpl` of the Katta Server Helm chart from [katta-helm](https://github.com/shift7-ch/katta-helm)
using the variables of the env file, and Keycloak imports it on start. There is no realm file in this project.
To render the realm from a local checkout of katta-helm instead, mount it into `keycloak-realm` with a
[Compose override file](https://docs.docker.com/compose/how-tos/multiple-compose-files/merge/) and set `KATTA_CHART` to the mount path.

After the import, the service `keycloak-allow-http` sets `sslRequired` to `NONE` for the `master` and the Katta realm, so that
Keycloak accepts plain HTTP requests from the host. The realm does not enable direct access grants.
The `demo` profile creates the storage profiles with the service account of client `cryptomatorhub-system`.

#### Changing Variables of a Running Environment

Compose resolves variables when it creates a container, so `docker compose restart` keeps the previous values.
For example, after changing `CSP_CONNECT_SRC_EXTRA`, recreate only Katta Server with the profile the environment was started with:

```bash
docker compose --profile demo up -d --no-deps hub
```

To verify the Content-Security-Policy header, run:

```bash
curl -sI http://hub.localhost:8280/ | grep -i content-security-policy
```

### Provisioned Users

| User         | Password     | Description                                           |
|--------------|--------------|-------------------------------------------------------|
| `admin`      | `admin`      | Katta administrator and Keycloak realm administrator. |
| `minioadmin` | `minioadmin` | MinIO root user.                                      |
| `testuser`   | `top-secret` | MinIO user for static storage access.                 |

The realm also contains the service account of the `cryptomatorhub-system` client used by Katta Server.

### Endpoints

| Component     | URL                            | Discovery                                                                          |
|---------------|--------------------------------|------------------------------------------------------------------------------------|
| Katta Web     | http://hub.localhost:8280      |                                                                                    |
| Katta API     | http://hub.localhost:8280      | http://hub.localhost:8280/api/config                                               |
| Keycloak      | http://keycloak.localhost:8380 | http://keycloak.localhost:8380/realms/cryptomator/.well-known/openid-configuration |
| MinIO Console | http://minio.localhost:9101    |                                                                                    |
| MinIO S3 API  | http://minio.localhost:9100    |                                                                                    |

The hostnames are subdomains of `localhost`, which resolve to the loopback address on the host as specified in
[RFC 6761](https://www.rfc-editor.org/rfc/rfc6761.html#section-6.3), and to the containers through network aliases
inside the Docker network. Therefore, the same URLs work in the browser on the host and in the containers, such as for the issuer of tokens.
Browsers and recent versions of curl resolve subdomains of `localhost` without DNS, but the system resolver of macOS does not. For other clients on the host, add the hostnames to `/etc/hosts`:

```
127.0.0.1 hub.localhost keycloak.localhost minio.localhost
```

> [!TIP]
> To access with Katta Desktop over plain HTTP (no HTTPS/TLS required) in a development or test environment,
install the _Katta Server (HTTP)_ connection profile from _Preferences → Profiles_.

### MinIO STS Setup

To configure MinIO for STS storage access with the `local` profile, use the `setup minio` command of the
[Katta Admin CLI](https://github.com/shift7-ch/katta-clientlib/blob/main/admin-cli/README.md#setup-minio-using-oidc-provider-and-security-token-service-sts-with-setup-command):

```bash
katta setup minio --hubUrl http://hub.localhost:8280 --endpointUrl http://minio.localhost:9100 --accessKey=minioadmin --secretKey=minioadmin
```

## Contents

| Path                                                     | Description                                                                                                             |
|----------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| [`compose.yaml`](compose.yaml)                           | Services for Katta Server, Keycloak, PostgreSQL and MinIO.                                                              |
| [`.env`](.env)                                           | Variables for running all services locally.                                                                             |
| [`hub`](hub)                                             | Image for Katta Server.                                                                                                 |
| [`minio`](minio)                                         | Image for MinIO.                                                                                                        |
| [`minio-setup`](minio-setup)                             | Image for the jobs configuring and tracing MinIO.                                                                       |
| [`keycloak`](keycloak)                                   | Self-signed certificate for HTTPS of Keycloak. For development only.                                                    |
| [`setup`](setup)                                         | Default MinIO policies.                                                                                                 |

## License

Licensed under the [GNU Affero General Public License v3.0](LICENSE).
