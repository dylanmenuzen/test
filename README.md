# TomorrowOS Digital Signage — CMS

A content management system for digital signage, built on the
[`@tomorrowos/sdk`](https://www.npmjs.com/package/@tomorrowos/sdk). It runs a
CMS server that pairs with player devices (screens), manages playlists of media,
schedules content, and pushes it to devices over WebSocket — with a built-in web
control panel.

## What you get

- **Web control panel** at `/` — pair devices, upload media, build playlists,
  assign them to screens, and monitor device status/screenshots.
- **Device pairing** — screens pair with a 6-character code; pairings survive
  restarts.
- **Playlists & scheduling** — group media into playlists with optional
  start/end schedules and publish them to specific devices.
- **Persistent storage** — SQLite by default (`./data/tomorrowos.db`), or point
  it at Postgres/Supabase with an env var.

## Quick start

```bash
# 1. Install dependencies
npm install

# 2. (optional) configure environment
cp .env.example .env

# 3. Run the server
npm run dev      # watch mode (auto-restart on changes)
# or
npm start
```

Then open <http://localhost:3000> to use the control panel. The brand endpoint
is served at <http://localhost:3000/brand.json>.

## Project layout

| Path                 | Purpose                                                        |
| -------------------- | -------------------------------------------------------------- |
| `server.ts`          | CMS server entry point — creates the store and starts listening |
| `brand.json`         | Branding + CMS feature flags (name, colors, logo, features)    |
| `public/`            | Built-in web control panel (served as static assets)           |
| `assets/`            | Brand assets (e.g. logo) referenced by `brand.json`            |
| `policy.example.json`| Example of the policy pushed to a device                       |
| `data/`              | SQLite database + local upload cache (gitignored)              |

## Configuration

### Branding

Edit `brand.json` to set your venue name, colors, logo, and which CMS features
are enabled (`contentScheduling`, `bulkCommands`, `proofOfPlay`,
`userManagement`). The logo referenced by `logoPath` is synced into `public/` on
startup.

### Storage

By default the CMS uses a local SQLite file at `./data/tomorrowos.db`. To use
Postgres or Supabase instead, set `SUPABASE_URL` or `DATABASE_URL` in your
environment (see `.env.example`); the store switches drivers automatically.

## How it works

`server.ts` builds a store with `createTomorrowOSStore(...)` and starts a
`TomorrowOS` server. The server:

1. Serves the control panel and brand config over HTTP.
2. Accepts device WebSocket connections and pairing requests.
3. On `device.paired` / `device.online`, pushes the latest playlist policy to
   the device automatically.

Content management (playlists, assignments, publishing) is exposed through the
SDK's `PlaylistCatalog` (`tomorrowos.playlists`) and the device APIs
(`tomorrowos.listDevices()`, `tomorrowos.device(id).sendCommand(...)`,
`tomorrowos.pairing.*`), all driven by the web panel.

## Building a player

To build a signage player for a target platform (e.g. Samsung Tizen):

```bash
npm run build-player   # tomorrowos build --platform tizen
```

## Notes

- This project depends on `@tomorrowos/sdk` (currently `0.9.x`, pre-1.0), so its
  API may change between minor versions. The version is pinned with a caret in
  `package.json`.
- `better-sqlite3` is a native module; if you install with `--ignore-scripts`,
  run `npm rebuild better-sqlite3` afterward.
