# auto-knowledge

A Claude Code Routine that publishes a "Dev News Digest" to dev.to three times a day
(06:00, 12:00 and 18:00 Europe/Stockholm).

Each run starts a fresh Claude Code cloud session that:

1. Looks up the newest earlier digest on dev.to to find what is new since then.
2. Reads the developer news feeds listed in [ROUTINE.md](ROUTINE.md).
3. Writes a short summary, grouped by topic, with a source link on every item.
4. Publishes it to dev.to and sends a phone notification when the run finishes.

## Setup

The Routine runs in the cloud environment **Default**, which needs:

- An environment variable `DEVTO_API_KEY` holding the dev.to API key.
- Network access to `dev.to` and every feed host in [ROUTINE.md](ROUTINE.md).

## Changing the Routine

[ROUTINE.md](ROUTINE.md) is a copy of the prompt stored on the Routine. When you change
one, change the other (ask Claude to update the Routine's prompt with `update_trigger`).
