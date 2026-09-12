# Katta Compose

Docker Compose environment to run [Katta Server](https://github.com/shift7-ch/katta-server) with Keycloak, PostgreSQL and MinIO.

Extracted with its history from [katta-clientlib](https://github.com/shift7-ch/katta-clientlib), where it served as the integration test environment.

> [!NOTE]
> The compose file still expects the Keycloak realm, the setup files and the env file provided by katta-clientlib.
> These will be made configurable so the environment runs on its own.

## Contents

| Path                                                               | Description                                                                              |
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| [`compose.yml`](compose.yml) | Services for Katta Server, Keycloak, PostgreSQL and MinIO with the `local`, `demo` and `hybrid` profiles. |
| [`docker`](docker)                                                 | Images for Katta Server, MinIO and the setup jobs, and nginx reverse proxy templates.    |
| [`certs`](certs)                                                   | Self-signed certificate for Keycloak HTTPS. For development only.                         |

## License

Licensed under the [GNU Affero General Public License v3.0](LICENSE).
