# Documentation

The index of the documents in this folder.

Haven is a self-hosted chat server: one Node.js process serves the browser
client, the JSON API and the Socket.IO endpoint, and keeps everything in one
SQLite file under a data directory outside the code. The documents here
cover how that is put together, how to put it in front of the internet, and
what the interface looks like.

| Document | What it covers |
| --- | --- |
| [architecture.md](architecture.md) | The modules, the data flows, the decisions behind them and the boundaries. |
| [screenshots/README.md](screenshots/README.md) | Where the picture in the README comes from, and why this folder is empty. |
| [CROSS-PLATFORM-GAMEPLAN.md](CROSS-PLATFORM-GAMEPLAN.md) | The plan for the native desktop and mobile clients. |
| [examples/haven-traefik-coturn/README.md](examples/haven-traefik-coturn/README.md) | A deployment behind Traefik with a coturn relay, with the ports it needs. |
| [index.html](index.html) | The landing page, served as a static file. `website/index.html` is kept identical to it. |
| [support.html](support.html) | The support page. |
| [../README.md](../README.md) | The front door: the installers, the capability list and the configuration. |
| [../GUIDE.md](../GUIDE.md) | The user guide: setup, channels, join codes, remote access, tunnels, voice and push. |
| [../PRODUCT.md](../PRODUCT.md) | Who the server is for, and what it must not turn into. |
| [../AGENTS.md](../AGENTS.md) | The commands, the layout of the tree, and the traps. |
| [../DESIGN.md](../DESIGN.md) | The visual system: the token vocabulary and the twenty-six themes over it. |

## Reading order

New here: [../README.md](../README.md), then [../GUIDE.md](../GUIDE.md) for
the setup and the capability list.

Changing the code: [architecture.md](architecture.md), then
[../AGENTS.md](../AGENTS.md) for the traps. The traps cover the data
directory, the startup migrations and the client mixins, which are the
parts that cost a rebuild to learn.

Putting the server behind a domain name:
[examples/haven-traefik-coturn/README.md](examples/haven-traefik-coturn/README.md)
for the ports and the certificate, and `../GUIDE.md` for the tunnels.
