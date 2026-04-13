---
name: quickshell
description: >
  Build, debug, and extend QuickShell configs — QML-based desktop shell/widget framework for Linux Wayland compositors.
  USE FOR: writing shell.qml configs, bars, panels, popups, widgets, process runners, IPC, Hyprland/Wayland integration,
  notifications, system tray, media controls (MPRIS), audio (PipeWire), power (UPower), multi-monitor layouts, singletons.
  DO NOT USE FOR: generic QML/Qt app development unrelated to desktop shells, non-Linux platforms.
---

# QuickShell

QuickShell is a QML-based framework for building custom desktop shells — bars, panels, popups, widgets — on Linux Wayland compositors. Configs are pure QML files; no build system required.

## When to Use

- User wants to create or modify a QuickShell `shell.qml` config
- User needs help wiring up Hyprland, Wayland, MPRIS, notifications, system tray, PipeWire, or UPower
- User wants to spawn processes, parse output, or do IPC from their shell
- User asks about any type from the `Quickshell.*` module namespace

## When Not to Use

- User is building a standalone Qt application (not a shell/bar/widget)
- User is on X11 only with no Wayland layer shell support
- User is asking about a different shell framework (AGS, Waybar, eww, etc.)

---

## Config Setup

| Item | Detail |
|------|--------|
| Entry point | `~/.config/quickshell/shell.qml` (or named subfolder: `~/.config/quickshell/<name>/shell.qml`) |
| Run default config | `quickshell` |
| Run named config | `quickshell -c <name>` |
| Run arbitrary file | `quickshell -p /path/to/shell.qml` |
| LSP support | Create empty `.qmlls.ini` next to `shell.qml`; QuickShell auto-fills it |

---

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Goal or existing QML snippet | Yes | What the user wants to build or fix |
| Compositor | No | Hyprland, generic Wayland, etc. — determines which modules to import |
| Target monitor | No | Single vs. multi-monitor affects whether to use `Variants` |

---

## Core Concepts

### Minimal bar (PanelWindow)

```qml
import Quickshell
import QtQuick

PanelWindow {
    anchors { top: true; left: true; right: true }
    implicitHeight: 30

    Text {
        anchors.centerIn: parent
        text: "hello world"
        color: "white"
    }
}
```

### ShellRoot — optional top-level wrapper

Use `ShellRoot` when you need to set global properties (e.g., `reloadCommand`) at the root level instead of having a bare window as the root.

```qml
import Quickshell

ShellRoot {
    PanelWindow {
        anchors { bottom: true; left: true; right: true }
        implicitHeight: 40
        // ...
    }
}
```

### Multi-monitor with Variants

Wrap windows in `Variants` to spawn one instance per screen. Instances auto-appear/disappear as monitors connect/disconnect.

```qml
import Quickshell

Variants {
    model: Quickshell.screens

    PanelWindow {
        required property var modelData
        screen: modelData

        anchors { top: true; left: true; right: true }
        implicitHeight: 30
    }
}
```

### Singletons — shared services

Add `pragma Singleton` to a `.qml` file (capitalized name). Access it by filename from anywhere in the config.

```qml
// Clock.qml
pragma Singleton
import Quickshell
import Quickshell.Widgets  // not needed here but common pattern

SystemClock {
    id: clock
    precision: SystemClock.Seconds
}
```

### Persistent properties across reloads

```qml
import Quickshell

PersistentProperties {
    id: persist
    reloadSource: "my-shell"

    property string lastWorkspace: ""
}
```

---

## Module Reference

### `Quickshell` (core)

| Type | Purpose |
|------|---------|
| `PanelWindow` | Panel attached to screen edges via anchors; reserves screen space |
| `FloatingWindow` | Standard OS window (decorations, free positioning) |
| `PopupWindow` | Temporary popup; use `PopupAnchor` to position relative to another item |
| `ShellRoot` | Optional root element for global config |
| `Variants` | Spawns one component instance per model entry (e.g., per screen) |
| `LazyLoader` | Async component loader; set `active: true` to trigger load |
| `BoundComponent` | Component loader that allows setting initial properties |
| `PersistentProperties` | Properties that survive config reload |
| `SystemClock` | System time; use `precision` to avoid unnecessary redraws |
| `ElapsedTimer` | Measures time between events |
| `ColorQuantizer` | Color quantization from an image |
| `DesktopEntries` | Index of `.desktop` application entries |
| `ObjectModel` / `ObjectRepeater` | Model/repeater for non-`Item` objects |
| `ScriptModel` | QML model reflecting a JS expression |

