# c9watch

> Monitor and control all your Claude Code sessions from one place.

**c9watch** (short for **c**laude cod**e** watch, like k8s for Kubernetes) is a cross-platform desktop app that gives you a real-time dashboard of every Claude Code session running on your machine. No more switching between terminals to check which agent needs permission, which one is working, and which one is idle.

Supports **macOS**, **Linux**, and **Windows**.

## Demo

[![Watch Demo](https://img.youtube.com/vi/9PdN7joYmUk/maxresdefault.jpg)](https://youtu.be/9PdN7joYmUk)

## Works with everything. Tied to nothing.

Unlike other Claude Code management tools that require you to launch sessions from within their app, **c9watch doesn't care where you start your sessions**. It discovers them automatically by scanning running processes at the OS level.

Start Claude Code from any terminal or IDE you already use -- VS Code, Cursor, Zed, iTerm2, Windows Terminal, you name it -- and c9watch picks them all up. No plugins to install. No workflows to change. No vendor lock-in.

Just open c9watch and see everything.

## Lightweight and fast.

Built with **Tauri**, **Rust**, and **Svelte** -- not Electron. The app binary is small, memory usage is minimal, and the UI stays snappy. Rust handles process scanning and file parsing at native speed. Svelte compiles away the framework overhead. You're already running multiple Claude Code agents eating up resources -- your monitoring tool shouldn't add to the pile.

## Install

### macOS

**Quick install:**

```bash
curl -fsSL https://raw.githubusercontent.com/minchenlee/c9watch/main/install.sh | bash
```

Or grab the latest `.dmg` from the [Releases](https://github.com/minchenlee/c9watch/releases) page.

### Windows

See [Build from source -- Windows](#windows-1) below.

> **Note:** On Windows, c9watch needs to read process information from running Claude Code sessions. If sessions are not detected, try running c9watch as administrator.

### Linux

Grab the latest `.deb` or `.AppImage` from the [Releases](https://github.com/minchenlee/c9watch/releases) page.

### Build from source

Prerequisites: [Rust](https://rustup.rs/) (1.77+), [Node.js](https://nodejs.org/) (v18+), and the [Tauri prerequisites](https://v2.tauri.app/start/prerequisites/) for your platform.

#### Windows

1. Install [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) with the **"Desktop development with C++"** workload (includes MSVC and the Windows SDK).
2. Install [Rust](https://rustup.rs/) -- the installer will detect the Build Tools automatically.
3. Install [Node.js](https://nodejs.org/) (v18+).
4. [WebView2](https://developer.microsoft.com/en-us/microsoft-edge/webview2/) is required at runtime but comes pre-installed on Windows 10 (1803+) and Windows 11.

```bash
git clone https://github.com/Kraminius/c9watch-windows.git
cd c9watch-windows
npm install
npm run tauri build
```

#### macOS / Linux

```bash
git clone https://github.com/minchenlee/c9watch.git
cd c9watch
npm install
npm run tauri build
```

The built app will be in `src-tauri/target/release/bundle/`:

| Platform | Output |
|----------|--------|
| macOS | `macos/c9watch.app` |
| Linux | `deb/c9watch_*.deb` and `appimage/c9watch_*.AppImage` |

## Screenshots

### Status view -- see what needs your attention first

Sessions grouped by status. Permission requests surface to the top so you never leave an agent stuck waiting.

![Status view](docs/screenshots/status-view.png)

### Project view -- organize by codebase

Sessions grouped by project, each with its own status columns. See what's happening across all your repos.

![Project view](docs/screenshots/project-view.png)

### Compact view -- monitor at a glance

Minimal cards for when you just need a quick status check without the details.

![Compact view](docs/screenshots/compact-view.png)

### Conversation viewer -- inspect any session

Expand any card to see the full conversation history with formatted code, tool usage, and a navigation map.

![Conversation viewer](docs/screenshots/conversation-view.png)

## Features

- **Zero-integration setup** -- Works with any terminal or IDE, no plugins or extensions required
- **Auto-discovery** -- Detects all running Claude Code sessions by scanning processes at the OS level
- **Real-time status** -- See at a glance which sessions are Working, Need Permission, or Idle
- **Conversation viewer** -- Expand any session to view the full conversation with formatted markdown and code blocks
- **Session control** -- Stop sessions, open their parent terminal/IDE, or rename them for easier tracking
- **Multi-project view** -- Sessions grouped by project with git branch info
- **System tray integration** -- Quick access from the tray icon (macOS menu bar, Windows system tray)
- **Cross-platform** -- Works on macOS, Windows, and Linux

## How it works

1. A background thread polls every 2 seconds, scanning for running `claude` processes using `sysinfo`
2. Each process is matched to its session file in `~/.claude/projects/` via path encoding and timestamp correlation
3. The last N entries of each session's JSONL file are parsed to determine status:
   - **Working** -- Claude is generating a response or executing tools
   - **Needs Permission** -- A tool is pending that requires user approval
   - **Idle** -- Session is waiting for your next prompt
4. Status updates are pushed to the Svelte frontend via Tauri events
5. The UI reactively updates, sorting sessions by priority (permission requests surface first)

## Tech stack

| Layer | Technology |
|-------|-----------|
| Desktop framework | [Tauri 2](https://v2.tauri.app/) |
| Frontend | [SvelteKit](https://svelte.dev/) + [Svelte 5](https://svelte.dev/docs/svelte/overview) |
| Backend | Rust |
| Process discovery | [sysinfo](https://crates.io/crates/sysinfo) |
| Design system | Vercel Noir (true black, [Geist](https://vercel.com/font) fonts) |

## Development

```bash
npm install
npm run tauri dev
```

This starts both the Vite dev server (hot-reload for the frontend) and the Tauri Rust backend.

### Project structure

```
c9watch/
├── src/                    # SvelteKit frontend
│   ├── routes/             # Pages (+page.svelte, +layout.svelte)
│   ├── lib/
│   │   ├── components/     # Svelte components (SessionCard, MessageBubble, etc.)
│   │   ├── stores/         # Reactive state management
│   │   ├── demo/           # Demo mode with mock data
│   │   ├── api.ts          # Tauri command wrappers
│   │   └── types.ts        # TypeScript types
│   └── app.css             # Global styles
├── src-tauri/              # Rust backend (Tauri)
│   └── src/
│       ├── lib.rs          # Tauri commands and app setup
│       ├── polling.rs      # Background session detection loop
│       ├── actions.rs      # Stop/open session actions
│       └── session/
│           ├── detector.rs # Process-to-session matching
│           ├── status.rs   # Status determination logic
│           ├── parser.rs   # JSONL file parsing
│           └── permissions.rs # Auto-approval rule checking
```

## Demo mode

Press `Cmd+D` (macOS) or `Ctrl+D` (Windows/Linux) to toggle demo mode, which loads simulated sessions with animated status transitions. Useful for testing the UI without running real Claude Code sessions.

## Platform notes

### Windows

- c9watch automatically distinguishes between Claude Desktop (the Electron app) and Claude Code CLI sessions -- only CLI sessions are shown
- If sessions are not detected, run c9watch as administrator to ensure process access permissions
- Supported terminals: Windows Terminal, PowerShell, Command Prompt, and all cross-platform terminals (Alacritty, WezTerm, Ghostty, etc.)
- Supported IDEs: VS Code, Cursor, Windsurf, Zed, Sublime Text

### macOS

- c9watch appears in the menu bar for quick access
- Supported terminals: Terminal.app, iTerm2, Alacritty, kitty, Warp, Ghostty, and more
- Supported IDEs: VS Code, Cursor, Windsurf, Zed

### Linux

- Requires `xdotool` for window focus functionality (optional)
- Supported terminals: GNOME Terminal, Konsole, Alacritty, kitty, foot, WezTerm, Ghostty, and more
- Supported IDEs: VS Code, Cursor, Windsurf, Zed, Sublime Text

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on:
- Setting up the development environment
- Coding standards and commit message format
- Pull request process
- Platform-specific contributions (Windows, Linux)

## Contributors

Thanks to these wonderful people who have contributed to c9watch:

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tbody>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/minchenlee"><img src="https://github.com/minchenlee.png?s=100" width="100px;" alt="Min-Chen Lee"/><br /><sub><b>Min-Chen Lee</b></sub></a><br /><a href="#code-minchenlee" title="Code">💻</a> <a href="#doc-minchenlee" title="Documentation">📖</a> <a href="#design-minchenlee" title="Design">🎨</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/cynaptic"><img src="https://github.com/cynaptic.png?s=100" width="100px;" alt="Ray Lee"/><br /><sub><b>Ray Lee</b></sub></a><br /><a href="#code-cynaptic" title="Code">💻</a> <a href="#platform-cynaptic" title="Platform">📦</a></td>
    </tr>
  </tbody>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

See [CONTRIBUTORS.md](CONTRIBUTORS.md) for the full list and contribution details.

## License

MIT
