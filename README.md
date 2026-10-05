## Quick start (Docker)

**macOS / Linux:**
```bash
git clone https://github.com/ManasJ17/Trackdesk.git
cd Trackdesk
chmod +x setup.sh
./setup.sh
```

**Windows (PowerShell):**
```powershell
git clone https://github.com/ManasJ17/Trackdesk.git
cd Trackdesk
powershell -ExecutionPolicy Bypass -File setup.ps1
```

This will:
1. Generate secure secrets automatically
2. Start PostgreSQL, Redis, and Trackdesk via Docker
3. Wait for the service to be healthy

Then open **http://localhost:3001** and create your admin account.

## Manual Docker setup

```bash
git clone https://github.com/ManasJ17/Trackdesk.git
cd Trackdesk
docker compose -f docker-compose.production.yml up -d
```

Open **http://localhost:3001** and create your admin account. Secrets are auto-generated on first run.

To pin a specific version: `IMAGE_TAG=<version> docker compose -f docker-compose.production.yml up -d` — see the [latest release](https://github.com/gorkem-bwl/Trackdesk/releases/latest) for the current tag.

## HTTPS with Caddy (optional)

```bash
# 1. Set your domain in .env
echo 'Trackdesk_DOMAIN=Trackdesk.yourdomain.com' >> .env

# 2. Point your domain's DNS A record to your server's IP

# 3. Start with HTTPS
docker compose -f docker-compose.production.yml -f docker-compose.https.yml up -d
```

Caddy automatically obtains and renews Let's Encrypt SSL certificates. Ports 80 and 443 must be open.

## Development setup

```bash
# 1. Start PostgreSQL and Redis
docker compose up -d

# 2. Install dependencies
npm install

# 3. Create environment file
cp .env.example .env

# 4. Update .env for local development:
#    - DATABASE_URL=postgresql://postgres:postgres@localhost:5432/Trackdesk
#    - CLIENT_PUBLIC_URL=http://localhost:5180
#    - CORS_ORIGINS=http://localhost:5180,http://localhost:3001
#    The JWT secrets in .env.example are placeholders — generate real ones:
#      openssl rand -hex 32  → paste into JWT_SECRET
#      openssl rand -hex 32  → paste into JWT_REFRESH_SECRET
#      openssl rand -hex 32  → paste into TOKEN_ENCRYPTION_KEY

# 5. Start dev servers
npm run dev
```

- Client: http://localhost:5180
- Server: http://localhost:3001
- On first visit, you'll be prompted to create an admin account.

## API documentation

Trackdesk exposes an OpenAPI 3.1 specification and an interactive reference UI:

- **Raw spec:** `http://localhost:3001/api/v1/openapi.json`
- **Interactive reference** (Scalar): `http://localhost:3001/api/v1/reference`

The spec is generated from Zod schemas in `packages/server/src/openapi/paths/`. When a route adopts `defineRoute()`, the same schema drives both the OpenAPI registration and runtime request validation — so documentation and wire contract cannot drift.

On a self-hosted deployment, replace `localhost:3001` with your Trackdesk domain.

## Apps

<table>
  <tr>
    <td width="50%">
      <img src="docs/screenshots/crm.png" alt="CRM" /><br/>
      <b>CRM</b> — Pipeline, contacts, companies, deals, leads, forecasting, saved views, web-to-lead forms
    </td>
    <td width="50%">
      <img src="docs/screenshots/hr.png" alt="HR" /><br/>
      <b>HRM</b> — Employees, departments, org chart, leave management, attendance
    </td>
  </tr>
  <tr>
    <td>
      <img src="docs/screenshots/projects.png" alt="Work" /><br/>
      <b>Work</b> — Projects, tasks, time tracking, billing, reports, budgets
    </td>
    <td>
      <img src="docs/screenshots/calendar.png" alt="Calendar" /><br/>
      <b>Calendar</b> — Month/week/day/year/agenda views with Google Calendar sync
    </td>
  </tr>
  <tr>
    <td>
      <img src="docs/screenshots/invoices.png" alt="Invoices" /><br/>
      <b>Invoices</b> — Invoice creation, recurring billing, templates, payment tracking
    </td>
    <td>
      <img src="docs/screenshots/sign.png" alt="Agreements" /><br/>
      <b>Agreements</b> — PDF contracts with e-signatures, templates, counterparty linking, sequential signing, audit trail, reminders
    </td>
  </tr>
  <tr>
    <td>
      <img src="docs/screenshots/drive.png" alt="Drive" /><br/>
      <b>Drive</b> — File storage with versioning, sharing, comments, activity log, password-protected links
    </td>
    <td>
      <img src="docs/screenshots/system.png" alt="System" /><br/>
      <b>System</b> — CPU/memory/disk monitoring, email settings, role-based app permissions
    </td>
  </tr>
  </table>

Trackdesk also includes **Draw** — an Excalidraw-based canvas with PDF export, image insertion, and presentation mode.

**Data import.** Bring your CRM data with you. Trackdesk ships with an Odoo importer (Settings → Data import) that ingests `res.partner`, `crm.lead`, and CRM activity CSV exports — mapping companies, contacts, leads, deals, and activities into your tenant in one pass, with per-row validation and a skipped-row report. HubSpot is on the way.


## Tech stack

- **Frontend**: React, TypeScript, Vite, TanStack Query, Zustand
- **Backend**: Express, TypeScript, Drizzle ORM, PostgreSQL
- **Infrastructure**: Docker, Redis, BullMQ

## Environment variables

Trackdesk boots with a small set of **required** secrets. Everything else is optional and only enables specific features — see the *What needs what* table below.

### Required (Trackdesk will not start without these)

| Variable | Description |
|----------|-------------|
| `JWT_SECRET` | JWT signing key. Min 32 chars. Generate: `openssl rand -hex 32` |
| `JWT_REFRESH_SECRET` | Refresh-token signing key. Min 32 chars. Generate: `openssl rand -hex 32` |
| `TOKEN_ENCRYPTION_KEY` | 64-char hex string used to encrypt Google OAuth tokens at rest. Generate: `openssl rand -hex 32` |

> The Docker `setup.sh` / `setup.ps1` scripts generate all three for you on first run. If you use `docker-compose.production.yml` directly, the compose file auto-generates them as well. You only need to set them manually when running Trackdesk outside Docker (e.g. development, or a custom deploy).

### Networking & database (defaults work for most setups)

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `postgresql://postgres:postgres@localhost:5432/Trackdesk` | PostgreSQL connection string |
| `POSTGRES_PASSWORD` | `Trackdesk` | Postgres password when using the bundled Docker compose |
| `REDIS_URL` | *(unset)* | Redis connection. Required **only** for the Google sync background worker; everything else runs without Redis. |
| `PORT` | `3001` | Server port |
| `SERVER_PUBLIC_URL` | `http://localhost:3001` | Publicly reachable URL of the Trackdesk API (used in invitation / password-reset links, and as the default OAuth redirect host) |
| `CLIENT_PUBLIC_URL` | `http://localhost:5180` in dev, same as `SERVER_PUBLIC_URL` in production | Publicly reachable URL of the Trackdesk web app |
| `CORS_ORIGINS` | *(derived from `CLIENT_PUBLIC_URL`)* | Comma-separated origins allowed to call the API |
| `DISABLE_PUBLIC_SIGNUP` | `true` | Public self-registration is **off by default** on every self-hosted instance. The first-run setup flow and tenant invitations always work. Set `DISABLE_PUBLIC_SIGNUP=false` only if you're running Trackdesk as an open SaaS where anyone can create their own workspace. |

## Troubleshooting

**"network ... not found" error**

```bash
docker compose -f docker-compose.production.yml down
docker compose -f docker-compose.production.yml up -d
```

**Port 3001 already in use**

Stop the process using port 3001, or set a different port in `.env`:
```
PORT=3002
```

**Update to latest version**

```bash
docker compose -f docker-compose.production.yml pull
docker compose -f docker-compose.production.yml up -d
```

**Reset everything (fresh start)**

```bash
docker compose -f docker-compose.production.yml down -v
docker compose -f docker-compose.production.yml up -d
```

