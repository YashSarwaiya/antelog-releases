# Antelog

**A safety guard for AI coding agents.** Antelog checks every action Claude Code, Codex or
Cursor takes *before* it runs. It blocks the dangerous ones (deleting a database, wiping cloud
storage, reading secrets) and keeps a tamper-evident record of everything.

In April 2026 a coding agent deleted a company's production database, and its backups, with
one API call in nine seconds. It found a powerful token in a file and used it. Antelog is
built to stop that kind of action.

> Status: early (v0.9). It works on macOS, Linux and Windows with Claude Code, Codex and Cursor.
> Feedback is very welcome; please open an issue.

## Install

macOS / Linux:

```sh
curl -fsSL https://github.com/YashSarwaiya/antelog-releases/releases/latest/download/install.sh | sh
```

Windows (PowerShell):

```powershell
irm https://github.com/YashSarwaiya/antelog-releases/releases/latest/download/install.ps1 | iex
```

The installer checks the package's SHA-256 checksum. It installs to your user folder (no admin
rights needed) and starts the guard in the background. It also connects Claude Code, Codex and
Cursor if they're installed, then runs a setup check. Restart any open sessions afterwards.

**Codex users:** Codex ignores new hooks until you approve them. Open Codex and type `/hooks`
to approve Antelog's hooks; do it again after each Antelog update.

## What it blocks

| Danger | Examples |
|---|---|
| Deleting databases or tables | `DROP DATABASE`, `DROP TABLE`, `TRUNCATE`, `dropdb` |
| Deleting cloud resources | Railway, AWS, Google Cloud, Supabase, Neon CLIs, and `DELETE` calls to their APIs |
| Wiping folders | `rm -rf ~`, `rm -rf /`, `Remove-Item -Recurse C:\`, `rd /s /q C:\`, `format d:` |
| Rewriting shared history | `git push --force` to `main` or `master` |
| Reading secrets | `.env` files, SSH keys, cloud CLI login files (Railway, AWS, gcloud, Supabase, Neon, kube) |
| Running code from the internet | `curl … \| sh`, `bash <(curl …)` |
| Destructive commands aimed at production | `rails db:drop`, `kubectl delete`, `DELETE FROM` when they mention a production name |
| Turning the guard off | editing its settings or log, stopping its service, uninstalling it |

Antelog also **reads the scripts a command runs** (`python cleanup.py`, `npm run reset`,
`psql -f fix.sql`) and blocks them if they contain any of the above.

Everything else runs normally. A check takes about 5–10 ms.

When something is blocked, the agent sees why, for example:
`Blocked by safety rule: deleting Railway volumes, services, environments, or projects.`

## How it works

```
 Claude Code / Codex / Cursor wants to run something
        │   (a hook asks Antelog first)
        ▼
 Antelog, on your computer
   1. writes the action to its log
   2. checks it against the rules
   3. allows it or blocks it
        │
        ▼
 ✅ runs    or    ❌ blocked, with the reason
```

- **Runs locally.** Nothing is sent to us, and the check happens on your machine.
- **Fails safe.** If the guard crashes, is stopped or is missing, actions are blocked, not allowed.
- **Passwords are hidden** in the log (`[REDACTED]`).
- **Tamper-evident log.** Each entry is fingerprinted and chained to the previous one, so edits are detected.

Built on [LogAct](https://github.com/facebookresearch/logact) by Meta. It uses LogAct's
write-down-then-vote design, with real blocking rules added on top.

## Commands

```sh
antelog doctor          # check your setup and get fix-it tips
antelog log             # recent actions, allowed or blocked
antelog blocked         # only what was blocked, and why
antelog verify          # check the log hasn't been changed
antelog setup cursor    # connect Cursor (add --remove to disconnect)
```

`antelog doctor` also warns about risky setups: cloud logins the agent can use, a production
database password in `.env`, an admin database user, or the Claude Code sandbox being off.

## Settings

All in `~/.antelog/` (Windows: `%LOCALAPPDATA%\antelog\`). The agent can't edit these files.
Restart the guard after changing them; `antelog doctor` shows the command.

| File | What it does |
|---|---|
| `production.txt` | Your production names (hosts, project names), one per line. `production` and `prod` are always included. |
| `rules.txt` | Your own commands to block, one per line, e.g. `terraform destroy`, or `regex: drop\s+user`. |
| `anthropic-api-key` | Optional. Turns on an AI second look for likely false alarms (see below). |

**AI checker (optional).** Some rules can fire on harmless text, like
`grep "drop table" docs/`. With an Anthropic API key, Antelog asks Claude whether such a command
is clearly harmless. It only does this for plain text commands (search, print, docs). It never
does it for real database tools, scripts, or any of the hard rules above. If the AI is unsure,
slow or unavailable, the block stays. When it runs, the command (with passwords hidden) and your
last request are sent to Anthropic.

## What it does not protect against

Antelog is a strong seatbelt, not a guarantee. Please also use the protections your tools
already have.

- **Deletes written in code,** such as Python's `shutil.rmtree` or an ORM call, don't look like
  commands. Turn on the Claude Code sandbox (`/sandbox`) to limit what scripts can touch.
- **Code downloaded while a script runs** can't be read ahead of time.
- **Passwords stored in unusual places** can still be found and used. Give agents limited,
  read-only credentials where you can, and keep backups outside the system they protect.
- **A determined agent running as your user** could find a way around a guard that runs as the
  same user. Antelog blocks the obvious ways and fails safe if it's stopped.
- **Rules match text,** so they can block harmless commands that mention a dangerous phrase.
  The AI checker or a quick manual run handles those.
- **Claude Code older than 2.1.295** may let an action through if the hook program can't start
  at all. Update Claude Code.
- **Cursor** has a known bug where blocking file *reads* doesn't always work. Antelog's shell
  rules (`cat .env`) still apply.
- **Codex** lets an action through if its hook can't start at all (for example, Antelog was
  removed by hand on Windows), and it skips hooks you haven't approved with `/hooks`.
  `antelog doctor` checks for this.

## Uninstall

```sh
sh ~/.local/share/antelog/uninstall.sh            # macOS / Linux
```
```powershell
& "$env:LOCALAPPDATA\Programs\antelog\share\uninstall.ps1"   # Windows
```

Your history is kept unless you add `--purge` (or `-Purge` on Windows).

## License and credits

Antelog is open source under the [Apache 2.0 license](LICENSE). Source code: [YashSarwaiya/antelog](https://github.com/YashSarwaiya/antelog).

Antelog builds on [LogAct](https://github.com/facebookresearch/logact) by Meta Platforms, Inc.,
used under the MIT license. The license text ships in every package as `LICENSE-logact`.
