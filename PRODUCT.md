# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

The server is Node.js 18 to 24: Express for the HTTP routes, Socket.IO for chat, presence, voice signalling and the admin panel, and better-sqlite3 for storage. The client is a browser page served by the same process, with a web manifest and a service worker, so it installs on a phone as a web application. A container image is published for Linux on amd64 and arm64. Two native clients live outside this repository: a desktop application in beta and an Android application in closed beta. This repository is a fork of ancsemi/Haven, which is where the release history and the issue tracker live.

## Users

The repository description says it in one line: a Discord alternative that runs on your own machine, with no cloud, no telemetry and no account with a large company.

The README names four groups: small friend groups who want a private place to talk, self-hosters who run services on their own hardware, communities that have finished with hosted platforms, and gaming crews on a local network who need voice and screen share without Discord.

The job in front of them is concrete. A group wants a chat that no third party holds, and one person in the group is willing to run the server, keep it updated and hand out the codes. That person is also a member of the chat, so the operator and a participant are often the same person.

The second audience is nobody yet. The code assumes an operator: registration needs a server, and there is no hosted tier to fall back on.

## Product Purpose

The server carries one conversation for a group of people who have decided to keep it on their own hardware.

Everything else hangs off that. Channels, voice, files, moderation and the admin panel are the parts of a chat server that a group needs and that an operator has to own.

Success for a participant: a friend sends a link, the recipient opens the browser, and the message is there. Success for the operator: the whole history is one SQLite file in one folder, and no company has the keys.

## Positioning

A neighbour of this product is Discord, and the difference that matters is where the messages sit. Discord is free and well made, and the messages sit on somebody else's disk, behind an account with an email address and a phone number. Here the messages are in a file the operator can open, copy and delete, and an account is a username and a password.

Two things are hard for a neighbour to copy. The first is the join code: membership is eight hex characters that the operator can rotate on a timer, rotate on a join count, make private, or replace with one code that opens every channel. There is no directory of public servers to browse and no invite from a stranger.

The second is the import. A group leaving Discord can bring its channel history, its reactions, its pins and its attachments across, so the switch is a step rather than a break.

A third difference is how small the whole thing is. The server is one process, one data directory and no build step for the client.

## Operating Context

The server runs where the operator can reach it: a home PC, a NAS, a small VPS. The operator is usually also the admin, and often a member of the chat. The people using it are in a browser, on the same local network or behind NAT.

Voice constrains the deployment. A browser will only hand over a microphone on a secure origin, so the operator either opens a port with the self-signed certificate that Haven generates, or puts a reverse proxy in front. Voice across NAT needs a TURN relay as well, and Haven will hand out credentials for one if `TURN_URL` is set.

The data directory is the operator's to back up: one folder holding the database, the configuration, the certificates and the uploads. The code folder holds none of it, so the checkout can be copied, forked or thrown away without taking a conversation along.

The client is used for hours, in a browser tab, next to other tabs. On a phone it is used in short bursts, and the native clients take over the same server with the same codes.

## Capabilities and Constraints

Confirmed by the code:

- One Node.js process serves the client, the JSON API and the socket endpoint.
- Channels nest one level as sub-channels, and a sub-channel can be private. Each channel has a topic, a type of standard, text only or voice only, a slow mode, and switches for uploads, screen sharing and the music player.
- A channel code is eight hex characters. It can be public or private, static or rotating on a timer or on a join count, and a server-wide code opens every top-level channel at once.
- Voice and screen sharing are peer to peer over WebRTC, with two STUN servers and an optional TURN relay whose credentials expire after 24 hours.
- A message is up to 2000 characters, with formatting, replies, reactions, pins, edits, polls, typing indicators, search, and twenty-two slash commands.
- Files are accepted up to the size the admin sets, 25 MB by default, with a 2 MB ceiling on an avatar and a 2 GB hard ceiling on a stored file.
- The browser receives web-push with VAPID keys generated on the first boot. The mobile client receives Firebase Cloud Messaging, sent directly with a service account, through a relay of your own, or through the shared relay the project runs.
- Direct messages are sealed in the browser with ECDH P-256, HKDF-SHA256 and AES-256-GCM. The private key is wrapped with a key derived from the password, and the server holds the public key and the wrapped blob.
- Moderation covers roles with per-channel permissions, a registration whitelist, kicks, timed mutes, bans, and automatic cleanup of old messages and files.
- A webhook posts into a channel with a 64-character token, and an admin can give it a name and an avatar.
- Discord history arrives three ways: with a token, as a DiscordChatExporter JSON file, or as a Discord data package.
- Plugins and themes are files dropped into two directories and listed by the server.
- The limits the server states: 20 authentication requests per 15 minutes per address, 10 uploads per minute, 30 link previews per minute, 30 GIF requests per minute, 30 webhook posts per minute, 64 KB per socket message, and 16 KB per JSON body.

