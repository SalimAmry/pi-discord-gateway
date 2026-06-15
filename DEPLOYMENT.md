# Deployment — Majlis Al Jinn WoW Ops Bot

This document records how `pi-discord-gateway` (piscord) is deployed on the
WoW ops agent host to drive the local `pi` coding agent from a single Discord
channel. It contains no secrets.

## Overview

The "Potato Man" Discord bot connects to a self-hosted piscord gateway, which
relays messages from one locked Discord channel to the `pi` coding agent and
posts replies back. The bot runs headless as a systemd user service.

## Gateway configuration

Config file: `~/.config/pi-discord-gateway/config.env` (mode `0600`, never
committed — the bot token lives only here).

| Setting          | Value                                      |
| ---------------- | ------------------------------------------ |
| `CHANNEL_POLICY` | `allowlist` (only registered channels are active) |
| `PI_MODEL`       | `claude-sonnet-4-6`                        |
| `PI_CWD`         | `/root`                                    |
| `PI_BIN`         | `pi`                                       |

`allowlist` policy means the gateway ignores every channel except the ones
explicitly registered, locking the bot to a single command channel.

## Registered channel

Exactly one channel is registered:

- **Majlis Al Jinn #potato-ops-commands** — registered with `--no-trigger`, so
  the agent responds to every message in this channel (no @mention required).

No other channel is registered. In particular, the audit channel is **not**
registered with the gateway.

## Pi login reuse

The gateway reuses the host's existing `pi` login (`~/.pi/agent/auth.json`); no
re-authentication is performed and no `pi` credentials are stored by piscord.

## Daemon (systemd user service)

Installed and managed via piscord's daemon commands:

```sh
sudo loginctl enable-linger "$USER"   # keep the user service running headless
piscord daemon install                # writes ~/.config/systemd/user/pi-discord-gateway.service
piscord daemon start                  # start the gateway
piscord daemon status                 # check service state
piscord status                        # full diagnostics (pi binary, auth, channels)
```

`enable-linger` lets the user service survive logout/reboot without an active
login session. `piscord status` should report the pi binary, pi auth present,
the gateway service active, and exactly one registered channel.
