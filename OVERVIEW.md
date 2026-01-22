# Go Learning Project: Homelab Service Monitor

A CLI tool that monitors your self-hosted services (GitLab, Nextcloud, Jellyfin, etc.) and reports their status.

## Why This Project?

This hits several Go learning goals at once while building something immediately useful for your homelab:

- File I/O and config parsing
- HTTP client usage with timeouts and error handling
- Concurrency patterns — this is where Go shines
- CLI argument parsing
- Struct design and JSON marshaling

## Core Features

### 1. Configuration File

YAML or TOML config for defining services:

```yaml
services:
  - name: GitLab
    url: https://gitlab.example.com
    expected_status: 200
    timeout: 5s

  - name: Nextcloud
    url: https://cloud.example.com
    expected_status: 200
    timeout: 10s

  - name: Jellyfin
    url: https://media.example.com
    expected_status: 200
    timeout: 5s
```

### 2. Concurrent Health Checks

Use goroutines and channels to check all services simultaneously:

```go
type CheckResult struct {
    Service    string
    Status     string
    StatusCode int
    Latency    time.Duration
    Error      error
}

func checkServices(services []Service) []CheckResult {
    results := make(chan CheckResult, len(services))
    
    for _, svc := range services {
        go func(s Service) {
            results <- checkHealth(s)
        }(svc)
    }
    
    // Collect results...
}
```

### 3. Color-Coded Terminal Output

- 🟢 Green: Service is up
- 🔴 Red: Service is down
- 🟡 Yellow: Service is slow (response > threshold)

### 4. CLI Flags

```bash
homelab-monitor                  # Basic status check
homelab-monitor --json           # Machine-readable output
homelab-monitor --watch          # Continuous monitoring
homelab-monitor --config ./my-config.yaml
```

## Suggested Libraries

| Purpose | Option 1 (stdlib) | Option 2 (external) |
|---------|-------------------|---------------------|
| Config parsing | `encoding/json` | `spf13/viper` |
| CLI flags | `flag` | `spf13/cobra` |
| HTTP client | `net/http` | — |
| Colors | — | `fatih/color` |

Start with stdlib to learn the language, then refactor with external libraries once you understand what they abstract away.

## Project Structure

```
homelab-monitor/
├── main.go
├── config/
│   └── config.go      # Config parsing and validation
├── checker/
│   └── checker.go     # Health check logic
├── output/
│   └── output.go      # Terminal and JSON formatting
├── config.example.yaml
├── go.mod
└── README.md
```

## Stretch Goals

Once comfortable with the basics:

1. **Embedded Web Dashboard** — Go's `net/http` is surprisingly capable for simple UIs
2. **Push Notifications** — Integrate with ntfy.sh or Gotify when services go down
3. **Historical Data** — Store uptime history in SQLite using `database/sql`
4. **Cross-Compilation** — Build binaries for all your Linux boxes with `GOOS` and `GOARCH`

## Learning Resources

- [Go by Example](https://gobyexample.com/) — Concise, practical examples
- [Effective Go](https://go.dev/doc/effective_go) — Official style and idioms guide
- [Let's Go](https://lets-go.alexedwards.net/) — If you want to extend into web development

## Getting Started

```bash
# Initialize the project
mkdir homelab-monitor && cd homelab-monitor
go mod init homelab-monitor

# Create main.go and start coding
touch main.go
```