Deliberately not true, and worth knowing before a change:

- There is no email address, no password reset by mail and no phone verification. A lost password means a recovery code or an administrator.
- Servers do not federate. The sidebar can list other Haven servers, and each one is a separate world with its own users.
- A running server does not update itself.
- The certificate is self-signed, so every new device has to accept the warning once.
- Registration is open by default. The whitelist exists and has to be turned on by an administrator.
- Voice works on a local network without any of the above. Across the internet it needs both a certificate and a TURN relay.

## Brand Commitments

**Name:** Haven. The mark is a hexagon, and it is the logo in the tab and in the sidebar.

**Tone:** plain and technical. The interface names the state of the thing it is doing, with the number in the message: "Upload rate limit, try again in a minute", "Image too large (max 25 MB)", the port in the startup banner. Nothing reassures the user; it tells them what happened.

**Keeping promises:** the changelog follows Keep a Changelog, the version in `package.json` matches the newest entry in the changelog, and the client asks the server for that version so it can tell the user when it is behind.

## Evidence on Hand

- `README.md` describes the installers, the configuration, the capability list and the troubleshooting.
- `GUIDE.md` is the user guide: setup, channels, join codes, remote access, tunnels, voice and push notifications.
- `CHANGELOG.md` records every release since the project began, newest first, with fixes described in terms of what the person would have seen.
- `docs/CROSS-PLATFORM-GAMEPLAN.md` and `docs/examples/haven-traefik-coturn/` show a deployment behind Traefik with a coturn relay.
- The code is the source for every default, every limit and every table above.

Absent, and not to be invented: no download counts, no number of running servers, no testimonials, no uptime figures and no benchmark against another chat product. The audience belongs to the upstream project, and this fork carries none of that history.

## Product Principles

1. **The server is the product.** The clients are thin. A person can stop the process, copy one folder and move it to another machine, and nothing is lost.
2. **Membership is a code.** Eight characters decide who is in. The operator can rotate them, expire them and keep them private.
3. **One file holds the history.** One SQLite database, one data directory, one backup.
4. **A refusal is named.** Limits, mute timers, slow mode and upload refusals reach the interface with the number in them, rather than failing silently or waiting for a log.
5. **Voice stays between the people.** The server carries the signalling and stays out of the audio path.

## Accessibility & Inclusion

- **The client is a web page.** A screen reader, a magnifier and a keyboard work without an install. The keyboard paths that exist are Enter to send, Shift+Enter for a new line, Tab to accept a suggestion from the mention, emoji or slash list, and Ctrl+F to search.
- **The interface states outcomes in words.** A refused upload, a mute and a rate limit each arrive as a sentence with the limit in it.
- **Nothing has been decided about a contrast standard.** The palette is computed from one accent colour and there is no ratio check, in the light themes or the dark ones.
- **The markup carries very little for a screen reader.** Seven elements in the chat client have an `aria-label` and the login page has none, and no part of the interface has been read aloud by a test.
- **Two-factor authentication asks for a device.** A TOTP code is entered by hand and the recovery codes are text, so a phone or a password manager is assumed.
- **The age rule is in the licence and in the form.** Registration asks the person to confirm that they are 18 or over, and the licence puts the obligation on whoever runs the server.
