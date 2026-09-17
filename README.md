<div align="center">
  <img src=".github/assets/logo.jpg" alt="dotkeep" width="80%">
  <br>
  <h1>dotkeep</h1>
  <h4>Dotfiles, kept simple</h4>
  <p>A single plain-text manifest for syncing configuration with Git</p>
</div>

<p align="center">
  <a href="#install">Install</a> ·
  <a href="#init">Init</a> ·
  <a href="#config">Config</a> ·
  <a href="#backup">Backup</a> ·
  <a href="#restore">Restore</a>
</p>

<div align="center">

[help.webm](https://github.com/user-attachments/assets/cf8ef873-958d-4aac-9d31-5915f300b7d0)

</div>

---

## Install

Needs `bash`, `git`, `rsync`, `tree`, `tput`, `sed`, and GNU coreutils

> [!IMPORTANT]
>
> This repo is the **tool** only.
>
> Your backups live in a **separate** Git repo ( see [Init](#init) )

```sh
git clone https://github.com/metaory/dotkeep
cd dotkeep
```

Then put it on PATH:

```sh
# a dir you already have on PATH
ln -s "$(realpath dotkeep)" /your/path/dir/dotkeep

# or we handle it
sudo ln -s "$(realpath dotkeep)" /usr/local/bin/dotkeep
```

## Init

```sh
# on a new empty directory
dotkeep init [DIR]
```

Creates your **state repo**:

```
state/
├── .dotkeep.conf
├── .gitignore
├── home/
└── root/
```

- `mkdir` `home/`, `root/`
- `touch` `.dotkeep.conf`
- a default `.gitignore`
- `git init`

> [!TIP]
>
> Or skip init:
> mkdir a dir and write `.dotkeep.conf` yourself
>
> Next: `cd` into that dir, `dotkeep config`, then `dotkeep backup`

```sh
dotkeep init ~/state
cd ~/state
dotkeep config
```

## Config

```sh
dotkeep config
```

Shows the full path to `.dotkeep.conf` and its contents

Then asks to open it with `$EDITOR`, else `nvim`, `vim`, `vi`

> [!NOTE]
> Plain file. One path per line. `#` comments
>
> Every entry must start with `home/` or `root/`

> [!CAUTION]
>
> No `~`, no absolute `/` forms, no `$VAR` expansion
>
> Prefer explicit files over whole dirs
>
> Unreadable files are skipped
> The rest of the dir is copied
>
> If both a dir and paths under it are listed
> the dir wins and children are dropped

> [!IMPORTANT]
>
> `symlinks` are skipped (leaf and nested)
>
> `devices`, `fifos`, and `sockets` are not synced
>
> Nested `.git` and `node_modules` are stripped _(content only, not repo metadata)_
>
> Directory sync respects the state dir `.gitignore`

Copy [dotkeep.conf.sample](dotkeep.conf.sample) or start from `dotkeep init`:

```sh
# shell / editor
home/.zshrc
home/.config/nvim
home/.config/tmux/tmux.conf

# git / terminal
home/.gitconfig
home/.config/starship.toml
home/.config/alacritty/alacritty.toml

# prefer files over whole dirs
home/.config/foo/config.toml
home/.config/foo/themes

# optional system paths
root/etc/hosts
```

```
home/.zshrc       is  $HOME/.zshrc
root/etc/hosts    is  /etc/hosts
```

## Where

`config` / `check` / `backup` / `restore` run from **your state dir**

(the one with `.dotkeep.conf`). Not the tool install. Any path, any remote

## Backup

```sh
dotkeep backup
```

- Reads `.dotkeep.conf`
- Copies each listed path from the live system
- into this repo (`home/` and `root/`)
- Prompt confirmation and write
- If this dir is a git repo:
  - fetch origin
  - show status
  - Prompt confirmation and `pull --rebase`

> [!WARNING]
>
> Ignored paths under `home/` / `root/` are pruned via `git clean -X`
>
> Files over `10 MiB` skipped and warned
>
> `DOTKEEP_MAX` (MiB) default `10`, max `100`
> above `100` capped and warned
>
> GitHub rejects over `100 MiB` on push
> **committed blobs still fail**

> [!IMPORTANT]
>
> Asks before pull, commit, and push
>
> Default is no. Git is yours unless you say yes

Say yes after the copy and it runs:

```sh
git add -A
git commit -m "snapshot 2026-09-16 15:40"
git push
```

## Restore

```sh
dotkeep restore
```

- Reads `.dotkeep.conf`
- Copies each listed path from this repo
- Prompt confirmation and write
- If this dir is a git repo:
  - fetch origin
  - show status
  - Prompt confirmation and `pull --rebase`

> (`home/` and `root/`) onto the live system

> Existing live targets backups go to `/tmp/dotkeep.XXXXXX/` first

### New machine:

```sh
git clone <url> <state-dir>
cd <state-dir>
dotkeep restore
```

### Existing clone:

```sh
cd <state-dir>
dotkeep restore
```

> [!NOTE]
>
> Prompts to pull `--rebase` when origin is ahead
> Default is no
> Dirty trees are stashed first

## Commands

```
dotkeep <command>

init [DIR]   create state repo (home/ root/ + .dotkeep.conf + git)
config       show .dotkeep.conf, offer to edit
check        validate + resolve paths
backup       copy live files into the state repo
restore      copy state repo files onto the live system
help
```

## Env

`NO_COLOR` disables color

`EDITOR` used by `config`, else `nvim` `vim` `vi`

`DOTKEEP_MAX` skip limit in `MiB`, `default 10`, `max 100`
above `100` capped and warned. GitHub rejects the push

## License

[MIT](LICENSE)
