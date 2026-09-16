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
  <a href="#restore">Restore</a> ·
  <a href="#commands">Commands</a>
</p>

<div align="center">

[help.webm](https://github.com/user-attachments/assets/c40d0e08-9b08-4edf-aa01-e8e608889162)

</div>

---

## Install

Needs `bash`, `git`, `rsync`, `tree`, `tput`, `sed`, and GNU coreutils

This repo is the **tool** only. Your backups live in a **separate** Git repo (see Init).

```sh
git clone https://github.com/metaory/dotkeep
cd dotkeep
```

Symlink `dotkeep` into any directory on your `PATH`:

```sh
ln -s "$(realpath dotkeep)" /path/on/PATH/dotkeep
```

## Init

```sh
dotkeep init [DIR]
```

Creates your **state repo**: `home/`, `root/`, empty `.dotkeep.conf`, a default `.gitignore`, and `git init`.
Not this project's clone. Pick any other dir, then add your own remote.

> [!TIP]
>
> Or skip init: mkdir a dir and write `.dotkeep.conf` yourself
> Next: `cd` into that dir, edit `.dotkeep.conf`, then `dotkeep backup`

```sh
dotkeep init ~/state
cd ~/state
$EDITOR .dotkeep.conf
```

## Config

```sh
dotkeep config
```

Shows the full path to `.dotkeep.conf` and its contents

> [!NOTE]
> Plain file. One path per line. `#` comments
> Every entry must start with `home/` or `root/`

> [!CAUTION]
>
> No `~`, no absolute `/` forms, no `$VAR` expansion
> Prefer explicit files over whole dirs
> If both a dir and paths under it are listed, the dir wins and children are dropped

> [!IMPORTANT]
>
> Symlinks are skipped (leaf and nested). Devices, fifos, and sockets are not synced
> Nested `.git` directories are stripped (content only, not repo metadata)
> Directory sync respects the state dir `.gitignore` (init seeds a Vite-style default)

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
(the one with `.dotkeep.conf`). Not the tool install. Any path, any remote.

```
state/
├── .dotkeep.conf
├── .gitignore
├── home/
└── root/
```

## Backup

```sh
dotkeep backup
```

Reads `.dotkeep.conf`. Copies each listed path from the live system
into this repo (`home/` and `root/`). Asks before writing

> [!NOTE]
>
> Ignored paths under `home/` / `root/` are pruned via `git clean -X`
> Files over 10 MiB are skipped and warned, not fatal; blobs already committed over 100 MiB still fail push until history is rewritten
> Next: `git add` / `commit` yourself. Dotkeep does not commit

```sh
dotkeep check
dotkeep backup
git add -A
git commit -m snapshot
git push
```

## Restore

```sh
dotkeep restore
```

Reads `.dotkeep.conf`. Copies each listed path from this repo
(`home/` and `root/`) onto the live system. Asks before writing

> Existing live targets go to `/tmp/dotkeep.XXXXXX/` first

### New machine:

```sh
git clone <url> <state-dir>
cd <state-dir>
dotkeep restore
```

### Existing clone:

```sh
cd <state-dir>
git pull
dotkeep restore
```

## Commands

```
dotkeep <command>

init [DIR]   create state repo (home/ root/ + .dotkeep.conf + git)
config       show .dotkeep.conf path + contents
check        validate + resolve paths
backup       copy live files into the state repo
restore      copy state repo files onto the live system
help
```

## Env

`NO_COLOR` disables color

## License

[MIT](LICENSE)
