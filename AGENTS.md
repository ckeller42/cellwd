# AGENTS.md

Rules for agents and contributors working in this repo. `CLAUDE.md` only imports this file.

## What cellwd is (planned)

A cellular watchdog for the GL.iNet GL-E5800 (Mudi 7) router: it detects a cellular data
session that is dead while the modem still reports "connected", and recovers it by cycling the
radio (`AT+CFUN=4` then `AT+CFUN=1`).

**Status: scaffold only.** This repo holds `README.md`, `LICENSE` (MIT), `.gitignore` and the
lint/CI setup. There is no code yet; everything below "planned" is the design, not what exists.

- **Today** the watchdog runs as a single script in the sibling repo `ckeller42/buspi-config`:
  `router-cellwd.sh`, installed on the router as `/etc/cellwd.sh` and run by root's cron every
  2 minutes. Its behaviour and the incident that motivated it are in buspi-config's
  `WATCHDOGS.md` (Layer 4) and `ROUTER-tailscale-subnet.md`.
- **Planned** (the cellwd package design spec and plan, kept local-only and not in any repo): a
  dependency-free OpenWrt ipk. One POSIX-shell script (`/usr/sbin/cellwd`, busybox ash target) run
  by a procd service and configured through uci (`/etc/config/cellwd`), tested with stubbed
  commands on `PATH`, and built with `ar`/`tar` without the OpenWrt SDK.

Until the full README lands (the README's "Task 15"), the design lives only in those local-only
notes. Ask the owner for them before adding code, and don't treat the plan as implemented.

## Checks

```sh
pipx install pre-commit
pre-commit install          # installs the pre-commit and pre-push hooks
pre-commit run --all-files  # what CI runs
```

`.pre-commit-config.yaml` pins every tool version (pre-commit-hooks, gitleaks,
markdownlint-cli2, shellcheck, actionlint, zizmor); CI (`.github/workflows/ci.yml`) runs the same
hooks plus a gitleaks scan of the whole working tree, and pins `pre-commit` itself. Markdown rules
live in `.markdownlint-cli2.jsonc` (line length 100). Shell code must be shellcheck-clean.
Workflows must pass actionlint and zizmor (offline audits only); fix findings, or annotate with
`# zizmor: ignore[<rule>]` and a reason.

## Never commit

- Secrets or router credentials (root/admin passwords, SSH keys, API tokens).
- Identifiers of a real modem or SIM: IMEI, ICCID, IMSI, phone number, or real public IPs.
  Use obviously fake values in tests and docs.
- Build output: `dist/` and `*.ipk` are gitignored; release packages come from CI.

## Workflow

- Branch off `main` and open a PR; don't push to `main` directly.
- CI must be green, and review threads must be resolved before merging.
- Commit subjects use the sibling repos' prefixes: `feat:`, `fix:`, `docs:`, `test:`, `chore:`.
