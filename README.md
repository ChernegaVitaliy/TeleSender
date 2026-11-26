<div align="center">

<img src="assets/logo-squircle.svg" alt="TeleSender" width="160" height="160" />

# TeleSender

*A modular Telegram userbot with scheduling, plugins, and a web interface.*

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Telethon](https://img.shields.io/badge/Telethon-1.36%2B-2AABEE?logo=telegram&logoColor=white)](https://docs.telethon.dev/)
[![aiohttp](https://img.shields.io/badge/aiohttp-3.9%2B-2C5D72)](https://docs.aiohttp.org/)
[![Jinja2](https://img.shields.io/badge/Jinja2-templates-427116)](https://jinja.palletsprojects.com/)
[![python-dotenv](https://img.shields.io/badge/python--dotenv-1.0%2B-FFE873)](https://pypi.org/project/python-dotenv/)
[![SQLite](https://img.shields.io/badge/SQLite-storage-003B57?logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![Docker](https://img.shields.io/badge/Docker-supported-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-CSSM%20UL%20v2.0-blue)](LICENSE)

[Features](#features) · [Installation](#installation) · [Configuration](#configuration) · [Running the Project](#running-the-project) · [Usage](#usage) · [Plugins](#plugins) · [Contributing](#contributing)

---

</div>

TeleSender is a modular Telegram userbot written in Python with async libraries. It combines a task scheduler, a plugin system, a local media gallery and a password-protected web interface in a single application.

## Features

- **Modular Architecture**: Separate packages for Services, Models, Commands, Handlers, Core and Console.
- **Telegram API**: Asynchronous client powered by `Telethon`.
- **Task Scheduler**: Intervals (`30`, `5m`, `2h`, `1d`, `25m10s`), ranges (`60-120`) and weekday schedules (`14:10@1,3,5`).
- **Message Rotation**: Cycle through several messages; one-time scheduled tasks are supported too.
- **Media Support**: Photos, videos, documents, audio, stickers, GIFs, voice messages and video notes, with a local media gallery.
- **Plugin System**: Load, unload, reload, install from Git, update and create plugins at runtime; plugins can add commands, loops, watchers and web pages.
- **Web Interface**: Dashboard, task and plugin management, live logs and settings, with password login and optional Telegram 2FA.
- **Cloudflared Tunnel**: Exposes the web interface over HTTPS and can keep the same public URL alive across bot restarts when `TUNNEL_KEEP_ALIVE=true`.
- **Configuration Management**: Environment-based config via `python-dotenv`.
- **Docker Support**: A non-root image with `docker compose` and mounted runtime state.

## Installation

```bash
git clone https://github.com/ChernegaVitaliy/TeleSender.git
cd TeleSender
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Or install the package directly:

```bash
pip install -e .
```

After installation the bot can also be started with the `TeleSender` command.

## Configuration

Copy the example file and fill in your own values:

```bash
cp .env.example .env
```

```bash
API_ID=your_api_id
API_HASH=your_api_hash
PHONE_NUMBER=+380XXXXXXXXX
MY_USER_ID=your_telegram_id
ALLOWED_USERS=your_telegram_id
BOT_TOKEN=your_bot_token
WEB_AUTH_PASSWORD=your_web_password
```

`API_ID` and `API_HASH` come from https://my.telegram.org/apps. `BOT_TOKEN` is used by the inline helper bot. Telegram control is restricted to the IDs in `ALLOWED_USERS` and the logged-in account owner. The main settings are documented in [.env.example](.env.example); the full list is read in `src/config/config.py`.

## Running the Project

### Local

```bash
python main.py
```

Or, if the package is installed:

```bash
TeleSender
```

On the first run Telethon asks for the login code for the `PHONE_NUMBER` from `.env`. The session is stored under `data/`. An interactive console menu opens in the terminal; without a TTY the bot and web interface simply run in the background.

The web interface is available at `http://127.0.0.1:8000` when `WEB_ENABLED=true`.

### Docker

The first run needs a TTY for the Telegram login:

```bash
docker compose build
docker compose run --rm telesender
```

Later runs:

```bash
docker compose up -d
```

The web interface is available on port `8000`. The container runs as a non-root user with UID `1000`, so the mounted directories (`data`, `plugins` and the gallery directory) must be writable by that user.

To view logs:

```bash
docker compose logs -f
```

To stop:

```bash
docker compose down
```

## Project Structure

```
TeleSender/
+-- main.py                     # Entry point
+-- core/                       # Plugin base class, loader and registry
+-- src/
|   +-- bot.py                  # Bot assembly and startup
|   +-- commands/               # Telegram command classes
|   +-- config/                 # Environment configuration
|   +-- console/                # Interactive console menus
|   +-- core/                   # Bootstrap, client, service container
|   +-- handlers/               # Message, command and plugin handlers
|   +-- inline/                 # Inline bot, forms and buttons
|   +-- interfaces/             # Command interface
|   +-- models/                 # Task model
|   +-- services/               # Database, scheduler, tasks, plugins, tunnel, etc.
|   \-- web/                    # aiohttp server, routes, auth, templates, static files
+-- plugins/                    # Installed plugins (one folder per plugin)
+-- data/                       # Sessions, SQLite database, logs, web state
+-- assets/                     # Logos and bot avatar
+-- .env.example                # Documented configuration template
+-- requirements.txt
+-- pyproject.toml              # Packaging and console script
+-- Dockerfile
+-- docker-compose.yml
\-- LICENSE
```

## Usage

### Telegram Commands

- `.id` — your ID, chat ID and API ID
- `.id @username` — user ID (also works as a reply)
- `.apibid` — API ID used for tasks
- `.tasks` — task list
- `.stats` — statistics
- `.add` — add a task through an interactive dialog
- `.edit <ID>` — edit a task through an interactive dialog
- `.start_task <ID>` — start a task
- `.stop_task <ID>` — stop a task
- `.stop` — stop all tasks
- `.plugins` — show loaded plugins
- `.clone <url>` (or `.install <url>`) — install a plugin from Git
- `.update [name]` — update one or all plugins
- `.help` — help

### Console Menu

While the bot is running you can view statistics, add, list, start, stop, edit, activate, deactivate and delete tasks, stop everything, inspect task info, clear the media cache, export/import tasks, toggle the web interface and manage plugins.

### Web Interface

Dashboard with task statistics and bot info, task management, plugin installation and creation, live logs, export/import and 2FA settings. Login uses `WEB_AUTH_PASSWORD` and can require confirmation through the inline bot.

## Plugins

Each plugin is a folder inside `plugins/`:

```
plugins/myplugin/
+-- __init__.py
+-- plugin.py                   # Plugin class
+-- manifest.json               # Metadata
+-- config.json                 # Default settings (optional)
+-- static/                     # Web assets (optional)
\-- templates/                  # Web pages (optional)
```

```json
{
  "name": "myplugin",
  "author": "YourName",
  "version": "1.0.0",
  "description": "My first plugin",
  "category": "General",
  "id": "myplugin.yourname",
  "requirements": []
}
```

Plugins can be created from the console menu or the web interface, installed with `.clone <git-url>` and updated with `.update <name>`. Newly created plugins are loaded automatically; a restart is only needed if the automatic load fails.

## Contributing

Contributions are welcome and appreciated! Here's how you can contribute:

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

Please keep the existing code style and document any behaviour changes.

## License

This project is licensed under the CSSM Unlimited License v2.0 (CSSM-ULv2). See the [LICENSE](LICENSE) file for details.
