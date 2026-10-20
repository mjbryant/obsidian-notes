# Architecture — dotfiles

## Repository layout
```
dotfiles/
├── bash/           # .bash_profile, .bashrc, .bash/ (git-aware prompt)
├── claude/         # Claude Code config (symlinked to ~/.claude/)
│   ├── CLAUDE.md   # Global instructions → ~/.claude/CLAUDE.md
│   ├── .claude/
│   │   └── settings.json  # Permissions + hooks → ~/.claude/settings.json
│   └── obsidian/   # Obsidian integration scripts
│       ├── init-project.sh   → ~/.local/bin/obsidian-init-project
│       ├── sync-session.sh   → ~/.local/bin/obsidian-sync-session
│       ├── resolve-link.sh   → ~/.local/bin/obsidian-resolve
│       └── templates/        # Starter notes for new projects
├── git/            # .gitconfig, .git-completion.bash
├── tmux/           # .tmux.conf (prefix: Ctrl-F)
├── vim/            # .vimrc (Vundle plugins)
└── setup-symlinks.sh  # Creates all symlinks into $HOME
```

## Key files
- `setup-symlinks.sh` — symlink manifest + creation logic (skips existing, `--force` to overwrite)
- `claude/obsidian/init-project.sh` — bootstrap Obsidian notes + CLAUDE.md for a new project
- `claude/obsidian/sync-session.sh` — called by Stop hook; appends to session-log + git commits vault
- `claude/obsidian/resolve-link.sh` — resolves `[[wiki-links]]` to absolute file paths

## Patterns and conventions
- Everything in `claude/` that Claude Code uses is symlinked (not copied) so edits in dotfiles take effect immediately
- `claude/.gitignore` whitelists specific paths; new claude/ subdirs need explicit `!path/` exceptions added
- Scripts go to `~/.local/bin/` via setup-symlinks.sh; already on `$PATH` via `.bashrc`

## Obsidian integration
- Vault: `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Michael/`
- Git dir: `~/bed/obsidian` (worktree setup — vault `.git` is a pointer file)
- Per-project: `obsidian-init-project` creates `Projects/{name}/` in vault + `.claude/obsidian-context.md` symlink + project `CLAUDE.md`
- Stop hook: auto-appends session entry to `session-log.md` + commits vault on every session end
