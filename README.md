# GitHub Copilot CLI - Dev Container Setup

A secure, consistent environment for using GitHub Copilot CLI. Works on Windows and macOS.

---

## Prerequisites

Install these once on your machine.

**Required for everyone:**

| Tool                             | Download                                                                                                                                  |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| VS Code                          | https://code.visualstudio.com                                                                                                             |
| VS Code Dev Containers extension | Search "Dev Containers" in VS Code Extensions, or: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers |

**Then choose one container runtime:**

### Option A — Docker Desktop

Download and install Docker Desktop: https://www.docker.com/products/docker-desktop

On **Windows**: make sure Docker Desktop is set to use WSL 2 (the default). You do not need to install WSL yourself.

### Option B — Rancher Desktop

Download and install Rancher Desktop: https://rancherdesktop.io — a free, open-source alternative to Docker Desktop, used here to avoid Docker Desktop's commercial license requirements for organizational use.

On first launch, in Rancher Desktop's preferences, set the **Container Engine** to **dockerd (moby)** (not `containerd`). This gives you a Docker-compatible engine and CLI, so no VS Code settings changes are needed — the Dev Containers extension detects it the same way it detects Docker Desktop.

---

## Using the container

> **For each new folder you want to work in:** copy the `.devcontainer/` folder into it first.

1. Ensure your container runtime (Docker Desktop or Rancher Desktop) is running.
2. Open your working folder in VS Code (`File > Open Folder`).
3. Click **"Reopen in Container"** when prompted, or open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`) and run **Dev Containers: Reopen in Container**.
   - The first build takes a few minutes; subsequent opens are fast.
4. Run `copilot` in the terminal to start, then use `/login` to authenticate.

---

## What is and isn't accessible inside the container

- **Only the folder you opened** is accessible inside the container. Copilot cannot see other files on your machine.
- Organisation security settings are enforced via server-managed settings and cannot be changed.

### Network access

A firewall runs automatically on every container start and restricts outbound traffic to a fixed allowlist:

| Service                                                                                                                                  | Purpose                          |
| ---------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| `api.githubcopilot.com`, `api.{individual,business,enterprise}.githubcopilot.com`, `copilot-proxy.githubusercontent.com`                 | GitHub Copilot CLI               |
| `login.microsoftonline.com`                                                                                                              | Authentication                   |
| GitHub IP ranges (web, API, git), `objects.githubusercontent.com`                                                                        | Git operations, `gh`, downloads  |
| SSH (port 22, any host)                                                                                                                  | Git over SSH                     |
| `pypi.org`, `files.pythonhosted.org`                                                                                                     | Python package installs (`pip`)  |
| `registry.npmjs.org`                                                                                                                     | Node.js package installs (`npm`) |
| `binaries.soliditylang.org`                                                                                                              | Solidity compiler downloads      |

All other outbound internet access is blocked. If a tool or script tries to reach an unlisted host it will be rejected immediately (ICMP admin-prohibited).

The `NET_ADMIN`/`NET_RAW` capabilities used by the firewall are scoped to the container's own network namespace and grant no privilege on your host.

The firewall is (re-)applied via `sudo init-firewall.sh` every time the container starts (`postStartCommand`). The `vscode` user's passwordless `sudo` is locked down to exactly that one script (`/usr/local/bin/init-firewall.sh`) — the base image's default blanket passwordless sudo for `vscode` is removed, so any other `sudo` command prompts for a password.

### Included tooling

- **Node.js** (via the `node` dev container feature, installed through nvm) and **Python 3** are both preinstalled, so `npm` and `pip`/`python3` work out of the box against the allowlisted registries above.
- **zsh** (with the Powerlevel10k theme) is the default shell, in both the integrated VS Code terminal and any terminal you open.
- Common CLI utilities are included: `git`, `gh` (GitHub CLI), `curl`, `jq`, `vim`, `nano`, `unzip`, `build-essential`.

### Persistent configuration

Copilot's configuration and state (`~/.copilot/`) are stored in a named Docker volume (`devc-<folder-name>-config-<container-id>`), not inside the container filesystem, so settings and login survive container rebuilds. Shell command history is similarly persisted in its own named volume.

---

## For administrators

Copilot policy (models, features, MCP) is managed through your GitHub organisation or enterprise Copilot settings.
