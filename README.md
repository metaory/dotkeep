# dotkeep

Sync the paths you list between machine and git.

Config owns inventory. Git owns history. The tool only syncs.

## Config

Plain file. One path per line. `#` comments.
`home/.zshrc` is `$HOME/.zshrc`. `root/etc/hosts` is `/etc/hosts`.
Same path in the repo.

Every entry must start with `home/` or `root/`.
No `~`, no absolute `/` forms, no `$VAR` expansion.
Prefer explicit files over whole dirs when the tree holds cache/state.

Copy [dotkeep.conf.sample](dotkeep.conf.sample) or start from `dotkeep init`:

```
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
repo/
├── .dotkeep.conf
├── home/
└── root/
```

Discovery, first hit wins. Repo = directory of the config file.

1. `./.dotkeep.conf`
2. `~/.dotkeep/.dotkeep.conf`
3. `~/.config/dotkeep/dotkeep.conf`

## Install

The executable is the `dotkeep` script in this repo. Needs `bash` and `rsync`.
Pick one.

Download the script:

```sh
mkdir -p ~/.local/bin
curl -fsSL -o ~/.local/bin/dotkeep \
  https://raw.githubusercontent.com/metaory/dot-snapshot/master/dotkeep
chmod +x ~/.local/bin/dotkeep
```

Or copy it from a local clone:

```sh
install -Dm755 dotkeep ~/.local/bin/dotkeep
```

`~/.local/bin` must be on `PATH`.

## Workflow

```sh
dotkeep init ~/state
cd ~/state
# edit .dotkeep.conf
dotkeep check
dotkeep backup
git init && git add -A && git commit -m snapshot
```

On a new machine, install `dotkeep` first, then:

```sh
git clone <state-repo> ~/state
cd ~/state
dotkeep restore
```

Existing targets are copied to `/tmp/dotkeep.XXXXXX/` before restore.

## Commands

```
dotkeep <command> [args]

init [DIR]      home/ root/ + empty .dotkeep.conf
check           validate + resolve paths
backup          machine → repo
restore         repo → machine (/tmp safety copy first)
help
```

## Env

`DOTKEEP_ROOT` skips discovery.
`NO_COLOR` disables color.

## License

[MIT](LICENSE)
