# MailFlow templates for Unraid

Community-maintained Docker templates for the [MailFlow](https://github.com/maathimself/mailflow) frontend and backend. MailFlow is a webmail client for existing IMAP/SMTP accounts. These templates use upstream container images; this repository is not the upstream MailFlow project.

## Before installing

- Provide PostgreSQL 16+ and Redis 7+ with persistent storage.
- Create a dedicated PostgreSQL database and user, both named `mailflow` by default.
- Create a user-defined Docker network from the Unraid terminal:

  ```sh
  docker network create mailflow-network
  ```

If that network already exists, reuse it. Both MailFlow containers must join it. Database and Redis containers can join the same network, or use reachable external endpoints.

## Install the backend

1. Install `templates/mailflow-backend.xml` using Unraid's Docker template mechanism.
2. Select `mailflow-network` and retain the `backend` network alias in Extra Parameters.
3. Enter your database hostname, database name, user, and password. For a database on the same network, use its internal port, normally `5432`.
4. Enter your Redis URL, including URL-encoded credentials if required.
5. Generate two independent values with `openssl rand -hex 32`: one for SESSION_SECRET and one for ENCRYPTION_KEY. Store them securely. Do not change an existing encryption key without planning recovery of encrypted credentials.
6. Set APP_URL and FRONTEND_URL to the same browser-facing origin, including HTTPS and any nonstandard port, without a trailing slash.
7. Start PostgreSQL and Redis before the backend. Keep the internal API port at `3000` for the default configuration. No backend host port is published.

## Install the frontend

1. Install `templates/mailflow-frontend.xml` and select `mailflow-network`.
2. Leave BACKEND_HOST as `backend` and BACKEND_PORT as `3000` for the supplied backend. Recent frontend images support changing these to another reachable hostname/IPv4 address and port.
3. The default host ports are `50443` for HTTPS and `5080` for HTTP. Change them if already occupied.
4. Keep the certificate directory writable. It defaults to `/mnt/user/appdata/mailflow/certs`. Provide `cert.pem` and `key.pem`, or the container generates a self-signed pair. Generated certificates are not publicly trusted.
5. Start the frontend after the backend is ready, then open its WebUI. Configure the backend URLs to match the actual browser address.
6. Create your admin account and review registration settings.

Frontend APP_URL is retained as an optional advanced compatibility field. Its consumption by the frontend has not been verified; it does not replace the backend URL settings.

For a TLS-terminating reverse proxy, forward to the frontend HTTP port and send `X-Forwarded-Proto: https`. Leave TRUST_PROXY_HOPS blank unless access is restricted to a trusted proxy and you have verified the required hop count. See the [upstream installation guide](https://github.com/maathimself/mailflow#installation).

## Optional features and updates

Google OAuth and web push fields are optional. Supply your own OAuth application details and matching VAPID key pair/contact when enabling those features.

To pin a release, edit the Repository field in both templates to matching published image tags. Setting a MAILFLOW_VERSION environment variable in Unraid does not change which image Docker pulls.

The default backend alias also works with older images using a fixed `backend:3000` upstream. Custom backend overrides require an image containing `set-backend.sh` and a writable managed nginx configuration.

Unraid individual-container templates do not reproduce Compose health-based startup dependencies. Configure startup order and verify readiness. Back up PostgreSQL, Redis as appropriate, certificates, and encryption settings.

## Publication and validation

This repository contains two templates, not bundled PostgreSQL/Redis installations. XML has been checked locally; a live Unraid installation and Community Applications review are still required. Before submission, test image pulls, installation, login, API/WebSocket traffic, restarts, and persistence.

Submit the public repository through [Community Applications](https://ca.unraid.net/submit), run Validate and Scan, and resolve review findings. Keep `ca_profile.xml` and this license at the repository root. Never commit local passwords, keys, or exported installation templates containing private values.

## Support and license

Report template problems through this repository's Issues tab. Report application problems to [MailFlow upstream](https://github.com/maathimself/mailflow/issues).

The template files, documentation, and original generic envelope icon in this repository are provided under the MIT license. The icon is not an official MailFlow logo. MailFlow itself and its Docker images remain subject to their upstream licenses.
