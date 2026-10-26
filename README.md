# GT Global: shipping site and back end

A single small server that:

- serves the GT Global website
- lets customers **register a package** and get a tracking code
- lets customers **track** a package by code
- gives staff an **admin page** to view packages and update their status

No npm packages to install. It uses Node's built-in SQLite.

## Run it

You need **Node.js 22.13 or newer** (https://nodejs.org).

```bash
cd gt-global
ADMIN_TOKEN="choose-a-long-random-password" npm start
```

On Windows (PowerShell):

```powershell
$env:ADMIN_TOKEN="choose-a-long-random-password"; npm start
```

Then open:

- Website: http://localhost:3000
- Admin: http://localhost:3000/admin (sign in with your `ADMIN_TOKEN`)

If you don't set `ADMIN_TOKEN`, a random one is printed when the server starts.

## Settings (environment variables)

| Name | Default | What it does |
| --- | --- | --- |
| `ADMIN_TOKEN` | random each run | Password for the admin page and admin API |
| `PORT` | 3000 | Port to listen on |
| `DATA_DIR` | `./data` | Where the database file (`gtglobal.db`) is kept |
| `TRUST_PROXY` | off | Set to `1` if running behind a reverse proxy (nginx, Caddy, a hosting platform) so rate limits use the real visitor address |

## API

Public:

- `POST /api/shipments` registers a package. Returns `{ trackingCode }`.
  Fields: `service` (`shipping` or `consignment`), `senderName`, `senderPhone`, `senderEmail` (optional), `pickupAddress`, `recipientName`, `recipientPhone`, `recipientAddress`, `description`, `weightKg`, `fragile`.
- `GET /api/track/:code` returns status and history. It never returns names, phone numbers or addresses.

Admin (header `Authorization: Bearer <ADMIN_TOKEN>`):

- `GET /api/admin/shipments?q=&status=&limit=&offset=` lists packages with full details.
- `GET /api/admin/shipments/:code` shows one package.
- `PATCH /api/admin/shipments/:code` updates it. Body: `{ "status": "in_transit", "note": "Left Lagos hub", "receivedBy": "..." }`.
  Statuses: `booked`, `collected`, `in_transit`, `delivered`, `cancelled`. `delivered` needs `receivedBy`. Delivered and cancelled packages are locked.

Notes you add are shown to customers on the tracking page.

## Before going live

- **Use HTTPS.** Put the app behind a host or proxy that provides it. The admin token and customers' details are sent over the connection.
- **Set a strong `ADMIN_TOKEN`** and keep it private.
- **Back up** `data/gtglobal.db` regularly.
- Customer names, phones and addresses are personal data. Check what privacy rules apply to you.
- Registrations are limited to 20 per hour per visitor, and lookups to 60 per minute.
- There is one shared admin password. If several staff need their own logins, that is the next thing to add.
