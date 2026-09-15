<div align="center">
  <h1>dotkeep</h1>
  <img src=".github/assets/logo.png" alt="dotkeep" width="80%">
  <h4>Dotfiles, kept simple</h4>
  <br>
  <p>A single plain-text manifest for syncing configuration with Git</p>
</div>

## Init

```sh
dotkeep init [DIR]
```

Creates `home/`, `root/`, empty `.dotkeep.conf`, and `git init`.

> Or skip init: mkdir a dir and write `.dotkeep.conf` yourself.
> Next: `cd` into that dir, edit `.dotkeep.conf`, then `dotkeep backup`.

```sh
dotkeep init ~/state
cd ~/state
$EDITOR .dotkeep.conf
```

## Config

```sh
dotkeep config
```

Shows the full path to `.dotkeep.conf` and its contents.

Plain file. One path per line. `#` comments.
Every entry must start with `home/` or `root/`.
No `~`, no absolute `/` forms, no `$VAR` expansion.
Prefer explicit files over whole dirs.
If both a dir and paths under it are listed, the dir wins and children are dropped.

Copy [dotkeep.conf.sample](dotkeep.conf.sample) or start from `dotkeep init`:

```sh
# shell / editor
home/.zshrc
home/.config/nvim
home/.config/tmux/tmux.conf

# git / ssh
home/.gitconfig
home/.ssh/config

# prefer files over whole dirs
home/.config/foo/config.toml
home/.config/foo/themes

# system
root/etc/hosts
root/etc/sysctl.d
```

```
home/.zshrc       is  $HOME/.zshrc
root/etc/hosts    is  /etc/hosts
```

## Where

`config` / `check` / `backup` / `restore` run inside *your* state dir
(the dir that holds `.dotkeep.conf`).
No fixed path or name. Any dir, any git remote, you choose.

```
repo/
├── .dotkeep.conf
├── home/
└── root/
```

## Backup

```sh
dotkeep backup
```

Reads `.dotkeep.conf`. Copies each listed path from the live system
into this repo (`home/` and `root/`). Asks before writing.
Next: `git add` / `commit` yourself. Dotkeep does not commit.

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
(`home/` and `root/`) onto the live system. Asks before writing.
Existing live targets go to `/tmp/dotkeep.XXXXXX/` first.

New machine:

```sh
git clone <url> <state-dir>
cd <state-dir>
dotkeep restore
```

Existing clone:

```sh
cd <state-dir>
git pull
dotkeep restore
```

## Commands

```
dotkeep <command>

init [DIR]   home/ root/ + empty .dotkeep.conf + git init
config       show .dotkeep.conf path + contents
check        validate + resolve paths
backup       copy live files into the repo
restore      copy repo files onto the live system
help
```

## Env

`NO_COLOR` disables color.

## Install

Needs `bash`, `rsync`, `git`, and `tree`.

```sh
mkdir -p ~/.local/bin
curl -fsSL -o ~/.local/bin/dotkeep \
  https://raw.githubusercontent.com/metaory/dotkeep/master/dotkeep
chmod +x ~/.local/bin/dotkeep
```

Or from a local clone:

```sh
install -Dm755 dotkeep ~/.local/bin/dotkeep
```

`~/.local/bin` must be on `PATH`.

## License

[MIT](LICENSE)
