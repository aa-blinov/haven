# Haven

Haven is a chat server for a group of people who would rather run it themselves. One Node.js process serves the web client, the JSON API and the Socket.IO endpoint, and keeps messages, users and settings in a SQLite file on the machine you started it on. Voice calls go from browser to browser over WebRTC, so the audio never passes through the server.

![The Haven chat window](https://github.com/user-attachments/assets/0c85ca6c-f811-43db-a26b-9b66c418830e)

[![Version](https://img.shields.io/badge/version-2.7.4-blue)](https://github.com/aa-blinov/haven/releases)
[![License](https://img.shields.io/badge/license-MIT--NC-green)](LICENSE)
[![Node](https://img.shields.io/badge/node-%3E%3D18-brightgreen)](https://nodejs.org)
[![Docker image](https://github.com/aa-blinov/haven/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/aa-blinov/haven/actions/workflows/docker-publish.yml)

---

## Quick start

### Docker

```bash
git clone https://github.com/aa-blinov/haven.git
cd haven
docker compose up -d
```

The compose file builds the image from this checkout, keeps the data in the `haven_data` volume, and publishes port 3000 for HTTPS plus port 3001 for the HTTP redirect. Open `https://localhost:3000`, register with the name `admin`, create a channel, and send the eight-character channel code to the people you want in it.

A published image exists as well, at `ghcr.io/ancsemi/haven:latest`. It is built by the upstream project and carries a newer version than this checkout, so pull it only when you want that code.

### The other routes

| Route | Command | What it is for |
| --- | --- | --- |
| Windows launcher | `Start Haven.bat` | Installs Node.js if it is missing, installs the dependencies, generates the certificate and opens the browser. |
| Linux and macOS launcher | `chmod +x start.sh && ./start.sh` | The same steps in a shell. |
| Web installer | `chmod +x install.sh && ./install.sh` | A wizard in the browser that asks for the server name and the admin password. It binds to 127.0.0.1 only. |
| Plain Node | `npm install && node server.js` | Nothing around it: no certificate, no launcher, no wizard. |

The certificate is self-signed, so the browser asks once before it opens the page. Voice chat needs a secure origin: it works on `http://localhost`, and from another machine it needs the HTTPS listener.

## What it does

- **Text channels.** Channels nest one level as sub-channels, and a sub-channel can be private and reachable only by invitation. Each channel has a topic, a type (standard, text only, voice only), a slow mode, and switches for uploads, screen sharing and the music player.
- **Join codes.** Every channel carries an eight-character code. An admin decides whether the code is public or private, whether it stays the same or rotates on a timer or on a join count, and a server-wide code joins every top-level channel at once.
- **Voice and screen sharing.** Calls are peer to peer. Each participant has a volume slider, and there are mute, deafen, talking indicators and synthesized tones when someone joins or leaves. Several people can share their screen or camera at once and the tiles are laid out in a grid.
- **Messages.** Bold, italic, strikethrough, inline code, spoilers, fenced code blocks with a language label and blockquotes. Replies, reactions, pins, edits, polls, typing indicators, unread counts that survive a reload, search with Ctrl+F, and twenty-two slash commands from `/shrug` to `/roll 2d20`.
- **Files.** Images and any other kind of file up to the size the admin sets, 25 MB by default. Avatars come in five shapes, an admin can upload custom emojis and notification sounds, and audio and video get an inline player.
- **Link previews and GIFs.** A pasted URL is fetched on the server and its Open Graph title, description and thumbnail are shown. The GIF picker searches GIPHY and needs an API key.
- **Accounts.** A username and a password, hashed with bcrypt, and a JWT that lasts seven days. Two-factor authentication with TOTP and backup codes, and recovery codes for a lost password, both optional. No email address is ever asked for.
- **Encrypted direct messages.** A key pair is generated in the browser, the private key is wrapped with a key derived from the password, and direct messages are sealed with AES-256-GCM. The server stores the public key and the wrapped blob, and cannot read a message.
- **Moderation.** Kick, timed mute, ban, delete a user, roles with per-channel permissions, a registration whitelist, slow mode, and automatic cleanup of old messages and files.
- **Bot webhooks.** An admin creates a webhook for a channel and posts to it with a 64-character token. The message lands in the channel with the name and avatar of your choosing.
- **Discord import.** A channel history can be brought across from a Discord server, either by connecting with a token or by uploading a DiscordChatExporter JSON file or a Discord data package. Messages, replies, reactions, pins, attachments and avatars are carried over.
- **Plugins and themes.** A file named `something.plugin.js` in `plugins/` becomes a plugin, and a file named `something.theme.css` in `themes/` becomes a theme. The loader lists both, and the admin panel turns them on.
- **Games.** A canvas mini-game with a leaderboard kept in the database, and pages for .io games and for Flash through Ruffle.

## How it works

```
browser  --HTTP/JSON-->  Express (server.js)  -->  SQLite (better-sqlite3)
   |                              |
   +---------Socket.IO------------+
   (chat, voice signalling, admin)
```

One Node.js process does everything. It serves the client from `public/`, the JSON API, and the Socket.IO endpoint that carries chat, presence, voice signalling and the admin panel. There is no build step for the client: `public/app.html` loads `public/js/app.js` as an ES module, and that file pulls in eleven modules from `public/js/modules/` and merges them onto the prototype of one class.

The database is SQLite through better-sqlite3. The schema lives in `src/database.js`, which creates the tables and adds the columns an older install is missing every time the server starts. Users, channels, memberships, messages, reactions, roles, permissions, webhooks, read positions and settings are all rows.

Voice is WebRTC. The server never carries audio: it relays the offer, the answer and the ICE candidates between the participants. The client asks `/api/ice-servers` for its STUN and TURN configuration, and the server answers with two Google STUN servers plus, when the operator has set `TURN_URL`, a coturn relay. With a TURN shared secret the credentials are HMAC-SHA1 and expire after 24 hours.

Push notifications come in two shapes. The browser gets web-push with VAPID keys that are generated on the first boot and written to `.env`. The mobile client gets Firebase Cloud Messaging, which the server reaches directly with a service account, through a relay of your own, or through the shared relay the project runs, depending on what is configured.

Uploads are written to the data directory and served from `/uploads`. Anything that is not a JPEG, PNG, GIF or WebP is sent with `Content-Disposition: attachment`, so a browser cannot execute an uploaded file.

## Configuration

The server reads `.env` from the data directory, and creates it from `.env.example` on the first boot. It replaces the placeholder `JWT_SECRET` with 48 random bytes in base64 and appends a VAPID key pair, both in the same file.

| Variable | Default | What it does |
| --- | --- | --- |
| `PORT` | `3000` | The port the server listens on. The HTTP redirect server takes the next port up, 3001. |
| `HOST` | `0.0.0.0` | The address it binds to. |
| `SERVER_NAME` | `Haven` | The name shown in the sidebar and in the multi-server list. |
| `ADMIN_USERNAME` | `admin` | The username that carries admin rights. It is re-read on every login. |
| `JWT_SECRET` | generated | Signs the session tokens. |
| `SSL_CERT_PATH` | detected | The certificate. A relative path resolves against the data directory. |
| `SSL_KEY_PATH` | detected | The private key. |
| `FORCE_HTTP` | `false` | Serve plain HTTP even when a certificate exists, for a reverse proxy in front. |
| `HAVEN_DATA_DIR` | per system | Where the database, the configuration, the certificates and the uploads live. |
| `TURN_URL` | none | The TURN relay for voice across NAT. |
| `TURN_SECRET` | none | The shared secret of that relay, used for HMAC credentials that expire in 24 hours. |
| `TURN_USERNAME`, `TURN_PASSWORD` | none | Static credentials for that relay, used when there is no shared secret. |
| `GIPHY_API_KEY` | none | Turns the GIF picker on. |
| `VAPID_EMAIL` | `mailto:admin@haven.local` | The contact address in the web-push subscription. |
| `FIREBASE_SERVICE_ACCOUNT` | none | The path to a service account JSON, for mobile push sent directly. |
| `FCM_RELAY_URL`, `FCM_PUSH_KEY` | the shared relay | Mobile push through a relay of your own. |

The data directory holds `haven.db`, `.env`, `certs/` and `uploads/`, and it sits outside the code folder so that copying the checkout never copies a conversation.

| System | Data directory |
| --- | --- |
| Windows | `%APPDATA%\Haven\` |
| Linux, macOS | `~/.haven/` |

The rest of the settings live in the database and are changed from the admin panel: `member_visibility`, `cleanup_enabled`, `cleanup_max_age_days`, `cleanup_max_size_mb`, `max_upload_mb`, `server_name`, `server_icon`, `server_code`, `whitelist_enabled`, `default_theme`, `permission_thresholds` and `giphy_api_key`.

## Development

```bash
npm ci          # install
npm start       # run the server
npm run dev     # run it with node --watch
```

Node 18 to 24. There is no build step and no test suite: the repository has no `test` script, and the workflows that run build the Docker image for amd64 and arm64 and attach a tarball to each release.

| Path | What it holds |
| --- | --- |
| `server.js` | The Express app, the HTTP routes, the upload handlers, the Discord import and the auto-cleanup timer. |
| `src/socketHandlers.js` | Every Socket.IO event: channels, messages, voice signalling, roles, moderation, webhooks. |
| `src/auth.js` | Registration, login, TOTP, recovery codes, the rate limiter and the token helpers. |
| `src/database.js` | The schema, the migrations that run at startup, and the prepared statement cache. |
| `src/paths.js` | Where the data directory is, and the move of old data out of the project folder. |
| `src/tunnel.js` | The localtunnel and cloudflared tunnel that expose the server without port forwarding. |
| `src/fcm.js` | Mobile push, in direct, relay and shared-relay modes. |
| `src/importDiscord.js` | The parser for DiscordChatExporter JSON and for Discord data packages. |
| `public/app.html`, `public/index.html` | The chat client and the login page. |
| `public/js/modules/` | The eleven client modules, one per concern. |
| `public/js/voice.js`, `public/js/e2e.js` | The WebRTC manager and the encryption manager. |
| `installer/` | The local web installer, built on the Node standard library alone. |
| `haven-push-relay/` | The Firebase Cloud Function and a standalone relay for mobile push. |
| `plugins/`, `themes/` | Drop-in plugins and themes, read at runtime. |
| `website/`, `docs/` | The landing page and the guides. |

The client files are imported with a `?v=` query string, and the versions in `public/js/app.js` and `public/app.html` are per file rather than per release. Bump the one you touched, or a browser that cached the module will keep running the old code.

## License

MIT-NC, copyright 2026 the Haven contributors. The MIT terms carry a non-commercial clause: the software may not be sold or monetised without written permission from the original author. See [LICENSE](LICENSE).
