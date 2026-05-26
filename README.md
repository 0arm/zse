# zse
A Python CLI built for UNSW students to submit files and run autotests on local files against CSE machines over SSH. Heavily inspired by [cserun](https://cserun.bojin.co/).

`zse` syncs your local working directory to the CSE servers, runs the command you'd normally type into an SSH session (`1521 autotest`, `give`, etc.), streams the output back, and tidies up after itself.

> **On Windows?** Run everything from a WSL shell.

## Installation

Install directly from GitHub. The recommended way is with [uv](https://docs.astral.sh/uv/), which drops `zse` into an isolated environment on your `PATH`:

```bash
uv tool install git+https://github.com/0arm/zse.git
```

Or with plain pip:

```bash
pip install git+https://github.com/0arm/zse.git
```

Verify the install:

```bash
zse --version
```

To upgrade later, re-run the install command (uv: `uv tool upgrade zse`).

Runtime dependencies (`paramiko`, `colorama`, `click`) are pinned to exact versions in [pyproject.toml](pyproject.toml), and a [uv.lock](uv.lock) is committed for fully reproducible dev environments.

## Initial setup

`zse` authenticates to CSE with an SSH keypair — password auth isn't supported.

### 1. Generate an SSH keypair (skip if you already have one)

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
```

Press Enter at the passphrase prompt to leave it empty, or pick one (zse will ask for it during setup). This creates:

- `~/.ssh/id_ed25519` — your **private** key (keep this secret)
- `~/.ssh/id_ed25519.pub` — your **public** key (safe to share)

### 2. Copy your public key to the CSE server

Use `ssh-copy-id` to install your public key on `login.cse.unsw.edu.au`. Replace `z5555555` with your zID:

```bash
ssh-copy-id z5555555@login.cse.unsw.edu.au
```

You'll be prompted for your zPass once. After this, you should be able to SSH in without a password:

```bash
ssh z5555555@login.cse.unsw.edu.au
```

### 3. Configure zse

Run any `zse` command for the first time and you'll be walked through an interactive setup — it asks for your zID, your private key path (default `~/.ssh/id_ed25519`), and an optional passphrase. The config is written to `~/.zse/config.ini`.

To edit it later:

```bash
zse config
```

This opens `~/.zse/config.ini` in VS Code. The file looks like:

```ini
[server]
address = login.cse.unsw.edu.au
port = 22
username = z5555555

[auth]
private_key_path = ~/.ssh/id_ed25519
passphrase =
```

You're done — try `zse 1511 autotest lab01` from a project directory.

## Usage

```
zse run 1511 autotest bad_pun -d ./tests/      # upload local files and run a command
zse fetch 6991 fetch lab08                     # run a command and download files into ./
zse fetch 6991 fetch lab08 --to ./labs/lab08   # download into a custom dir
zse shell python                               # interactive ssh -t session
zse 1511 autotest bad_pun                      # shorthand for `zse run ...`
zse 6991 fetch lab08                           # shorthand for `zse fetch ...`
zse config                                     # open config.ini in VS Code
zse purge                                      # wipe the remote ~/.zse/ folder
```

Common flags (work with `run`, `fetch`, and `shell`):

| Flag | Description |
| --- | --- |
| `-d, --dir PATH` | Local directory to upload (default: `./`) |
| `-e, --exclude PATTERNS` | Comma/whitespace-separated names to skip |
| `-c, --clear` | Clear the remote `~/.zse/` folder before syncing |
| `-f, --force` | Overwrite existing local files without prompting (used with `fetch`) |
| `-v, --verbose` | Print extra detail about uploads, paths, and exit codes |

Example output:

<pre style="font-family: 'Cascadia Mono', monospace; font-size: 12px;">
<span style="color: lightgreen;">> zse run 1511 autotest bad_pun -d ./tests/</span>
<span style="color: cyan;">[1/5]</span> Connecting to: <span style="color: yellow;">login.cse.unsw.edu.au:22</span>
<span style="color: cyan;">[2/5]</span> Authenticated as: <span style="color: lightgreen;">z5583960</span>
<span style="color: cyan;">[3/5]</span> Establishing SFTP connection
<span style="color: cyan;">[4/5]</span> Synced local files to remote
<span style="color: cyan;">[5/5]</span> Command sent: <span style="color: lightgreen;">1511 autotest bad_pun</span>
============== Output ==============
<span style="color: lightblue;">1511 c_check bad_pun.c</span>
<span style="color: lightblue;">dcc -Werror -o bad_pun bad_pun.c</span>
Test 0 (./bad_pun) - <span style="color: lightgreen;">passed</span>
====================================
<span style="color: lightgreen;">1 tests passed</span> <span style="color: red;">0 tests failed</span>
====================================
Exit Status: <span style="color: lightgreen;">0</span>
</pre>

## Troubleshooting

- **`Permission denied (publickey)`** — your public key isn't on the server yet. Re-run `ssh-copy-id z5555555@login.cse.unsw.edu.au` and confirm `ssh z5555555@login.cse.unsw.edu.au` works before retrying `zse`.
- **`Error: Cannot connect to CSE server`** — double-check `[server] address` and `username` in `config.ini`. Run `zse config` to open it.
- **Key has a passphrase and zse can't connect** — set `passphrase = ...` under `[auth]`, or remove the passphrase with `ssh-keygen -p -f ~/.ssh/id_ed25519`.
- **`Password auth was removed.`** — you have an old config from a previous zse version. Delete `~/.zse/config.ini` and re-run zse to start the new interactive setup.

## Development

Clone the repo and sync the locked dev environment with uv:

```bash
git clone https://github.com/0arm/zse.git
cd zse
uv sync
uv run zse --version
```

`uv sync` installs the exact versions recorded in `uv.lock`. Run `uv lock --upgrade` to refresh pins after editing `pyproject.toml`.

## Task list
- [x] add y/n confirmation before fetching from remote
- [x] enhance pipe feature i.e. actually make it useful
- [x] document SSH key setup in the README
- [ ] document how to build a standalone exe and add it to PATH
