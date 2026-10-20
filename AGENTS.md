# AGENTS.md — agent context for `haven`

Haven is a self-hosted chat server: a Node.js process that serves a browser client, a JSON API and a Socket.IO endpoint, and stores everything in a SQLite file under a data directory outside the code. This file is what you need to work on it without reading all of it.

## Run / build / test

```bash
npm ci                    # install
npm start                 # run the server, port 3000 by default
npm run dev               # run it with node --watch
node installer/server.js  # the local web installer, binds to 127.0.0.1
docker compose up -d      # the container, with the data in the haven_data volume
```

Node 18 to 24. The client has no build step: the browser loads the ES modules in `public/js/` as they are on disk. The repository has no test suite and no `test` script, so a change is checked by running it.

## Where things live

| Path | What it holds |
| --- | --- |
| `server.js` | The Express app, the HTTP routes, the upload handlers, the Discord import and the auto-cleanup timer. |
| `src/socketHandlers.js` | Every Socket.IO event: channels, messages, voice signalling, roles, moderation, webhooks. |
| `src/auth.js` | Registration, login, TOTP, recovery codes, the rate limiter and the token helpers. |
| `src/database.js` | The schema, the migrations that run at startup and the prepared statement cache. |
| `src/paths.js` | Where the data directory is, and the move of old data out of the project folder. |
| `src/tunnel.js` | The localtunnel and cloudflared tunnels that expose the server without port forwarding. |
| `src/fcm.js` | Mobile push, in direct, relay and shared-relay modes. |
| `src/importDiscord.js` | The parser for DiscordChatExporter JSON and for Discord data packages. |
| `public/index.html` | The login page. |
| `public/app.html` | The chat client. |
| `public/js/app.js` | The client class and the list of modules merged onto it. |
| `public/js/modules/` | The eleven client modules, one per concern. |
| `public/js/voice.js` | The WebRTC manager: peers, volumes, screen share, webcam, soundboard. |
| `public/js/e2e.js` | The key handling and the message sealing for direct messages. |
| `public/js/plugin-loader.js` | The loader for `plugins/*.plugin.js` and `themes/*.theme.css`. |
| `installer/` | The web installer, built on the Node standard library alone. |
| `haven-push-relay/` | The Firebase Cloud Function, and a standalone relay for mobile push. |
| `plugins/`, `themes/` | Drop-in plugins and themes, read at runtime. The `themes` directory is created empty on first boot. |
| `website/`, `docs/` | The landing page and the guides. `docs/index.html` and `website/index.html` are kept identical. |

## Traps

- **The data directory sits outside the code.** `src/paths.js` resolves it to `%APPDATA%\Haven` on Windows and `~/.haven` on Linux and macOS, and `HAVEN_DATA_DIR` overrides it. The database, the `.env`, the certificates and the uploads all live there, and the old in-folder copies are moved out on the first boot. A path that reaches the database from `__dirname` is wrong.
- **The schema is migrated by the server, on every boot.** `src/database.js` adds a column by running a SELECT that fails on an old database and catching the error. There is no migration table, and there is no way back: the database moves forward only.
- **`db.prepare` is replaced with a cache.** `initDatabase` swaps in a function that reuses one statement per SQL string, and evicts the older half when the cache passes 500 entries. SQL with an `IN` list produces a new string every time and pushes the cache out.
- **Admin rights are read from the database, not from the token.** The JWT carries `isAdmin`, and every privileged route re-reads `users.is_admin` before it acts. `ADMIN_USERNAME` is compared again on every login, so changing it in `.env` demotes the previous admin the next time they log in, and any registration under that name becomes an admin.
- **A channel code is the join secret.** It is eight hex characters, and the server-wide code joins every top-level channel at once. Rotating a code replaces it in the database, so a client that holds the old code gets "Invalid channel code".
- **SQLite writes UTC without the `Z`.** `CURRENT_TIMESTAMP` produces a bare UTC string and `utcStamp` adds the suffix. A new query that formats `created_at` by hand will be read by the browser as local time, and the message will look a day out.
- **The client is a set of mixins.** `public/js/app.js` defines one class and merges eleven modules onto its prototype with `Object.assign`. A method name that appears in two modules silently wins by load order, so keep them unique.
- **The client files carry a `?v=` query string.** The versions in `public/js/app.js` and `public/app.html` are per file and do not follow the release version. Bump the one you changed, or a browser that cached the module keeps the old code.
- **Inline scripts do not run.** The Content Security Policy that `helmet` sets allows `unsafe-eval` for scripts but not `unsafe-inline`, so an inline event handler in the markup is refused by the browser. The client attaches its listeners in JavaScript.
- **An uploaded file that is not an image is sent as an attachment.** The `/uploads` handler sets `Content-Disposition: attachment` for everything that is not a JPEG, PNG, GIF or WebP, and the image routes check the magic bytes of the file before they store it. Both halves have to stay.
- **The database pragmas are small on purpose.** The page cache is 8 MB, the memory map is 32 MB and the hard heap limit is 64 MB, because the same server runs next to Electron in the desktop client. Raising them brings back the crash they were set to avoid.
- **An old copy of the client is still in the repository.** `public/js/app.js.bak` is the single-file client from before the split into modules, and `express.static` serves it at `/js/app.js.bak`.
- **The update banner watches the upstream repository.** `_checkForUpdates` in `public/js/modules/app-platform.js` reads the version from `/api/version` and compares it with the newest release published by ancsemi/Haven on GitHub, every thirty minutes. In a fork the banner therefore follows the original project's releases.

## Style / safety rails

- **No user data in the project folder.** The data directory rule is the reason the project can be copied, forked and pushed without taking a conversation with it.
- **A new client feature goes in the module that owns the concern.** `public/js/app.js` holds the class and the module list, and nothing else.
- **The server answers with a short JSON error and no stack trace.** The catch-all returns `{"error":"Not found"}` and the error handler logs the cause and returns a generic message. A stack trace in a response is a bug.
- **The process stays up.** `uncaughtException` and `unhandledRejection` are caught and logged so that one bad socket handler cannot take the server down with it.
- **Comments explain why.** The codebase is good at this, especially in `src/database.js` and `src/socketHandlers.js`. A comment that repeats the line below it is noise.

## Where the product truth lives

`PRODUCT.md` holds it. Read it before you change what the product does, and write the change there when the behaviour changes.
