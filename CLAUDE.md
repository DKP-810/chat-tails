# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Build the binary
make build              # or: go build -o chat-server ./cmd/ts-chat

# Run the server
make run                # builds and runs
./chat-server           # run directly

# Run tests
make test               # runs: go test -v ./internal/chat/

# Run a single test
go test -v -run TestName ./internal/chat/

# Cross-compile
make build-all          # builds for linux, macos, windows, arm
```

## Architecture

This is a terminal-based chat server written in Go that supports both regular TCP mode and Tailscale networking.

### Package Structure

- `cmd/chat-tails/main.go` - Entry point, CLI flag parsing with spf13/pflag
- `internal/server/` - Server lifecycle (start/stop), connection handling, Tailscale integration via tsnet
- `internal/chat/` - Core chat logic:
  - `room.go` - Room manages clients via channels (join/leave/broadcast pattern)
  - `client.go` - Client handles per-connection I/O, commands, rate limiting
- `internal/ui/` - Terminal styling using charmbracelet/lipgloss

### Key Patterns

**Room event loop** (`room.go:run`): Uses channel-based concurrency with `join`, `leave`, and `broadcast` channels processed in a single goroutine to avoid race conditions on the client map.

**Client handling** (`client.go:Handle`): Uses goroutine-based reader with context cancellation for clean shutdown. Rate limiting uses sliding window (5 messages per 5 seconds).

**Connection modes**: Regular TCP (`net.Listen`) or Tailscale (`tsnet.Server.Listen`) based on `--tailscale` flag. Tailscale auth via `TS_AUTHKEY` env var.

### Chat Commands

`/who`, `/me <action>`, `/help`, `/quit` - implemented in `client.go:handleCommand`

## Recent Updates

**2026-01-16**: Updated Tailscale to v1.92.5
- Updated `tailscale.com` dependency from v1.82.5 to v1.92.5 in go.mod
- Updated Go toolchain to 1.25.5 (auto-updated by go mod tidy)
- Updated Dockerfile base image to `golang:1.25.6-alpine`
- Docker image published to `ghcr.io/dkp-810/chat-tails:v1.92.5`
