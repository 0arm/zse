# zse
A Python CLI built for UNSW students to submit files and run autotests on local files against CSE machines over SSH. Heavily inspired by [cserun](https://cserun.bojin.co/).

`zse` syncs your local working directory to the CSE servers, runs the command you'd normally type into an SSH session (`1521 autotest`, `give`, etc.), streams the output back, and tidies up after itself.

<pre style="font-family: 'Cascadia Mono', monospace; font-size: 12px;">
<span style="color: lightgreen;">> zse 1511 autotest bad_pun</span>
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

**What you get over plain `cserun`:**
- Coloured, streamed output for autotests and `give` submissions
- `zse fetch` — pull lab files (or any other remote output) straight into your local folder
- `zse shell` — interactive `ssh -t` session inside the synced temp dir. Useful when you need to keep interacting with your program on the remote — e.g. debugging an interactive game or stepping through input prompts

## Installation

Install with [uv](https://docs.astral.sh/uv/) — drops `zse` into an isolated environment on your `PATH`:

```bash
uv tool install git+https://github.com/0arm/zse.git
```

Verify it landed:

```bash
zse --version
```

Upgrade later with `uv tool upgrade zse`. Runtime deps are pinned in [pyproject.toml](pyproject.toml) and [uv.lock](uv.lock).

## Setup

On first run, `zse` walks you through an interactive setup (zID, key path, optional passphrase) and writes `~/.zse/config.ini`. Run `zse config` to edit it later.

You need an SSH keypair authorised on CSE before that works — expand below if you haven't set one up.

<details>
<summary><strong>One-time SSH key setup</strong> (skip if you can already <code>ssh z5555555@login.cse.unsw.edu.au</code> without a password)</summary>

1. **Generate a keypair** (skip if `~/.ssh/id_ed25519` already exists):

   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
   ```

2. **Authorise it on CSE** — replace `z5555555` with your zID. You'll enter your zPass once:

   ```bash
   ssh-copy-id z5555555@login.cse.unsw.edu.au
   ```

3. **Sanity-check** — should connect without a password prompt:

   ```bash
   ssh z5555555@login.cse.unsw.edu.au
   ```

</details>

## Usage

### Running commands on CSE

The bare form is `zse <args...>` — anything that isn't a known subcommand is forwarded to the remote as-is, after syncing your current directory:

```bash
zse 1511 autotest bad_pun           # autotest from cwd
zse 1521 dryrun ass1 ass1.c         # dryrun before submitting
zse give cs1521 ass1 ass1.c ass1.h  # submit via `give`
zse 1511 explain bad_pun            # any other CSE command works too
```

The explicit form is `zse run <args...>` — identical behaviour, only useful if your command starts with a word that clashes with a subcommand name.

### Fetching files from CSE

`zse fetch <args...>` runs a command on the remote and downloads whatever it produces into the local directory. Detected automatically when the second positional is `fetch`:

```bash
zse 6991 fetch lab 00                    # → ./
zse 6991 fetch lab 00 --to ./labs/lab00  # → custom dir
zse fetch 6991 fetch lab 00              # explicit form
```

### Interactive shell

`zse shell` syncs your cwd and drops you into an `ssh -t` session inside the temp dir. Use it when your program needs ongoing interaction — debugging an interactive game, stepping through input prompts, poking around with a REPL:

```bash
zse shell          # bare shell in the synced temp dir
zse shell ./game   # run ./game, then drop to shell on exit
zse shell python3  # launch python, then drop to shell on exit
```

### Housekeeping

```bash
zse config    # edit ~/.zse/config.ini
zse purge     # delete the remote ~/.zse/ folder
zse purge -y  # skip the confirm prompt
```

Run `zse <command> --help` for the full flag list.

## Development

Clone the repo and sync the locked dev environment with uv:

```bash
git clone https://github.com/0arm/zse.git
cd zse
uv sync
uv run zse --version
```

`uv sync` installs the exact versions recorded in `uv.lock`. Run `uv lock --upgrade` to refresh pins after editing `pyproject.toml`.
