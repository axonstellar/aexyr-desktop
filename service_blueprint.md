# AEXYR Desktop — Service Blueprint

**Project:** AEXYR Desktop  
**Repo:** [axonstellar/aexyr-desktop](https://github.com/axonstellar/aexyr-desktop)  
**Version:** 0.1.0 (planning phase)  
**Status:** Pre-development — architecture and specification  
**Last Updated:** 2026-09-11  

---

## 1. Overview

AEXYR Desktop is a native cross-platform desktop application for managing one or more AEXYR instances. It provides a rich visual interface for infrastructure monitoring, service management, and conversational AI interaction — all from a single window on macOS, Windows, or Linux.

The desktop app is the **mission control** interface — purpose-built for daily operators and teams who manage AEXYR-powered infrastructure.

## 2. Product Context

AEXYR Desktop is part of the **Product Trilogy**:

| Interface | User | Use Case |
|---|---|---|
| **Web UI** | Anyone with a browser | Quick access, mobile, sharing dashboards |
| **AEXYR CLI** | Power users, DevOps, CI/CD | Automation, scripting, terminal-native workflows |
| **AEXYR Desktop** | Daily operators, teams | Mission control, multi-instance management, persistent monitoring |

All three share the same APIs, same agent, same infrastructure — three interfaces for different contexts. All connect through Cloudflare Tunnel with zero open ports.

## 3. Why Desktop Over Browser

| Capability | Browser | Desktop App |
|---|---|---|
| **Multi-instance management** | One tab per server, no unified view | Single window, all instances in sidebar |
| **System tray / persistent presence** | Tab must stay open | Always running, notification badges |
| **Native notifications** | Browser notifications (often blocked) | OS-native alerts — critical service down, build complete |
| **Local file integration** | Upload/download dialogs | Drag-and-drop deploy from local filesystem |
| **SSH/tunnel management** | Can't initiate SSH from browser | Manages Cloudflare tunnel connections natively |
| **Offline access** | Nothing | Cached dashboards, queued commands |
| **Keyboard shortcuts** | Conflicts with browser shortcuts | Full OS-level hotkeys |
| **Terminal integration** | Not possible | Embedded terminal, split views |

## 4. Architecture

### 4.1 Technology Stack — Tauri

Built with [Tauri](https://tauri.app/) — a lightweight, secure framework for cross-platform desktop applications.

| Factor | Tauri (Chosen) | Electron (Rejected) |
|---|---|---|
| **Binary size** | ~5-15 MB | ~150-200 MB |
| **Memory usage** | 30-50 MB | 200+ MB |
| **Renderer** | System webview (WebKit/WebView2/WebKitGTK) | Bundled Chromium |
| **Backend language** | Rust | Node.js |
| **Frontend** | Any web framework | Any web framework |
| **Security** | Rust (memory-safe, minimal IPC surface) | JS everywhere (larger attack surface) |
| **Auto-update** | Built-in updater | electron-updater |

**Why Tauri fits AxonStellar:**
- Frontend is web tech (HTML/CSS/JS) — existing web UI patterns, D3/Three.js visualizations, Alpine.js components transfer directly
- Rust backend handles secure operations — SSH tunnel management, credential storage, CF access token flow
- 5-15 MB binary vs 200 MB — faster installs, lighter footprint
- Tauri's explicit IPC allowlists align with AxonStellar's security-first architecture

### 4.2 Connection Model

Two communication channels through the same Cloudflare Tunnel:

```

  AEXYR Desktop                                               │
                                                              │
  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
  │ Prod    │  │ Dev     │  │ Client  │  ← Instance profiles │
  └────┬────┘  └────┬────┘  └────┬────┘                     │
       └────────┬───┴────────┬───┘                           │
          HTTPS/WSS      SSH (CF Tunnel)                      │
          (dashboards)   (agent chat)                         │

               │              │
          Cloudflare Edge Network
               │              │
       ┌───────┴──────────────┴────────┐
       │  AEXYR Container              │
       │  cloudflared tunnel           │
       │    ├── *.domain → :443 (HTTPS)│
       │    └── ssh.domain → :22 (SSH) │
       │  Flask API (:80) ← all data   │
       │  WebSocket ← real-time        │
       └───────────────────────────────┘
```

| Channel | Transport | Purpose |
|---|---|---|
| **HTTPS + WebSocket** | CF Tunnel → nginx :443 → Flask :80 | Dashboard data, topology, vitals, service status, logs, real-time updates |
| **SSH** | CF Tunnel → sshd :22 | Agent conversation, command execution, file operations |

### 4.3 Security

Same three-layer model as AEXYR CLI:

| Layer | Mechanism | Purpose |
|---|---|---|
| **1. Cloudflare Access** | SSO, email OTP, mTLS, IP allowlists | Identity verification at the edge |
| **2. SSH Key Auth** | Ed25519/RSA keypair | Cryptographic session authentication |
| **3. API Token** | Internal API token | Flask API authorization |

Additional desktop security:
- Credentials stored in OS-native keychain (macOS Keychain, Windows Credential Manager, Linux Secret Service)
- Tauri IPC allowlists restrict which Rust commands the frontend can invoke
- No server-side code running in the desktop app

## 5. User Interface

### 5.1 Layout

```

  AxonStellar                                          ─ □ ✕ │

           │                                                 │
  INSTANCES│  ┌─ Topology ─────────────────────────────────┐ │
           │  │                                             │ │
  ● prod   │  │     [Interactive 3D constellation           │ │
    acme   │  │      showing all services, connections,     │ │
           │  │      health status — topology view           │ │
  ● dev    │  │      rendered natively via Three.js/D3]     │ │
    acme   │  │                                             │ │
           │  └─────────────────────────────────────────────┘ │
  ○ client │                                                 │
    (off)  │  ┌─ Server Rack ──────┐  ┌─ Vitals ──────────┐ │
           │  │ inventory-api  ● UP │  │ CPU  ▓▓▓░░  34%   │ │
  ─────────│  │ postgres       ● UP │  │ MEM  ▓▓░░░  28%   │ │
           │  │ redis-cache    ● UP │  │ DISK ▓▓▓▓░  72%   │ │
  + Add    │  │ task-worker    ○ DN │  │ NET  ↑2.1 ↓0.8 MB │ │
  Instance │  └────────────────────┘  └────────────────────┘ │
           │                                                 │
           ├─────────────────────────────────────────────────┤
           │                                                 │
           │  ┌─ Agent Chat ───────────────────────────────┐ │
           │  │ ▌ You: add rate limiting to inventory-api  │ │
           │  │                                             │ │
           │  │ ▌ Aexyr: I'll add express-rate-limit...    │ │
           │  │   [████████████░░] Deploying...             │ │
           │  │                                             │ │
           │  │ > _                                         │ │
           │  └─────────────────────────────────────────────┘ │

```

### 5.2 Panels

| Panel | Data Source | Features |
|---|---|---|
| **Instance Sidebar** | Local config | Online/offline status, quick switch, add/remove |
| **Topology** | `GET /api_topology` | Interactive 3D service constellation, health indicators, click-to-inspect |
| **Server Rack** | `GET /manifests_list` | Service list, start/stop/restart via `POST /visualizer_service_control` |
| **Vitals** | `GET /visualizer_vitals` | Real-time CPU, memory, disk, network charts |
| **Agent Chat** | Agent chat API + WebSocket | Conversational interface, streaming responses, progress indicators |
| **Logs** | `GET /api_node_logs` | Tabbed log viewer, filterable by service, real-time tail |
| **Files** | `GET /get_work_dir_files`, etc. | Remote file browser, edit, drag-and-drop deploy |
| **SSL/Domains** | `GET /api_certificates`, `GET /api_assignments` | Certificate status, domain assignments, provision/assign actions |
| **Ops Center** | `GET /api_ops_system`, `GET /api_ops_network` | System metrics, network listeners, process manager |

All panels consume existing AEXYR APIs — no new backend needed.

### 5.3 System Tray

- Persistent system tray icon with instance status badges
- Quick actions: open dashboard, check health, view notifications
- Native OS notifications for critical events (service down, SSL expiring, build complete)
- Runs in background when window is closed

## 6. Build & Distribution

### 6.1 Platform Matrix

One codebase → three (or more) build targets:

| Platform | Architecture | Installer | Size |
|---|---|---|---|
| **macOS** | Apple Silicon (aarch64) | `.dmg` | ~15 MB |
| **macOS** | Intel (x86_64) | `.dmg` | ~15 MB |
| **macOS** | Universal (both) | `.dmg` | ~25 MB |
| **Windows** | x64 | `.msi` | ~15 MB |
| **Linux** | x64 | `.deb` | ~15 MB |
| **Linux** | x64 | `.AppImage` | ~15 MB |
| **Linux** | x64 | `.rpm` | ~15 MB |

### 6.2 CI/CD — GitHub Actions

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    tags: ['v*']

jobs:
  build:
    strategy:
      matrix:
        include:
          - platform: macos-latest
            target: aarch64-apple-darwin
          - platform: macos-latest
            target: x86_64-apple-darwin
          - platform: windows-latest
          - platform: ubuntu-latest

    runs-on: ${{ matrix.platform }}
    steps:
      - uses: actions/checkout@v4
      - uses: tauri-apps/tauri-action@v0
        with:
          tagName: v__VERSION__
          releaseName: 'AxonStellar Desktop v__VERSION__'
```

Push a git tag → GitHub Actions builds on all platforms → attaches all installers to a GitHub Release.

### 6.3 Auto-Update

Tauri has a built-in updater:
1. App launches → checks JSON endpoint (GitHub Releases or custom URL)
2. New version available → prompts user
3. User clicks Update → downloads in background → restarts

User installs once, auto-updates forever.

### 6.4 Code Signing

| Platform | Required? | Cost | Without It |
|---|---|---|---|
| **macOS** | Optional | $99/year (Apple Developer Program) | "Unidentified developer" warning — right-click → Open to bypass |
| **Windows** | Optional | ~$200-400/year (DigiCert etc.) | SmartScreen "unknown publisher" — click "Run anyway" |
| **Linux** | Not needed | Free | No gatekeeper |

**v1 strategy:** Skip code signing ($0 cost). Users dismiss warning once. Add signing for polished v2.

## 7. Technology Stack

| Component | Technology | Rationale |
|---|---|---|
| **Framework** | Tauri v2 | Lightweight, secure, cross-platform |
| **Backend** | Rust | Memory safety, credential handling, SSH tunnel management |
| **Frontend** | HTML/CSS/JS (framework TBD) | Web tech — patterns transfer from existing AEXYR web UI |
| **Visualization** | Three.js / D3.js | 3D topology constellation, charts |
| **State Management** | TBD | Frontend state for multi-instance data |
| **Styling** | TBD (Tailwind CSS candidate) | Consistent design system |
| **Build** | Tauri CLI + GitHub Actions | Cross-platform CI/CD |
| **Installer** | Tauri bundler (built-in) | .dmg, .msi, .deb, .AppImage, .rpm |
| **Auto-Update** | Tauri updater (built-in) | Seamless update flow |
| **Keychain** | OS-native (via Rust crates) | Secure credential storage |

### Frontend Framework Candidates

| Framework | Pros | Cons |
|---|---|---|
| **Vanilla JS + Alpine.js** | Matches existing AEXYR web UI, minimal bundle | Less structure for complex app |
| **React** | Large ecosystem, strong state management | Heavier, JSX build step |
| **Svelte** | Lightweight compiled output, reactive | Smaller ecosystem |
| **SolidJS** | React-like but faster, fine-grained reactivity | Newer, smaller community |

Decision deferred to scaffolding phase.

## 8. File Structure (Planned)

```
aexyr-desktop/
 src-tauri/                      # Rust backend
   ├── src/
   │   ├── main.rs                 # Entry point
   │   ├── commands/               # IPC command handlers
   │   │   ├── connection.rs       # Instance connection management
   │   │   ├── tunnel.rs           # cloudflared SSH tunnel wrapper
   │   │   ├── auth.rs             # CF Access + credential management
   │   │   └── system.rs           # OS integration (tray, notifications)
   │   ├── state/
   │   │   └── instances.rs        # Multi-instance state management
   │   └── keychain/
   │       └── mod.rs              # OS-native credential storage
   ├── Cargo.toml
   ├── tauri.conf.json             # Tauri configuration
   └── icons/                      # App icons for all platforms
 src/                            # Web frontend
   ├── index.html                  # Entry HTML
   ├── js/
   │   ├── app.js                  # App initialization
   │   ├── components/
   │   │   ├── topology.js         # 3D constellation view
   │   │   ├── server-rack.js      # Service management panel
   │   │   ├── vitals.js           # System metrics charts
   │   │   ├── agent-chat.js       # Agent conversation UI
   │   │   ├── logs.js             # Log viewer
   │   │   ├── files.js            # Remote file browser
   │   │   ├── ssl.js              # SSL/domain management
   │   │   └── instance-sidebar.js # Instance selection
   │   ├── lib/
   │   │   ├── api-client.js       # AEXYR API wrapper
   │   │   ├── websocket.js        # Real-time data connection
   │   │   └── tauri-bridge.js     # Tauri IPC communication
   │   └── stores/
   │       └── instances.js        # Instance state store
   ├── css/
   │   └── app.css                 # Application styles
   └── assets/
       └── icons/
 tests/
 .github/
   └── workflows/
       └── release.yml             # Cross-platform build CI/CD
 package.json
 README.md
 LICENSE
 service_blueprint.md
 service_changelog.md
 service_pitfalls.md
```

## 9. Multi-Instance Management

The killer feature that distinguishes the desktop app from the web UI.

### 9.1 Instance Profiles

```json
// Stored in app data directory + OS keychain for secrets
{
  "instances": [
    {
      "id": "prod-acme",
      "name": "Production",
      "host": "ssh.acme.example.com",
      "apiUrl": "https://acme.example.com",
      "auth": "cloudflare_access",
      "icon": "🏭",
      "color": "#ef4444"
    },
    {
      "id": "dev-acme",
      "name": "Development",
      "host": "ssh.dev.acme.example.com",
      "apiUrl": "https://dev.acme.example.com",
      "auth": "cloudflare_access",
      "icon": "🔧",
      "color": "#3b82f6"
    }
  ]
}
```

### 9.2 Fleet Overview (v3)

Future unified dashboard showing all instances simultaneously:
- Aggregated health metrics
- Cross-instance service discovery
- Rolling deployments across fleet
- Centralized log aggregation

## 10. AEXYR API Dependencies

All consumed APIs already exist — no backend changes needed:

| Panel | AEXYR API Endpoint |
|---|---|
| Topology | `GET /api_topology`, `GET /api_node`, `GET /api_node_config` |
| Server Rack | `GET /manifests_list`, `POST /visualizer_service_control` |
| Vitals | `GET /visualizer_vitals`, `GET /system_resources` |
| Agent Chat | Agent chat WebSocket/HTTP |
| Logs | `GET /api_node_logs` |
| Files | `GET /get_work_dir_files`, `GET/POST /edit_work_dir_file` |
| SSL | `GET/POST /api_certificates`, `GET/POST /api_assignments` |
| Ops | `GET /api_ops_system`, `GET /api_ops_network`, `GET /api_pm/status` |
| Notifications | `POST /notifications_history`, `POST /notification_create` |
| Scheduler | `POST /scheduler_tasks_list`, `POST /scheduler_task_create` |

## 11. Roadmap

| Phase | Version | Milestone |
|---|---|---|
| **Planning** | 0.1.0 | Architecture, spec, UI design, technology decisions (current) |
| **Scaffold** | 0.2.0 | Tauri project setup, Rust backend skeleton, frontend shell |
| **Single Instance** | 0.3.0 | Connect to one AEXYR instance, render topology + vitals |
| **Core Panels** | 0.4.0 | Server Rack, Logs, Files panels |
| **Agent Chat** | 0.5.0 | Integrated agent conversation with streaming |
| **System Tray** | 0.6.0 | Background mode, tray icon, native notifications |
| **SSL/Domains** | 0.7.0 | Certificate management, domain assignment panel |
| **Multi-Instance** | 0.8.0 | Instance sidebar, profile switching, per-instance state |
| **Polish** | 0.9.0 | Keyboard shortcuts, themes, error handling, onboarding |
| **v1 Release** | 1.0.0 | Cross-platform builds, auto-updater, GitHub Release |
| **Signed** | 1.1.0 | Code signing for macOS + Windows |
| **Fleet** | 2.0.0 | Unified fleet dashboard, aggregated metrics |

## 12. User Experience Flow

### First Launch
1. Download installer for their OS from GitHub Releases
2. Install (drag to Applications / run MSI / dpkg)
3. Launch AxonStellar Desktop
4. Click "+ Add Instance" → enter host URL
5. Browser opens for Cloudflare Access authentication
6. SSH key exchange happens automatically
7. Dashboard populates with live data from their AEXYR instance

### Daily Use
1. App is running in system tray (always connected)
2. Click tray icon or hotkey to open
3. See instance status at a glance in sidebar
4. Click into any instance → full dashboard
5. Chat with agent in bottom panel while monitoring topology above
6. Receive native OS notifications for critical events

## 13. Cost Summary

| Item | Cost | Required? |
|---|---|---|
| Tauri framework | Free (MIT) | Yes |
| Rust toolchain | Free | Yes |
| GitHub Actions CI/CD | Free (public repo) | Yes |
| Apple Developer Program | $99/year | No (v1) |
| Windows code signing | ~$200-400/year | No (v1) |
| Domain for update server | Already have | — |
| **Total for v1** | **$0** | — |
