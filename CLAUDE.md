# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## What this project is

A minimal **Telegram bot** that collects daily standup status updates from team
members. It is built on [`python-telegram-bot`](https://python-telegram-bot.org/)
v20+ and runs as a long-polling process. The current version is an early,
intentionally simple starting point — updates are acknowledged but not yet
persisted or forwarded to a group chat.

## Repository layout — read this first

The repository is unusual: the actual source code is **not unpacked in the
working tree**. It lives inside a zip archive.

```
.
├── README.md                    # One-line stub (`# telegram-standup-bot`)
├── telegram-standup-bot.zip     # Contains the real project
└── CLAUDE.md                    # This file
```

`telegram-standup-bot.zip` contains:

```
standup_bot.py                   # The entire bot (~30 lines)
README.md                        # The real, detailed project README
```

Note the two README files differ: the repo-root `README.md` is a placeholder,
while the README *inside the zip* is the full project documentation. When
answering questions about "the README", clarify which one you mean.

### Implication for any change

Before editing code, extract the archive:

```bash
unzip -o telegram-standup-bot.zip -d src/
```

When you finish, decide with the user whether the deliverable is:

1. **Unpacked source** committed to the tree (recommended going forward — makes
   the code diff-able, reviewable, and CI-friendly), or
2. **A refreshed zip** re-archiving the edited files to preserve the current
   structure.

Do not silently leave edits only inside a re-zipped blob if the user expects
reviewable diffs — surface the choice.

## The code (`standup_bot.py`)

Single-file, async application using the PTB v20 API:

- **`start(update, context)`** — handler for `/start`; greets the user and
  points them to `/update`.
- **`update_status(update, context)`** — handler for `/update <text>`; reads the
  status from `context.args`, and either acknowledges it or, if empty, prompts
  for input. **The actual save/forward logic is a TODO** — the code comment
  (`# Здесь должен быть сбор и отправка данных`) marks where persistence and
  group-forwarding belong.
- **`__main__` block** — reads `TELEGRAM_BOT_TOKEN` from the environment, builds
  the `Application`, registers the two `CommandHandler`s, and calls
  `app.run_polling()`.

Some inline comments are in Russian; keep or translate them consistently with
surrounding code when editing, and match the file's existing comment style.

## Running the bot

Requirements: **Python 3.10+** and **`python-telegram-bot` v20+** (the async
API — v13 and earlier are incompatible).

```bash
pip install --upgrade python-telegram-bot

# Get a token from @BotFather on Telegram, then:
export TELEGRAM_BOT_TOKEN=your_token_here     # Linux/macOS
# set TELEGRAM_BOT_TOKEN=your_token_here       # Windows CMD

python standup_bot.py                          # after unzipping
```

The process runs `app.run_polling()` and stays in the foreground until
interrupted. There is no test suite, linter config, or CI in the repo yet.

## Conventions & guardrails

- **Secrets**: the bot token comes *only* from the `TELEGRAM_BOT_TOKEN`
  environment variable. Never hard-code a token, and never commit one. If you
  add config, keep secrets in the environment (or a git-ignored `.env`).
- **Async everywhere**: PTB v20 handlers are `async def` and awaited. New
  handlers must follow the same pattern and be registered in the `__main__`
  block with `app.add_handler(...)`.
- **Keep it minimal**: this is a small starter project. Prefer small, focused
  additions over large frameworks unless the user asks to scale it up.

## Roadmap (from the project README — good "next feature" candidates)

- Persist updates to a file or database.
- Automatically compile a daily summary.
- Send the summary to a group chat.
- Schedule daily standup prompts (PTB's `JobQueue` fits here).

When implementing any of these, wire the logic into `update_status` (for
collection) and add a scheduled job or new command for summaries.

## Git workflow

- Default branch: `main`.
- Do work on a feature branch; push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
