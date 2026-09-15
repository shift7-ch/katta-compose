# Katta Compose

Docker Compose environment to run [Katta Server](https://github.com/shift7-ch/katta-server) with Keycloak, PostgreSQL and MinIO.

Extracted with its history from [katta-clientlib](https://github.com/shift7-ch/katta-clientlib), where it served as the integration test environment.

## Usage

```bash
docker compose --profile demo up --wait
docker compose --profile demo down
```

> [!TIP]
> Open Katta Web at http://localhost:8280 and log in with username `admin` and password `admin`.

### Profiles

| Profile  | Description                                                                                                 |
|----------|-------------------------------------------------------------------------------------------------------------|
| `local`  | Katta Server, Keycloak, PostgreSQL and MinIO.                                                               |
| `demo`   | Same as `local`, and creates storage profiles for MinIO with static and STS storage access in Katta Server. |
| `hybrid` | Katta Server and PostgreSQL only, using an existing Keycloak and MinIO configured in the env file.          |

### Configuration

All variables are set in [`.env`](.env), which Compose loads automatically.
To provide your own values, pass a complete copy with `--env-file`, which replaces `.env`.

| Variable                                         | Default                                 | Description                                                                                                                                    |
|--------------------------------------------------|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| `KEYCLOAK_REALM_FILE`                            | `./keycloak/cryptomator-realm.json`     | Realm imported into Keycloak. Its name must match `HUB_KEYCLOAK_REALM`.                                                                        |
| `SETUP_DIR`                                      | `./setup`                               | Directory with the MinIO policies and storage profiles, using the layout of [`setup`](setup).                                                  |
| `KATTA_SERVER_IMAGE`                             | `ghcr.io/shift7-ch/katta-server:latest` | Image to build Katta Server from.                                                                                                              |
| `MINIO_USER_ACCESS_KEY`, `MINIO_USER_SECRET_KEY` |                                         | MinIO user created for the storage profile with static storage access.                                                                         |
| `HUB_INITIAL_LICENSE`, `HUB_INITIAL_ID`          |                                         | License of Katta Server. [`.env`](.env) sets a test license.                                                                                   |
| `CSP_CONNECT_SRC_EXTRA`                          |                                         | Additional `connect-src` sources for the Content-Security-Policy header of Katta Server, such as the S3 and STS endpoints of storage profiles. |

Relative paths resolve against the directory containing `compose.yaml`.
Use absolute paths to provide a realm or setup files from another project.

### Provisioned Users

| User                                      | Password     | Description                                                   |
|-------------------------------------------|--------------|---------------------------------------------------------------|
| `admin`                                   | `admin`      | Katta administrator and Keycloak realm administrator.         |
| `minioadmin`                              | `minioadmin` | MinIO root user.                                              |
| `testuser`                                | `top-secret` | MinIO user for static storage access.                         |

The realm also contains the service accounts of the `cryptomatorhub-system` client used by Katta Server and of the `cryptomatorhub-cli` client used by the Katta Admin CLI.

### Endpoints

| Component     | URL                   | Discovery                                                                 |
|---------------|-----------------------|---------------------------------------------------------------------------|
| Katta Web     | http://localhost:8280 |                                                                           |
| Katta API     | http://localhost:8280 | http://localhost:8280/api/config                                          |
| Keycloak      | http://localhost:8380 | http://localhost:8380/realms/cryptomator/.well-known/openid-configuration |
| MinIO Console | http://localhost:9101 |                                                                           |
| MinIO S3 API  | http://localhost:9100 |                                                                           |

> [!TIP]
> To access with Katta Desktop over plain HTTP (no HTTPS/TLS required) in a development or test environment,
install the _Katta Server (HTTP)_ connection profile from _Preferences → Profiles_.

### MinIO STS Setup

To configure MinIO for STS storage access with the `local` profile, use the `setup minio` command of the
[Katta Admin CLI](https://github.com/shift7-ch/katta-clientlib/blob/main/admin-cli/README.md#setup-minio-using-oidc-provider-and-security-token-service-sts-with-setup-command):

```bash
katta setup minio --hubUrl http://localhost:8280 --endpointUrl http://localhost:9100 --accessKey=minioadmin --secretKey=minioadmin
```

## Contents

| Path                                                     | Description                                                                                                             |
|----------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| [`compose.yaml`](compose.yaml)                           | Services for Katta Server, Keycloak, PostgreSQL and MinIO.                                                              |
| [`.env`](.env)                                           | Variables for running all services locally.                                                                             |
| [`hub`](hub)                                             | Image for Katta Server with an nginx reverse proxy.                                                                     |
| [`hub-setup-storage-profile`](hub-setup-storage-profile) | Image for the job creating the demo storage profiles in Katta Server.                                                   |
| [`minio`](minio)                                         | Image for MinIO with an nginx reverse proxy.                                                                            |
| [`minio-setup`](minio-setup)                             | Image for the jobs configuring and tracing MinIO.                                                                       |
| [`nginx`](nginx)                                         | Reverse proxy templates for Keycloak, Katta Server and MinIO.                                                           |
| [`keycloak`](keycloak)                                   | Default realm with the clients required by Katta Server, and a self-signed certificate for HTTPS. For development only. |
| [`setup`](setup)                                         | Default MinIO policies and storage profiles.                                                                            |

## License

Licensed under the [GNU Affero General Public License v3.0](LICENSE).
