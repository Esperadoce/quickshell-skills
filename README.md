# quickshell-skills

A [Claude Code](https://claude.ai/code) skill for building desktop shells with [QuickShell](https://quickshell.org) — a QML-based shell framework for Linux Wayland compositors.

## What's included

The skill covers:

- Config setup and entry points
- Core patterns: `PanelWindow`, `ShellRoot`, `Singleton`, `Scope`, `Variants`
- `pragma ComponentBehavior: Bound` — when and why
- Typed functions and properties (QML 6 style)
- `Connections`, null safety (`?.`, `??`), inline `component` definitions
- `IpcHandler` — expose CLI control with `qs ipc call`
- `PersistentProperties` — survive `--reload`
- `Process`, `FileView`, `LazyLoader`, `Quickshell.execDetached`
- Full module reference: `Quickshell.Io`, `Hyprland`, `Wayland`, `Mpris`, `Notifications`, `SystemTray`, `Pipewire`, `UPower`, `Widgets`
- Validation checklist and common pitfalls

## Installation

> Requires [Claude Code](https://claude.ai/code) with plugin support.

```bash
# Install via Claude Code CLI (once local plugin support is available)
# For now, copy the files manually:
mkdir -p ~/.claude/plugins/cache/local/quickshell/0.1.0/skills/quickshell
cp plugin.json ~/.claude/plugins/cache/local/quickshell/0.1.0/
cp skills/quickshell/SKILL.md ~/.claude/plugins/cache/local/quickshell/0.1.0/skills/quickshell/
```

Then register in `~/.claude/plugins/installed_plugins.json` and enable in `~/.claude/settings.json`.

## License

[MIT](LICENSE)
