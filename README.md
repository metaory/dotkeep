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

[help.webm](https://github.com/user-attachments/assets/3cd57b4e-3d81-454b-a949-55938a64345c)

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
- empty `.dotkeep.conf`
- a default `.gitignore`
- `git init`

Remote is **optional**. Add one yourself when you want push/pull:

```sh
cd ~/state
git remote add origin <url>
```

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

## Why

**One file.** `.dotkeep.conf` is the source of truth

One path per line, `home/` or `root/`

Edit that file to add or drop a path

**State tree.** The backup dir looks like the live system:

```
home/.zshrc
home/.config/nvim
root/etc/hosts
```

Same names. Same nesting. Home and root in one list

**Copies.** `backup` and `restore` rsync both ways. Live paths stay regular files

**Ask first.** Both commands ask before writing. Restore parks live targets under `/tmp/dotkeep.XXXXXX/` first. A bad path is skipped. The rest is copied

**Git optional.** The store is that file tree. After backup it asks to add, commit, and push (each opt-in, default no). Syncthing or a disk copy can hold the same tree

### Compared with

**bare git, yadm.** Store is `$HOME`. Live files are the work tree. `git add -A` can commit SSH keys and caches. `/etc` does not fit

**Stow, rcm, homeshick, dotbot.** Store is the repo. Live path is a symlink. An editor that writes a tempfile and renames it over the path replaces the link. The repo keeps the old file

**chezmoi, dotdrop.** Store is the repo. Live path is a copy. Names are `dot_zshrc` and templates. Use them for per-host files or encrypted secrets

**dotkeep.** Store is `home/` and `root/` in a separate dir. Live path is a copy. Names match the disk

### Limits

**Platform.** Linux, bash 5, git, rsync, GNU coreutils. No Windows

**Root.** The tool does not call `sudo`. Restoring `root/` needs write access to that path

**Skip.** Symlinks (leaf and nested). Files over `10 MiB` (`DOTKEEP_MAX`, cap `100`)

**Out of scope.** Templates, encryption, per-host source

## Where

`config` / `check` / `backup` / `restore` run from **your state dir**

(the one with `.dotkeep.conf`). Not the tool install. Any path, any remote

## Backup

```sh
dotkeep backup
```

Run from your **state dir**. Flow:

1. **Git prep** _(optional; skipped if no `.git`)_
   - `fetch` origin if configured (fetch failure → continue local)
   - show status
   - if behind: ask `pull --rebase` (stash first if dirty). Default **no**
2. Show manifest + preview (`machine → repo`)
3. Ask before write. Default **no** → abort
4. Copy each listed path into `home/` / `root/`
5. Prune ignored paths under `home/` / `root/` via `git clean -X` _(if git)_
6. Rewrite `README.md` with a `tree` snapshot of the state repo
7. **Git ship** _(optional; only if `.git` and the tree is dirty)_
   - ask `git add -A` (default **no**)
   - ask commit message (default `snapshot YYYY-MM-DD HH:MM`)
   - ask `git push` if `origin` exists (default **no**; warn and skip if no remote)
8. Show short status + last 5 commits _(if git)_

> [!IMPORTANT]
>
> Pull, add, commit, and push are all opt-in. Default is no.
> Git itself is optional. No remote means no push prompt.
> A disk copy or Syncthing can hold the same tree.

> [!WARNING]
>
> Files over `10 MiB` skipped and warned
>
> `DOTKEEP_MAX` (MiB) default `10`, max `100`
> above `100` capped and warned
>
> GitHub rejects over `100 MiB` on push
> **committed blobs still fail** (history is scanned and warned)

After yes on ship:

```sh
git add -A
git commit -m "snapshot 2026-09-16 15:40"
git push   # only if origin exists and you say yes
```

## Restore

```sh
dotkeep restore
```

Run from your **state dir**. Flow:

1. Same **git prep** as backup _(optional; fetch + ask pull if behind)_
2. Show manifest + preview (`repo → machine`)
3. Ask before write. Default **no** → abort
4. Copy existing live targets to `/tmp/dotkeep.XXXXXX/` first
5. Prune ignored paths under state `home/` / `root/` _(if git)_
6. Copy each listed path onto the live system (`rsync --delete` under those paths)
7. Print the safety bak path when anything was saved

No commit or push on restore.

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
> Pull is opt-in when origin is ahead. Default is no.
> Dirty trees are stashed before pull, then popped.
> `--delete` means extras under a listed dir on the live side are removed.
> Prior live copies stay under `/tmp/dotkeep.XXXXXX/` until you clear them.

## Env

`NO_COLOR` disables color

`EDITOR` used by `config`, else `nvim` `vim` `vi`

`DOTKEEP_MAX` skip limit in `MiB`, `default 10`, `max 100`
above `100` capped and warned. GitHub rejects the push

## License

[MIT](LICENSE)
