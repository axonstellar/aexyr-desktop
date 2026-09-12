# AEXYR Desktop — Pitfalls & Lessons Learned

Known issues, design considerations, and things to watch out for.

---

## Platform & Build

### 1. No Single Cross-Platform Binary
**Fact:** There is no such thing as a single binary that runs on macOS, Windows, and Linux. Each OS has a completely different executable format (Mach-O, PE, ELF).  
**Solution:** One codebase, three builds via GitHub Actions. Each build runs on a platform-specific runner and produces platform-specific installers.  
**Date:** 2026-09-11

### 2. macOS Architecture Split
**Issue:** macOS has two active architectures: Apple Silicon (aarch64) and Intel (x86_64). Need to build for both, or create a Universal binary containing both.  
**Consideration:** Apple Silicon is now dominant. For v1, consider aarch64-only or Universal. Intel-only users are a shrinking minority.  
**Date:** 2026-09-11

### 3. Code Signing Warnings
**Issue:** Without code signing certificates, macOS shows "unidentified developer" and Windows SmartScreen shows "unknown publisher".  
**v1 Strategy:** Skip signing ($0 cost). Users dismiss once via right-click → Open (macOS) or "Run anyway" (Windows). Add signing in v1.1.0 when product is validated.  
**Cost:** Apple $99/yr, Windows $200-400/yr.  
**Date:** 2026-09-11

### 4. System Webview Differences
**Issue:** Tauri uses the system webview (WebKit on macOS, WebView2 on Windows, WebKitGTK on Linux). Rendering can differ across platforms.  
**Mitigation:** Test on all three platforms. Avoid platform-specific CSS features. WebView2 (Windows) requires a runtime that's pre-installed on Windows 10 21H2+ and Windows 11.  
**Date:** 2026-09-11

## Connectivity

### 5. cloudflared Dependency
**Issue:** Cloudflare Tunnel SSH requires `cloudflared` installed on the user's machine.  
**Options:** (a) Document as a prerequisite, (b) Bundle cloudflared with the app, (c) Auto-download on first connection.  
**Consideration:** Bundling increases binary size. Auto-download needs admin permissions on some systems. Documenting as prerequisite is simplest for v1.  
**Date:** 2026-09-11

### 6. Connection State Management
**Issue:** Managing connections to multiple instances simultaneously, with different auth states, network conditions, and reconnection logic.  
**Consideration:** Each instance connection should be independent with its own reconnection backoff. A dropped connection to Instance A shouldn't affect Instance B.  
**Date:** 2026-09-11

### 7. WebSocket Through Cloudflare Tunnel
**Issue:** WebSocket connections for real-time updates (topology, vitals, logs) need to persist through the Cloudflare Tunnel.  
**Consideration:** Cloudflare supports WebSocket proxying but has idle timeout defaults. Implement ping/pong keepalive. Handle reconnection gracefully.  
**Date:** 2026-09-11

## Security

### 8. Credential Storage
**Issue:** The app stores connection credentials and API tokens for multiple instances.  
**Solution:** Use OS-native keychain (macOS Keychain, Windows Credential Manager, Linux Secret Service) via Rust crates. Never store secrets in plain text config files. Never log credentials.  
**Date:** 2026-09-11

### 9. Tauri IPC Surface
**Issue:** Tauri's IPC bridge between the web frontend and Rust backend is a potential attack surface if misconfigured.  
**Solution:** Use Tauri's explicit allowlist to restrict which Rust commands the frontend can invoke. Validate all IPC arguments in Rust. Never expose raw shell execution through IPC.  
**Date:** 2026-09-11

## UI/UX

### 10. Æ Character Rendering
**Issue:** The Æ (ash) character renders correctly in some font weights/contexts but disappears in others (observed on GitHub).  
**Solution:** Use HTML entity `&#198;` or plain `AEXYR` in contexts where rendering is unreliable. Test across platforms.  
**Date:** 2026-09-11

### 11. Three.js in System Webview
**Issue:** The 3D topology constellation uses Three.js/WebGL. System webviews may have different WebGL support levels compared to Chrome.  
**Mitigation:** Test WebGL support on all platform webviews. Provide a 2D fallback (D3.js force graph) if WebGL is unavailable.  
**Date:** 2026-09-11