### `Quickshell.Io`

| Type | Purpose |
|------|---------|
| `Process` | Spawn a child process; set `command`, read stdout via a parser |
| `SplitParser` | Splits process stdout by a delimiter (e.g., `\n`) |
| `StdioCollector` | Buffers all output; `onStreamFinished` fires when process exits |
| `FileView` | Read small files; `JsonAdapter` for JSON |
| `Socket` / `SocketServer` | Unix domain socket IPC |
| `IpcHandler` | Handle IPC messages from a socket |

**Run a command and read output:**

```qml
import Quickshell
import Quickshell.Io

Process {
    id: dateProc
    command: ["date", "+%H:%M"]
    running: true

    stdout: StdioCollector {
        onStreamFinished: root.timeText = text.trim()
    }
}
```

**Periodic refresh with Timer:**

```qml
Timer {
    interval: 60000
    repeat: true
    running: true
    onTriggered: dateProc.running = true
}
```

### `Quickshell.Hyprland`

| Type | Purpose |
|------|---------|
| `Hyprland` | Main entry point; access workspaces, windows, monitors |
| `HyprlandWindow` | Hyprland-specific `QsWindow` properties |
| `HyprlandWorkspace` | Workspace state and management |
| `HyprlandMonitor` | Per-monitor properties |
| `HyprlandToplevel` | Top-level surface representation |
| `HyprlandFocusGrab` | Grab keyboard/mouse input focus |
| `GlobalShortcut` | Register a global keyboard shortcut |
| `HyprlandEvent` | Live IPC event from Hyprland |

### `Quickshell.Wayland`

| Type | Purpose |
|------|---------|
| `WlrLayershell` | Wlroots layer-shell window (panels, overlays) |
| `WlrLayer` | Layer enum (`Background`, `Bottom`, `Top`, `Overlay`) |
| `WlrKeyboardFocus` | Keyboard focus mode for layer-shell surfaces |
| `ToplevelManager` | List of all open windows via `wlr-foreign-toplevel` |
| `Toplevel` | A window from another application |
| `WlSessionLock` / `WlSessionLockSurface` | Implement a lockscreen |
| `ScreencopyView` | Capture and display another window or monitor |

### `Quickshell.Services.Mpris`

| Type | Purpose |
|------|---------|
| `Mpris` | Service entry point; `Mpris.players` lists active players |
| `MprisPlayer` | Media player: `trackTitle`, `trackArtist`, `playbackState`, `play()`, `pause()`, `next()`, `previous()` |
| `MprisPlaybackState` | `Playing`, `Paused`, `Stopped` |
| `MprisLoopState` | `None`, `Track`, `Playlist` |

### `Quickshell.Services.Notifications`

| Type | Purpose |
|------|---------|
| `NotificationServer` | Implement a notification daemon; receives `onNotification` signal |
| `Notification` | `summary`, `body`, `appName`, `urgency`, `actions`, `expire()`, `dismiss()` |
| `NotificationAction` | Action button attached to a notification |
| `NotificationUrgency` | `Low`, `Normal`, `Critical` |
| `NotificationCloseReason` | Why a notification was closed |

### `Quickshell.Services.SystemTray`

| Type | Purpose |
|------|---------|
| `SystemTray` | Entry point; `SystemTray.items` is the list of tray entries |
| `SystemTrayItem` | `icon`, `tooltip`, `menu`, `activate()` |
| `Status` | `Active`, `Passive`, `NeedsAttention` |
| `Category` | Application, Communications, SystemServices, Hardware |

### `Quickshell.Services.Pipewire`

| Type | Purpose |
|------|---------|
| `Pipewire` | Entry point; lists all nodes and links |
| `PwNode` | Audio node (device or app); `PwNodeAudio` for volume/mute |
| `PwLink` / `PwLinkGroup` | Connection between nodes |
| `PwNodeLinkTracker` | Track links to/from a specific node |
| `PwNodeType` | `Source`, `Sink`, `Filter`, etc. |

