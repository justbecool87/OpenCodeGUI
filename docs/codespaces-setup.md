# Running OpenCode GUI in GitHub Codespaces (recommended remote setup)

This is the lowest-maintenance way to run this extension against the OpenCode CLI:
the VS Code extension host, the `opencode` CLI, and your repository all live in the
same Linux machine, so there is no port forwarding, no SSH tunneling, and no
cross-device reachability to manage. Any provider configured in opencode — including
GitHub Copilot models and OpenCode zen models — shows up in the GUI's model picker
automatically, because the extension queries models from the running opencode server.

## 1. Open the repository in a Codespace

This repository ships a `.devcontainer/devcontainer.json` that:

- starts from a Ubuntu base image with Node.js 20,
- installs the OpenCode CLI globally (`npm install -g opencode-ai`),
- installs this extension's dependencies and compiles it (`npm ci && npm run compile`).

Open the repo in a Codespace (Code → Codespaces → Create) or use
"Dev Containers: Reopen in Container" in desktop VS Code with the Dev Containers
extension. Wait for `postCreateCommand` to finish, then verify:

```bash
opencode --version
```

## 2. Sign in to your providers

Authentication is stored per-Codespace under `~/.local/share/opencode`, so you sign
in once per Codespace:

```bash
opencode auth login
```

- **GitHub Copilot**: choose the GitHub Copilot provider and complete the device
  flow in the browser. Your paid Copilot models then appear in the sidebar model
  picker, including quota/speed-multiplier display.
- **OpenCode zen models**: select the opencode provider; zen models are included.

## 3. Run the extension

- **Use it on this repository**: press `F5` (Run Extension) to launch an Extension
  Development Host inside the Codespace, then open the OpenCode sidebar. In a
  browser-based Codespace the development host opens as a new VS Code Web tab.
- **Use it in other repositories**: package the extension once
  (`npx @vscode/vsce package`) and install the resulting `.vsix` into your
  Codespaces via Settings Sync or "Install from VSIX...". In any Codespace with
  `opencode` on `PATH`, the extension manages the opencode server per workspace
  itself (`.opencode/server.lock.json`) — nothing else to configure.

## Alternative: attaching to an external opencode server

Only needed when the opencode CLI cannot run inside the Codespace (for example, it
runs on another machine such as an Ubuntu-on-Termux device). The extension reuses
an existing server when a lock file matches a healthy server:

1. Forward the remote server port to the Codespace's localhost, e.g.
   `ssh -L 4096:127.0.0.1:4096 <remote-host> -p <port>`.
2. On the remote host, start opencode with a known password:
   `OPENCODE_SERVER_PASSWORD=<password> opencode serve --port 4096`.
3. In the workspace, write `.opencode/server.lock.json`:

   ```json
   {
     "workspaceRoot": "/workspaces/<repo>",
     "port": 4096,
     "password": "<password>",
     "updatedAt": "2026-01-01T00:00:00Z"
   }
   ```

4. Reload the VS Code window. The health check (`/global/health` with Basic Auth)
   succeeds and the extension attaches instead of spawning a new server.

Prefer the in-Codespace install whenever possible — it avoids keeping tunnels
alive and is the configuration covered by the devcontainer in this repository.
