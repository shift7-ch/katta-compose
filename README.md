# Katta Compose

Docker Compose environment to run [Katta Server](https://github.com/shift7-ch/katta-server) with Keycloak, PostgreSQL and MinIO.

Extracted with its history from [katta-clientlib](https://github.com/shift7-ch/katta-clientlib), where it served as the integration test environment.

## Usage

```bash
docker compose --env-file local.env --profile demo up --wait
docker compose --env-file local.env --profile demo down
```

> [!TIP]
> Open Katta Web at http://localhost:8280 and log in with username `admin` and password `admin`.

### Profiles

| Profile  | Description                                                                                                  |
|----------|--------------------------------------------------------------------------------------------------------------|
| `local`  | Katta Server, Keycloak, PostgreSQL and MinIO.                                                                |
| `demo`   | Same as `local`, and creates storage profiles for MinIO with static and STS storage access in Katta Server. |
| `hybrid` | Katta Server and PostgreSQL only, using an existing Keycloak and MinIO configured in the env file.           |

### Configuration

All variables are set in [`local.env`](local.env). Copy it to provide your own values.

| Variable                                         | Default                                 | Description                                                                          |
|--------------------------------------------------|-----------------------------------------|--------------------------------------------------------------------------------------|
| `KEYCLOAK_REALM_FILE`                            | `./keycloak/cryptomator-realm.json`     | Realm imported into Keycloak. Its name must match `HUB_KEYCLOAK_REALM`.              |
| `SETUP_DIR`                                      | `./setup`                               | Directory with the MinIO policies and storage profiles, using the layout of [`setup`](setup). |
| `KATTA_SERVER_IMAGE`                             | `ghcr.io/shift7-ch/katta-server:latest` | Image to build Katta Server from.                                                    |
| `MINIO_USER_ACCESS_KEY`, `MINIO_USER_SECRET_KEY` |                                         | MinIO user created for the storage profile with static storage access.               |

Relative paths resolve against the directory containing `compose.yml`.
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

## Contents

| Path                         | Description                                                                           |
|------------------------------|---------------------------------------------------------------------------------------|
| [`compose.yml`](compose.yml) | Services for Katta Server, Keycloak, PostgreSQL and MinIO.                            |
| [`local.env`](local.env)     | Variables for running all services locally.                                           |
| [`docker`](docker)           | Images for Katta Server, MinIO and the setup jobs, and nginx reverse proxy templates. |
| [`keycloak`](keycloak)       | Default realm with the clients required by Katta Server.                              |
| [`setup`](setup)             | Default MinIO policies and storage profiles.                                          |
| [`certs`](certs)             | Self-signed certificate for Keycloak HTTPS. For development only.                     |

## License

Licensed under the [GNU Affero General Public License v3.0](LICENSE).