### `Quickshell.Services.UPower`

Battery/power management. Access the device list and monitor charge state via the `UPower` singleton.

### `Quickshell.Bluetooth`

Bluetooth device connectivity. Use the `Bluetooth` singleton to list adapters and devices.

### `Quickshell.DBusMenu`

System tray menu integration. Used internally by `SystemTrayItem.menu`.

### `Quickshell.Widgets`

| Type | Purpose |
|------|---------|
| `IconImage` | Renders theme icons by name |
| `ClippingRectangle` | Rectangle that clips children inside its border radius |
| `WrapperRectangle` / `WrapperItem` | Manages size/position of a single visual child |
| `WrapperMouseArea` | MouseArea that wraps a single visual child |

---

## Workflow

### Step 1: Check the config path

Ensure `~/.config/quickshell/` exists and contains (or will contain) a `shell.qml`.

### Step 2: Identify needed modules

Map the user's goal to modules:

| Goal | Module(s) |
|------|-----------|
| Bar / panel | `Quickshell` (`PanelWindow`) |
| Run shell commands | `Quickshell.Io` (`Process`, `StdioCollector`) |
| Workspaces (Hyprland) | `Quickshell.Hyprland` |
| Media controls | `Quickshell.Services.Mpris` |
| Notification popup | `Quickshell.Services.Notifications` |
| System tray | `Quickshell.Services.SystemTray` |
| Volume / audio | `Quickshell.Services.Pipewire` |
| Battery | `Quickshell.Services.UPower` |
| Lockscreen | `Quickshell.Wayland` (`WlSessionLock`) |
| Window list | `Quickshell.Wayland` (`ToplevelManager`) |

### Step 3: Write the QML

- Always add `import Quickshell` plus any service module imports needed
- Use `Variants { model: Quickshell.screens }` when the widget should appear on all monitors
- Use `pragma Singleton` in separate files for shared state (clock, network, etc.)
- Prefer `SystemClock` over spawning `date` for time — more battery-efficient
- Use `PersistentProperties` for state that must survive `quickshell --reload`

### Step 4: Reload and test

```bash
quickshell --reload          # hot-reload running instance
quickshell -p shell.qml      # run a specific file directly
```

### Step 5: Debug

- QML errors print to stderr when launching `quickshell` from a terminal
- Add `console.log(...)` in QML for runtime debugging
- Use `Process` with `StdioCollector` and log `stdout.text` to inspect command output

---

## Validation

- [ ] All required modules are imported (`import Quickshell`, `import Quickshell.Io`, etc.)
- [ ] Multi-monitor widgets use `Variants { model: Quickshell.screens }`
- [ ] Shared state is extracted to a `pragma Singleton` file
- [ ] `Process` commands are arrays, not strings: `command: ["bash", "-c", "..."]`
- [ ] `PanelWindow` anchors are set (`top`, `bottom`, `left`, `right` booleans)
- [ ] `quickshell --reload` or re-run confirms no QML parse errors

---

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| `command` is a plain string | Must be a string array: `["cmd", "arg"]` |
| Widget only appears on one monitor | Wrap in `Variants { model: Quickshell.screens }` and bind `screen: modelData` |
| State lost on `--reload` | Use `PersistentProperties` for values that must survive reload |
| Spawning `date` every second for a clock | Use `SystemClock { precision: SystemClock.Seconds }` instead |
| `StdioCollector.text` used before process finishes | Read text in `onStreamFinished`, not immediately after setting `running: true` |
| No LSP / autocomplete in editor | Create an empty `.qmlls.ini` next to `shell.qml`; QuickShell fills it automatically |
| Singleton not accessible | File must be capitalized (`Clock.qml`), have `pragma Singleton`, and be in the same config directory |
| Panel not reserving screen space | `PanelWindow` only reserves space with `anchors` set; add `exclusiveZone: implicitHeight` if auto-reservation fails |

---

## More Info

- Type reference: https://quickshell.org/docs/v0.2.1/types/
- Usage guide: https://quickshell.org/docs/v0.2.1/guide/
- Examples repo: https://github.com/outfoxxed/quickshell-examples
