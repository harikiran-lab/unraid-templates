# MailFlow for Unraid

MailFlow is a self-hosted webmail client for your existing IMAP and SMTP accounts. These community-maintained Unraid templates install its backend API and frontend web interface using the upstream Docker images.

Both containers use **bridge** networking. No custom Docker network is required.

## Requirements

- PostgreSQL 16 or newer, with a dedicated MailFlow database and user.
- Redis 7 or newer.
- Persistent storage for your database, Redis, and TLS certificates.
- A browser-facing HTTPS address, either through the frontend or your reverse proxy.

Install PostgreSQL and Redis separately before setting up MailFlow. For services running in bridge-mode containers on Unraid, use your Unraid server address and each service's mapped host port.

## 1. Install the backend

In Unraid Apps, locate **MailFlow-backend** and open its installation settings. Leave Network Type set to **Bridge**.

Fill in the following settings:

| Setting | What to enter |
| --- | --- |
| PostgreSQL host | Your database server address, or the Unraid server address if PostgreSQL has a published host port. |
| PostgreSQL port | The port reachable at that address. PostgreSQL normally listens internally on 5432; its mapped host port may differ. |
| Database name and user | The dedicated database and user you created for MailFlow. |
| Database password | The password for that database user. |
| Redis connection URL | Your reachable Redis endpoint, including URL-encoded credentials if authentication is enabled. |
| Application URL and Frontend origin | The same URL users will open, including `https://` and any nonstandard port, without a trailing slash. |
| Session secret | A unique random secret. |
| Credential encryption key | A separate random encryption key. |

Run this command twice in the Unraid terminal to generate two independent values, one for each secret:

```sh
openssl rand -hex 32
```

Store the encryption key securely with your backups. Losing or changing it can make saved email credentials unreadable.

### Backend ports

MailFlow's backend listens on **container port 3000**. Keep the internal `PORT` setting at `3000`.

**Backend API host port** is the configurable port published on your Unraid server. Choose an unused port. The supplied template currently prefills `3220`; this is a host-port choice, not MailFlow's internal port.

| Your chosen host port | Docker mapping | Frontend BACKEND_PORT |
| --- | --- | --- |
| 3000 | Host 3000 → container 3000 | 3000 |
| 3220 | Host 3220 → container 3000 | 3220 |

Start PostgreSQL and Redis first, then start the backend.

## 2. Install the frontend

Install **MailFlow-frontend** and leave Network Type set to **Bridge**.

| Setting | What to enter |
| --- | --- |
| Backend host address (`BACKEND_HOST`) | Your Unraid server's reachable IPv4 address or resolvable hostname. Do not enter `localhost` or the backend container name. |
| Backend port (`BACKEND_PORT`) | The **host port you selected for the backend**, as shown above. |
| HTTPS web interface | An available host port for HTTPS; the template prefills 50443. |
| HTTP reverse-proxy port | An available host port for a TLS-terminating reverse proxy; the template prefills 5080. |
| TLS certificate storage | A writable persistent directory; the template uses `/mnt/user/appdata/mailflow/certs`. |

If you fill in the optional frontend Application URL field, use the same browser-facing address as the backend. Always configure the backend Application URL and Frontend origin fields.

Use a recent frontend image that supports `BACKEND_HOST` and `BACKEND_PORT`.

Start the frontend and open its **WebUI**. Ensure the browser address matches the URL configured in the backend. Create your administrator account, review registration settings, and add your email accounts.

## HTTPS and reverse proxies

For direct HTTPS, place your certificate and private key in the certificate directory as `cert.pem` and `key.pem`. If they are absent, the container generates a self-signed pair, which browsers will not trust automatically.

For your own TLS-terminating reverse proxy, route traffic to the frontend's mapped HTTP port and forward `X-Forwarded-Proto: https`. Leave `TRUST_PROXY_HOPS` blank unless you have restricted access to your trusted proxy and confirmed the correct hop count. See the [upstream setup documentation](https://github.com/maathimself/mailflow).

Keep the backend API port accessible only to trusted clients; do not forward it from the internet.

## Optional features

Google OAuth and web push notifications are optional. Configure the Google OAuth fields with your own application credentials. For web push, provide a matching VAPID public/private key pair and your contact value.

## Updates and backups

Update the frontend and backend together through Unraid. To use a specific release, select corresponding published image tags in each container's Repository field.

Configure startup order so PostgreSQL and Redis start before the backend, followed by the frontend. Back up the database, encryption key, and certificates, along with any required Redis data.

### Moving from an older custom-network installation

Existing containers may retain their previous settings. To switch to this bridge configuration:

1. Change both containers to **Bridge**.
2. Remove `--network-alias backend` from the backend's Extra Parameters.
3. Add the backend TCP mapping from your chosen host port to container port `3000`.
4. Set frontend `BACKEND_HOST` to your Unraid server address and `BACKEND_PORT` to that chosen host port.
5. Update database and Redis endpoints to addresses and ports reachable from bridge networking.
6. Apply the changes and confirm login and email access work.

Keep your existing secrets and database settings when migrating.

## Troubleshooting

If the frontend loads but cannot reach the backend, check that `BACKEND_HOST` is reachable, `BACKEND_PORT` matches the published host port, and the backend is running. Check backend logs for database or Redis connection errors. Older frontend images may need updating to support the backend address settings.

## Support

- [Template support](https://github.com/harikiran-lab/unraid-templates/issues)
- [MailFlow documentation](https://github.com/maathimself/mailflow)
- [MailFlow application issues](https://github.com/maathimself/mailflow/issues)

## License

These templates, documentation, and the original generic envelope icon are MIT licensed. The icon is not an official MailFlow logo. MailFlow and its Docker images retain their upstream licenses.
