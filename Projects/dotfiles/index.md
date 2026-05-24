# dotfiles

## What this is
Personal dotfiles repository managing shell (bash), git, vim, tmux, and Claude Code configuration. Uses a symlink-based setup (`setup-symlinks.sh`) to link configs into `$HOME`. Managed at `/Users/mbryant/bed/dotfiles`, pushed to GitHub.

## Active context
Recently set up Claude + Obsidian integration: scripts in `claude/obsidian/` provide session-end auto-sync, project context loading via `@`-import, and `[[wiki-link]]` resolution.

## Key notes
- [[architecture]] — directory layout, what each config does, symlink manifest
- [[ongoing]] — current work in progress
- [[ideas]] — planned improvements
- [[session-log]] — history of Claude Code sessions

## Quick reference
- Setup: `./setup-symlinks.sh` (add `--force` to overwrite existing)
- New project: `obsidian-init-project [name] [dir]`
- Resolve link: `obsidian-resolve "note-name"`
