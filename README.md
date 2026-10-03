# MailFlow templates for Unraid

Community-maintained templates for the upstream MailFlow frontend and backend. Both use standard Docker bridge networking; no separate Docker network is needed.

## Install

1. Provide PostgreSQL 16+ and Redis 7+ with persistent storage. Create a dedicated MailFlow database and role. Use reachable server addresses and ports; for containers on Unraid, use the Unraid host address and their published host ports.
2. Install the backend using bridge networking. Set **Backend API host port** to an unused host port (default **3220**), mapped to container port **3000**. Keep the internal PORT variable at 3000.
3. Fill in database credentials and the Redis URL. Generate separate SESSION_SECRET and ENCRYPTION_KEY values with `openssl rand -hex 32`. Back up the encryption key; changing it can make stored credentials unreadable.
4. Set backend APP_URL and FRONTEND_URL to the same browser-facing origin, including HTTPS and any nonstandard port, without a trailing slash.
5. Start PostgreSQL and Redis, then the backend.
6. Install the frontend using bridge networking. Set required **BACKEND_HOST** to your Unraid host IPv4 address or resolvable hostname. Set **BACKEND_PORT** to the backend host port chosen above (default **3220**). Do not use localhost or a Docker container name for this bridge configuration.
7. Frontend HTTPS defaults to host port **50443**, and HTTP to **5080**. Certificate storage defaults to `/mnt/user/appdata/mailflow/certs`. Keep it writable; provide `cert.pem` and `key.pem`, or the image creates a self-signed pair.
8. Start the frontend, open its WebUI, create your admin account, and review registration settings. Verify login, API/WebSocket traffic, and persistence after restart.

Use a frontend image containing upstream `set-backend.sh`, which supports BACKEND_HOST and BACKEND_PORT. The managed nginx configuration must be writable for these overrides. Frontend APP_URL is retained as an optional compatibility field; its consumption remains unverified and does not replace backend URL settings.

For an additional TLS reverse proxy, forward to the frontend HTTP port and send `X-Forwarded-Proto: https`. Leave TRUST_PROXY_HOPS blank unless the trusted-proxy access restrictions and hop count have been verified. Keep the backend API port accessible only to trusted clients; do not forward it from the internet.

## Updates and optional features

Configure Google OAuth and VAPID fields only when enabling those features. Supply your own credentials. To pin a release, set matching published image tags in both Repository fields; MAILFLOW_VERSION as a runtime variable does not select the image.

These templates do not create database services or enforce startup health dependencies. Configure Unraid startup order and back up database data and encryption settings.

## Publication and support

Local XML checks passed; this bridge configuration still requires live testing on Unraid. Upload the templates and this README to replace their existing repository files. Existing installed containers may retain old settings: edit them explicitly to select bridge, remove the backend network alias, add the host-port mapping, and set frontend backend address/port.

Repository: https://github.com/harikiran-lab/unraid-templates

Template issues: https://github.com/harikiran-lab/unraid-templates/issues

Application documentation and issues: https://github.com/maathimself/mailflow

Submit and validate through https://ca.unraid.net/submit. Never upload local secrets or personalized exported templates.

## License

Templates, documentation, and the original generic envelope icon are MIT licensed. The icon is not an official MailFlow logo. Upstream MailFlow and its images retain their own licenses.
