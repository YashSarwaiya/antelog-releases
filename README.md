# Antelog downloads

Antelog is a safety guard for AI coding agents (Claude Code and Cursor). It checks every
action the agent takes, blocks dangerous ones (deleting databases or cloud resources,
reading secrets, force-pushing, wiping folders), and keeps a tamper-evident log.
Everything runs on your own computer.

This repository only hosts the installers and packages.

## Install

macOS / Linux:

```sh
curl -fsSL https://github.com/YashSarwaiya/antelog-releases/releases/latest/download/install.sh | sh
```

Windows (PowerShell):

```powershell
irm https://github.com/YashSarwaiya/antelog-releases/releases/latest/download/install.ps1 | iex
```

Every package has a SHA-256 checksum, and the installers check it before installing.

## After installing

```sh
antelog doctor     # check your setup
antelog log        # what the agent did
antelog blocked    # what was stopped, and why
antelog verify     # check the log wasn't changed
```

## Credits

Antelog is built on [LogAct](https://github.com/facebookresearch/logact) by Meta
(MIT license; the license text ships in every package as `LICENSE-logact`).
