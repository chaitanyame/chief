# Chief

> **Build big projects with Claude.** Chief breaks your work into tasks and runs Claude Code in a loop until they're done.

[![Go](https://img.shields.io/badge/Go-1.24+-00ADD8?logo=go)](https://go.dev)
[![Built with Bubble Tea](https://img.shields.io/badge/TUI-Bubble%20Tea-ff69b4?logo=go)](https://github.com/charmbracelet/bubbletea)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

**[Documentation](https://minicodemonkey.github.io/chief/)** · **[Quick Start](https://minicodemonkey.github.io/chief/guide/quick-start)**

![Chief TUI](https://minicodemonkey.github.io/chief/images/tui-screenshot.png)

---

## Features

- **🔄 Infinite Context via Ralph Wiggum Loop** — Each Claude iteration starts with a fresh context window. Progress persists in `prd.json`, so Claude never hits context limits on large projects.
- **📋 PRD-Driven Development** — Define your project as a Product Requirements Document with user stories, priorities, and acceptance criteria. Chief works through them one at a time.
- **🖥️ Beautiful TUI** — Built with [Bubble Tea](https://github.com/charmbracelet/bubbletea) and [Lip Gloss](https://github.com/charmbracelet/lipgloss). Real-time dashboard, scrollable log viewer, keyboard shortcuts, and tabbed multi-PRD management.
- **⚡ Parallel PRD Execution** — Run multiple PRDs simultaneously. Each gets its own Claude instance with isolated state.
- **🔄 Auto-Retry on Crash** — If Claude crashes mid-iteration, Chief retries automatically (configurable: up to 3 retries with progressive delays).
- **🎵 Completion Notification** — Plays a sound when all stories are complete (WAV synthesis via `oto` — no external audio files needed).
- **🛡️ Branch Protection** — Warns before running on a protected branch (main/master) to prevent accidental commits.
- **📁 Clean Git History** — One commit per user story. Easy to review, easy to revert.
- **🔍 Live File Watching** — PRD changes are detected in real-time — edit your `prd.md` and the TUI refreshes automatically.
- **🧑‍💻 First-Time Setup** — Guided TUI flow to create your first PRD and configure `.gitignore` entries.

## How It Works

Chief implements the [Ralph Wiggum loop](https://ghuntley.com/ralph/) pattern: each iteration invokes Claude with a fresh context window, but progress is persisted between runs. This lets Claude work through large projects without hitting context limits.

```
┌──────────────────────────────────────────────────┐
│  1. Write your PRD (list of user stories)        │
│  2. `chief` → TUI launches, press 's' to start   │
│  3. Claude picks the highest-priority story      │
│  4. Implements it, runs quality checks, commits  │
│  5. Repeats until all stories pass                │
│  6. 🎵 Completion notification!                   │
└──────────────────────────────────────────────────┘
```

Each iteration:
1. Claude reads the PRD and `progress.md`
2. Picks the next incomplete story (highest priority)
3. Implements the story with full context awareness
4. Runs project quality checks (lint, test, typecheck)
5. Commits with a conventional commit message
6. Updates `prd.json` → sets the story to `passes: true`
7. Appends learnings to `progress.md` for future iterations

## Install

```bash
# macOS (Homebrew)
brew install minicodemonkey/chief/chief

# Linux, macOS, or WSL (install script)
curl -fsSL https://raw.githubusercontent.com/MiniCodeMonkey/chief/refs/heads/main/install.sh | sh

# Specific version
curl -fsSL https://raw.githubusercontent.com/MiniCodeMonkey/chief/refs/heads/main/install.sh | sh -s -- --version v0.1.0
```

The install script detects your OS and architecture, downloads the correct binary from GitHub releases, verifies the checksum, and installs it to `/usr/local/bin` (or `~/.local/bin` if no sudo is available).

## Quick Start

```bash
# Create a new project
chief new

# Launch the TUI and press 's' to start
chief
```

```bash
# Create a named PRD
chief new auth "JWT authentication for REST API"

# Launch with a specific PRD
chief auth

# Check progress without TUI
chief status
chief list
```

## CLI Reference

### Commands

| Command | Description |
|---------|-------------|
| `chief` | Launch TUI (uses `.chief/prds/main/` or lets you pick) |
| `chief <name>` | Launch TUI with a named PRD |
| `chief ./path/to/prd.json` | Launch TUI with a specific PRD file |
| `chief new` | Create a new PRD interactively |
| `chief new <name> [context]` | Create a named PRD with optional context hint |
| `chief edit [name]` | Edit an existing PRD interactively |
| `chief status [name]` | Show progress for a PRD (default: main) |
| `chief list` | List all PRDs with progress |
| `chief help` | Show help |
| `chief --version` | Show version |
| `chief wiggum` | 🐟 Easter egg |

### TUI Options

| Option | Description |
|--------|-------------|
| `-n N`, `--max-iterations N` | Max iterations (default: auto-calculated) |
| `--no-sound` | Disable completion notification sound |
| `--no-retry` | Disable auto-retry on Claude crashes |
| `--verbose` | Show raw Claude output in the log viewer |
| `--merge` | Auto-merge progress on prd.md → prd.json conversion conflicts |
| `--force` | Auto-overwrite on conversion conflicts |

### TUI Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `s` | Start / resume |
| `p` | Pause |
| `x` | Stop |
| `t` | Toggle dashboard / log view |
| `n` | Create new PRD |
| `l` | List PRDs |
| `1`–`9` | Switch to PRD by tab number |
| `↑` / `k` | Scroll up |
| `↓` / `j` | Scroll down |
| `ctrl+d` / `ctrl+u` | Page down / page up |
| `g` / `G` | Top / bottom |
| `+` / `-` | Increase / decrease max iterations |
| `?` | Help overlay |
| `q` / `ctrl+c` | Quit |

### Edit Options

| Option | Description |
|--------|-------------|
| `--merge` | Auto-merge existing progress into edited PRD |
| `--force` | Auto-overwrite without confirmation |

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated (`claude` must be in your `$PATH`)
- macOS or Linux (Windows via WSL is supported by the install script)

## Project Structure

```
chief/
├── cmd/chief/main.go          # Entry point, CLI flag parsing
├── embed/
│   ├── prompt.txt             # Agent instructions for Claude
│   └── embed.go               # Embedded prompt via Go embed
├── internal/
│   ├── tui/                   # Bubble Tea terminal UI
│   │   ├── app.go             # Main TUI model
│   │   ├── dashboard.go       # PRD progress dashboard
│   │   ├── log.go             # Log viewer
│   │   ├── tabbar.go          # Multi-PRD tab bar
│   │   ├── picker.go          # PRD picker / switcher
│   │   ├── help.go            # Help overlay
│   │   ├── branch_warning.go  # Protected branch dialog
│   │   └── styles.go          # Lip Gloss styles
│   ├── loop/                  # Core agent loop
│   │   ├── loop.go            # Single-PRD loop with retry
│   │   ├── manager.go         # Multi-PRD parallel execution
│   │   └── parser.go          # Stream-json parser for Claude output
│   ├── prd/                   # PRD types and operations
│   │   ├── types.go           # UserStory, PRD structs
│   │   ├── loader.go          # Load/save prd.json
│   │   ├── generator.go       # prd.md → prd.json conversion
│   │   └── watcher.go         # File watcher for live updates
│   ├── cmd/                   # CLI commands
│   │   ├── new.go             # `chief new`
│   │   ├── edit.go            # `chief edit`
│   │   └── status.go          # `chief status`
│   ├── git/                   # Git helpers
│   │   ├── git.go             # Branch detection, gitignore
│   │   └── gitignore.go       # .chief gitignore management
│   └── notify/                # Sound notification
│       ├── sound.go           # Platform-aware sound playback
│       ├── wav.go             # WAV synthesis
│       └── gen_wav.go         # Completion chime generator
├── install.sh                 # Cross-platform install script
├── go.mod / go.sum            # Go module dependencies
└── README.md
```

## Development

### Prerequisites

- Go 1.24+
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) (for testing the full loop)

### Build

```bash
git clone https://github.com/MiniCodeMonkey/chief.git
cd chief
go build -o chief ./cmd/chief/
```

### Run from source

```bash
go run ./cmd/chief/ --verbose
```

### Run tests

```bash
go test ./...
```

### Cross-compile

```bash
GOOS=linux GOARCH=amd64 go build -o chief-linux-amd64 ./cmd/chief/
GOOS=darwin GOARCH=arm64 go build -o chief-darwin-arm64 ./cmd/chief/
```

## Why Chief?

Large projects inevitably overflow a single Claude context window. Chief solves this by:

- **Fresh context, every iteration** — No context bloat. No "I forgot about the file I wrote 50 turns ago."
- **Persistent state** — `prd.json` tracks story status across iterations. Progress never lost.
- **Breadcrumb trail** — `progress.md` accumulates learnings so each new iteration benefits from past explorations.
- **One commit per story** — Clean, reviewable git history. Each story is a logical unit of work.

It's the difference between asking Claude to "build an entire app" and asking it to "implement one well-defined feature, then commit and move on."

## Acknowledgments

- [snarktank/ralph](https://github.com/snarktank/ralph) — The original Ralph implementation that inspired this project
- [Geoffrey Huntley](https://ghuntley.com/ralph/) — For coining the "Ralph Wiggum loop" pattern
- [Bubble Tea](https://github.com/charmbracelet/bubbletea) — TUI framework
- [Lip Gloss](https://github.com/charmbracelet/lipgloss) — Terminal styling

## License

MIT
