# Architecture

How haven is put together.

One Node.js process serves the browser client, the JSON API and the
Socket.IO endpoint, and keeps everything in one SQLite file outside the
code folder. There is no build step for the client and no separate worker.
The browser loads the ES modules in `public/js/` as they are on disk.

## The shape

The server is a small set of files with sharp jobs. The client is one class
with eleven mixins merged onto its prototype. Both halves are described
here, because the boundary between them is where the interesting decisions
are.

| Path | What it is responsible for |
| --- | --- |
| `server.js` | The Express app, the HTTP routes, the upload handlers, the Discord import and the auto-cleanup timer. |
| `src/socketHandlers.js` | Every Socket.IO event: channels, messages, voice signalling, roles, moderation, webhooks. |
| `src/auth.js` | Registration, login, TOTP, recovery codes, the rate limiter and the token helpers. |
| `src/database.js` | The schema, the migrations that run at startup, and the prepared statement cache. |
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
| `public/css/style.css` | The token vocabulary and the twenty-six themes. |
| `installer/` | The web installer, built on the Node standard library alone. |
| `haven-push-relay/` | The Firebase Cloud Function and a standalone relay for mobile push. |
| `plugins/`, `themes/` | Drop-in plugins and themes, read at runtime. The `themes` directory is created empty on the first boot. |
| `website/`, `docs/` | The landing page and the guides. `docs/index.html` and `website/index.html` are kept identical. |

## The data flow

A message from one browser to another:

```
sender -> socket.io 'message:send' -> socketHandlers.js
  -> validation against the channel and the slow mode
  -> SQLite insert
  -> socket.io 'message:new' -> every client in the channel
  -> unread counters on the clients that were not looking at it
```

A file:

```
browser -> multipart POST /api/upload -> multer -> magic bytes checked
  -> written into the data directory
  -> served from /uploads, with Content-Disposition: attachment
    for anything that is not a JPEG, PNG, GIF or WebP
```

Voice:

```
browser A -> socket.io offer -> server relays -> browser B
browser A -> socket.io answer -> server relays -> browser B
browser A <-> browser B, media path, the server is not in it
```

The client asks `/api/ice-servers` for its STUN and TURN configuration and
gets two Google STUN servers back, plus a coturn relay when `TURN_URL` is
set. With a shared secret the credentials are HMAC-SHA1 and expire after 24
hours.

## Decisions worth knowing about

### The data directory sits outside the code

`src/paths.js` resolves it to `%APPDATA%\Haven` on Windows and `~/.haven`
on Linux and macOS, and `HAVEN_DATA_DIR` overrides it. The database, the
`.env`, the certificates and the uploads all live there, and the older
in-folder copies are moved out on the first boot.

The reason is that the checkout can then be copied, forked or thrown away
without taking a conversation with it. A path that reaches the database
from `__dirname` breaks that.

### The schema is migrated by the server, on every boot

`src/database.js` adds a column by running a SELECT that fails on an old
database and catching the error. There is no migration table and no way
back: the database moves forward only, and a column that is dropped by
hand has to be added back by hand.

### The prepared statement cache evicts half of itself

`initDatabase` replaces `db.prepare` with a function that keeps one
statement per SQL string and drops the older half of the cache when it
passes 500 entries. A query with an `IN` list produces a new string every
time and pushes the cache out, which is the reason the permission queries
are written the way they are.

### Admin rights are read from the database, not from the token

The JWT carries `isAdmin`, and every privileged route re-reads
`users.is_admin` before it acts. `ADMIN_USERNAME` is compared again on
every login, so changing it in `.env` demotes the previous admin the next
time they log in, and any registration under that name becomes an admin.

### The client is a set of mixins

`public/js/app.js` defines one class and merges eleven modules onto its
prototype with `Object.assign`. A method name that appears in two modules
wins silently by load order, so the names are kept unique across the
eleven files.

### The client files carry a `?v=` query string

The versions in `public/js/app.js` and `public/app.html` are per file and
do not follow the release version. A browser that cached the module keeps
running the old code until the query string changes.

### Inline scripts do not run

The Content Security Policy that `helmet` sets allows `unsafe-eval` for
scripts and not `unsafe-inline`, so an inline event handler in the markup
is refused by the browser. The client attaches its listeners in JavaScript,
and a new one goes into the module that owns the concern.

### A file that is not an image is sent as an attachment

The `/uploads` handler sets `Content-Disposition: attachment` for anything
that is not a JPEG, PNG, GIF or WebP, and the image routes check the magic
bytes of the file before they store it. Both halves have to stay: the first
stops a browser from running an uploaded file, the second stops a file
that lies about its type from being stored as an image.

### The database pragmas are small on purpose

The page cache is 8 MB, the memory map is 32 MB and the hard heap limit is
64 MB, because the same server runs next to Electron in the desktop
client. Raising them brings back the crash they were set to avoid.

### Themes are custom properties, not a build step

Each theme in `public/css/style.css` is a block of custom properties under
a `[data-theme="name"]` selector, and `public/js/theme.js` sets the
attribute on the root element. A theme that leaves a token out inherits the
default theme's value for it, which is why the themes fill the whole
vocabulary.

### The update banner watches the upstream repository

`_checkForUpdates` in `public/js/modules/app-platform.js` reads the version
from `/api/version` and compares it with the newest release published by
`ancsemi/Haven` on GitHub, every thirty minutes. In a fork the banner
therefore follows the original project's releases.

## Boundaries

- **No user data in the project folder.** The data directory rule is the
  reason the project can be copied, forked and pushed without taking a
  conversation with it.
- **The client reaches the server through two doors and no more.** HTTP for
  the REST calls and the uploads, Socket.IO for chat, presence, voice
  signalling and the admin panel. A new feature picks one of them.
- **`src/` may not require from `public/` and the client may not require
  from `src/`.** The server is Node and the client is the browser, and the
  boundary between them is the API.
- **The server answers with a short JSON error and no stack trace.** The
  catch-all returns `{"error":"Not found"}` and the error handler logs the
  cause and returns a generic message.
- **The process stays up.** `uncaughtException` and `unhandledRejection`
  are caught and logged so that one bad socket handler cannot take the
  server down with it.
- **The installer may not take a dependency.** It runs on the Node
  standard library alone, because it has to work before anything is
  installed.
