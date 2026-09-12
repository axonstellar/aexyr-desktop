<div align="center">

# Æxyr Desktop

**Cross-platform mission control for your AEXYR infrastructure**

*A native desktop application for managing one or many AEXYR instances from a unified control plane.*

![Status](https://img.shields.io/badge/Status-Coming_Soon-yellow)
![Platforms](https://img.shields.io/badge/Platforms-macOS_%7C_Windows_%7C_Linux-4A154B)
![License](https://img.shields.io/badge/License-MIT-green)
![Framework](https://img.shields.io/badge/Framework-Tauri_v2-24C8D8?logo=tauri&logoColor=white)

---

**[AEXYR](https://github.com/axonstellar/AEXYR)** · **[AEXYR CLI](https://github.com/axonstellar/aexyr-cli)** · **[AxonStellar](https://axonstellar.com)**

</div>

---

## What is AEXYR Desktop?

AEXYR Desktop is a native desktop application for managing your [AEXYR](https://github.com/axonstellar/AEXYR) infrastructure. Built with Tauri, it delivers a lightweight, fast, and secure experience across macOS, Windows, and Linux — with binary sizes around 15 MB and minimal resource usage.

The killer feature: **multi-instance management**. Connect to multiple AEXYR servers from a single window and manage your entire fleet from one place.

---

## Planned Features

### 🖥️ Native Desktop Experience
A proper desktop application — not a browser tab. Native window management, system tray integration, keyboard shortcuts, and OS-level notifications. Lightweight footprint with minimal CPU and memory usage.

### 🌐 Multi-Instance Management
Connect to one or many AEXYR servers from a single application. A sidebar lists all connected instances with live status indicators. Switch between servers instantly — no juggling browser tabs or SSH sessions.

### 🗺️ Network Topology
Interactive visualization of your service constellation. Drag nodes, trace connections, inspect health, and view configuration — all rendered natively with smooth performance.

### 🖧 Server Rack
Your constellation command center. View all deployed services as live cards with health status, port assignments, and one-click lifecycle controls. Start, stop, and restart services directly from the desktop.

### 📊 System Vitals
Real-time CPU, memory, disk, and process monitoring with historical charts. Keep a vitals panel open while you work — no need to switch to a browser.

### 💬 Agent Chat
Converse with your AEXYR agent natively. Send tasks, watch execution in real time, and review results — all within the desktop application.

### 🔒 SSL & Domain Management
Provision certificates, assign domains, and monitor SSL status across all your instances from a unified interface.

### 📁 File Browser
Browse, edit, and manage files across your AEXYR user space. Syntax-highlighted editor for quick edits without leaving the app.

### 🔐 Secure Connection
Connects to your AEXYR instances over encrypted tunnels. Credentials stored in your operating system's native keychain — macOS Keychain, Windows Credential Manager, or Linux Secret Service.

---

## Installation

> **Coming soon.** The desktop application is currently in development.

### macOS
```bash
# Download the .dmg from GitHub Releases
# Double-click → drag to Applications
```

### Windows
```bash
# Download the .msi from GitHub Releases
# Double-click → follow the installer
```

### Linux
```bash
# Debian/Ubuntu
sudo dpkg -i aexyr-desktop_1.0.0_amd64.deb

# Universal
chmod +x AexyrDesktop_1.0.0_amd64.AppImage
./AexyrDesktop_1.0.0_amd64.AppImage
```

---

## The AEXYR Ecosystem

AEXYR Desktop is part of the AEXYR product family — three interfaces to the same powerful infrastructure:

| Product | Interface | Status |
|---|---|---|
| **[AEXYR](https://github.com/axonstellar/AEXYR)** | Web UI — full-featured browser dashboard | ✅ Available |
| **[AEXYR CLI](https://github.com/axonstellar/aexyr-cli)** | Terminal — scriptable command-line interface | 🔜 Coming Soon |
| **[AEXYR Desktop](https://github.com/axonstellar/aexyr-desktop)** | Desktop — native multi-instance mission control | 🔜 Coming Soon |

All three share the same APIs and connect to the same AEXYR server. Use whichever interface fits your workflow — or use all three.

---

## Technology

| Component | Technology |
|---|---|
| **Framework** | [Tauri v2](https://v2.tauri.app/) |
| **Backend** | Rust |
| **Frontend** | HTML, CSS, JavaScript |
| **Binary Size** | ~15 MB per platform |
| **Memory Usage** | ~30-50 MB |

Built with Tauri instead of Electron — resulting in dramatically smaller binaries, lower resource usage, and native performance. Uses the system webview (WebKit on macOS, WebView2 on Windows, WebKitGTK on Linux) rather than bundling an entire Chromium instance.

---

## Platform Support

| Platform | Architecture | Installer |
|---|---|---|
| **macOS** | Apple Silicon (M1–M4) | `.dmg` |
| **macOS** | Intel | `.dmg` |
| **Windows** | x64 | `.msi` |
| **Linux** | x64 | `.deb`, `.AppImage`, `.rpm` |

Builds are automated via GitHub Actions — each release produces installers for all supported platforms simultaneously.

---

## Requirements

- **An AEXYR instance** to connect to ([get started here](https://github.com/axonstellar/AEXYR))
- macOS 11+, Windows 10+, or a modern Linux distribution

---

## License

AEXYR Desktop is open-source software released under the [MIT License](LICENSE).

The AEXYR platform itself is proprietary software by **AxonStellar LLC**. See the [AEXYR repository](https://github.com/axonstellar/AEXYR) for platform licensing details.

---

<div align="center">

*Your infrastructure, your desktop, your control.* 🖥️

**Built by [AxonStellar](https://axonstellar.com)**

</div>
