# Data Guardian

A Telegram bot for monitoring IPs/domains with global ping probes and country-based alerting.

Data Guardian lets users:
- register IPs or domains,
- monitor reachability from multiple countries (via Check-Host),
- configure per-address country notifications,
- run full checks on demand,
- receive scheduled alert messages when connectivity degrades.

---

## Features

- **Telegram-first workflow** using inline keyboards and conversations.
- **IP/Domain registration** with eligibility limits based on user rank.
- **Country-specific notification rules** per monitored address.
- **Scheduled checks** for 10/20/30 minute notification windows.
- **Multi-language UX** (English and Farsi) for text and keyboard labels.
- **PostgreSQL persistence** for users, addresses, countries, notification rules, and ranks.

---

## Project Structure

```text
.
├── main.py                         # App entrypoint, handlers, and scheduler registration
├── create_database.py              # Schema and seed helpers
├── posgres_manager.py              # PostgreSQL connection pool + query/transaction abstraction
├── language.py                     # Localized bot text and keyboard labels
├── utilities.py                    # Error handling, language lookup, helpers
├── private.py                      # Runtime secrets and DB credentials (local-only)
├── api/
│   ├── checkHostApi.py             # Check-Host API client and data cleaning
│   └── virtualizorApi.py           # Virtualizor integration (work in progress)
├── ipGuardian/
│   ├── ip_guardian.py              # IP guardian menu + add IP conversation
│   ├── ip_guardianCore.py          # Core IP registration/check logic
│   └── myIPs.py                    # User IP management and per-address settings
├── notification/
│   ├── check_addreses_ping.py      # Queue and periodic notification jobs
│   └── check_address_pingsCore.py  # Notification data selection and formatting
├── user/
│   └── registerCore.py             # User registration flow
└── admin/
    └── adminTelegram.py            # Admin-side bot notifications
```

---

## Requirements

- **Python** 3.11+
- **PostgreSQL** 13+
- A valid **Telegram bot token**
- Network access to:
  - `api.telegram.org`
  - `check-host.net`

Install dependencies:

```bash
pip install -r req.txt
```

---

## Configuration

The project currently reads runtime configuration from `private.py`.

### Required values

- `telegram_bot_token`
- `telegram_bot_url`
- `admin_chat_ids`
- `database_detail`:
  - `db_name`
  - `db_host`
  - `db_user`
  - `db_password`
  - `db_port`

> **Important security note:**
> Do not commit real tokens/passwords. Use local secrets and rotate any credentials that were previously exposed.

---

## Database Setup

1. Create your PostgreSQL database.
2. Update `database_detail` in `private.py`.
3. Initialize schema:

```bash
python -c "from create_database import create; create()"
```

4. Seed initial reference data (countries + default rank):

```bash
python -c "from create_database import init; init()"
```

---

## Run the Bot

```bash
python main.py
```

At startup, `main.py` ensures schema creation (`create()`) and then starts polling updates from Telegram.

---

## How Scheduling Works

The bot registers three repeating jobs in the Telegram job queue:

- every **10 minutes** (`Check10`)
- every **20 minutes** (`Check20`)
- every **30 minutes** (`Check30`)

These jobs gather eligible addresses, queue checks, call Check-Host, and send user notifications when thresholds are crossed.

---

## Localization

All user-facing messages and keyboard labels are centralized in `language.py`.

- Text dictionary: `text_transaction`
- Keyboard dictionary: `keyboard_transaction`

Supported languages in the current implementation:
- `en` (English)
- `fa` (Farsi)

---

## Development Notes

- The project uses `python-telegram-bot` async handlers.
- Database access is abstracted by a small custom client in `posgres_manager.py`.
- Error handling decorators in `utilities.py` report exceptions to admins and show user-friendly messages.

Recommended improvements for production hardening:
- Move secrets from `private.py` to environment variables.
- Add automated tests (unit/integration).
- Add structured logging + log rotation.
- Containerize deployment and add CI checks.

---

## Troubleshooting

- **Bot does not start:** verify token, internet access, and dependency installation.
- **DB errors on startup:** ensure PostgreSQL is running and `database_detail` is correct.
- **No notifications received:** confirm addresses/countries are registered and periodic jobs are running.
- **Language fallback issues:** verify user language value exists in `UserDetail` and keys exist in `language.py`.

---

## License

No license file is currently included. Add a `LICENSE` file if you plan to distribute this project.
