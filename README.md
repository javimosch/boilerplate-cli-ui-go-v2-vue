# boilerplate-cli-ui-go-v2-vue

Go CLI with embedded Vue 3 web UI. Single binary, no runtime dependencies.

Part of [SuperCLI](https://github.com/javimosch/supercli) - build CLI/UI plugins fast for 2026.

**v1**: [boilerplate-cli-ui-go](https://github.com/javimosch/boilerplate-cli-ui-go) | **React**: [boilerplate-cli-ui-go-v2-react](https://github.com/javimosch/boilerplate-cli-ui-go-v2-react) | **Node**: [boilerplate-cli-ui-node](https://github.com/javimosch/boilerplate-cli-ui-node) | **Python**: [boilerplate-cli-ui-python](https://github.com/javimosch/boilerplate-cli-ui-python)

## Architecture

```
boilerplate-cli-ui-go-v2/
├── main.go           # CLI entry point (start, stop, status, version)
├── server.go         # HTTP server with go:embed for UI files
├── daemon.go         # Daemon management (pid file, signals)
├── ui/               # Frontend (embedded at compile time)
│   ├── index.html    # Entry point (Vue 3 from CDN)
│   ├── css/
│   │   └── app.css
│   └── js/
│       ├── app.js
│       ├── components/
│       │   ├── AppLayout.js
│       │   ├── Sidebar.js
│       │   └── StatusCard.js
│       └── views/
│           ├── Dashboard.js
│           └── Settings.js
├── go.mod
├── build.sh
└── README.md
```

## Key Feature: `go:embed`

Frontend files are **separate** but **embedded into the binary** at compile time:

```go
//go:embed ui/*
var uiFiles embed.FS
```

**Benefits:**
- Single binary output (no runtime file dependencies)
- Separate HTML/CSS/JS files (proper syntax highlighting)
- No build step for frontend (CDN-based Vue/React)
- Hot-reload during development (serve from disk)

## Build

```bash
chmod +x build.sh
./build.sh
```

Output: Single binary `boilerplate-cli-ui-go-v2`

## Usage

```bash
# Start server (foreground)
./boilerplate-cli-ui-go-v2 start

# Start on custom port
./boilerplate-cli-ui-go-v2 start -port 3000

# Start as daemon
./boilerplate-cli-ui-go-v2 start -daemon

# Stop daemon
./boilerplate-cli-ui-go-v2 stop

# Check status
./boilerplate-cli-ui-go-v2 status
```

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /` | Web UI |
| `GET /api/status` | Server status (JSON) |
| `GET /api/health` | Health check (JSON) |

## Frontend Stack

- **Vue 3** (CDN) - Reactive UI
- **Tailwind CSS** (CDN) - Utility-first styling
- **Lucide Icons** (CDN) - Icon library
- **Hashbang routing** - `#/dashboard`, `#/settings`

No npm, no build step. Just open `ui/index.html` in your editor.

## Hashbang Routing

Routes use hashbang URLs:
- `http://localhost:8080/#/dashboard` - Dashboard view
- `http://localhost:8080/#/settings` - Settings view
- `http://localhost:8080/` - Defaults to dashboard

## Development

### Option 1: Edit embedded files

1. Edit files in `ui/`
2. Run `go run .` (files are re-embedded each run)
3. Refresh browser

### Option 2: Serve from disk (faster)

For development, you can serve files directly from disk:

```go
// In server.go, temporarily replace:
// uiSub, _ := fs.Sub(uiFiles, "ui")
// fileServer := http.FileServer(http.FS(uiSub))

// With:
fileServer := http.FileServer(http.Dir("ui"))
```

This allows hot-reload without recompiling.

## Adding New Views

1. Create `ui/js/views/MyView.js`:

```javascript
const MyView = {
    template: `
        <div>
            <h2>My View</h2>
            <!-- Your content -->
        </div>
    `,
    setup() {
        // Composition API logic
    }
};
```

2. Register in `ui/js/app.js`:

```javascript
app.component('my-view', MyView);
```

3. Add route in `ui/js/components/AppLayout.js`:

```javascript
// Add to navItems array
{ id: 'my-view', label: 'My View', icon: 'star' }

// Add to template
<my-view v-if="currentView === 'my-view'"></my-view>
```

## Adding New API Endpoints

1. Add handler in `server.go`:

```go
func handleMyEndpoint(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{"hello": "world"})
}
```

2. Register in `startServer()`:

```go
mux.HandleFunc("/api/my-endpoint", handleMyEndpoint)
```

## Comparison with v1

| Aspect | v1 (boilerplate-cli-ui-go) | v2 (this) |
|--------|---------------------------|-----------|
| HTML location | String literal in .go file | Separate `ui/` directory |
| Syntax highlighting | No | Yes |
| Component separation | No | Yes (js/components/) |
| Scalability | Poor | Good |
| Binary output | Single | Single |
| Frontend framework | Inline HTML | Vue 3 (CDN) |
