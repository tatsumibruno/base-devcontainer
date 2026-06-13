# Base Development Container

A ready-to-use Ubuntu 24.04 development container pre-installed with:

- Git, Zsh, build tools, Python 3, Node.js/npm
- Docker CLI (talks to the host Docker daemon via socket mount — [see security note](#-docker-socket-security-note))
- AI CLI tools: `claude` (Claude Code), `codex` (OpenAI Codex), `opencode`
- [rtk](https://github.com/rtk-ai/rtk) wired into Claude Code for token savings

The `workspace/` folder inside `.devcontainer/` is mounted as `/workspace` inside the container — put your project files there.

---

## Getting started

Clone the repo and enter it:

```bash
git clone git@github.com:tatsumibruno/base-devcontainer.git your-project-name
cd your-project-name
```

Then follow one of the two options below.

---

## Option 1 — VS Code Dev Container

**Prerequisites:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) and the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) installed.

### SSH agent forwarding (required for Git over SSH)

The container reads your SSH agent socket via `$SSH_AUTH_SOCK`. Make sure your SSH key is loaded before opening the container:

```bash
ssh-add ~/.ssh/id_ed25519   # or your key
ssh-add -l                  # verify it's listed
```

### Steps

1. Open this folder in VS Code:

   ```bash
   code .
   ```

2. When prompted, click **Reopen in Container** — or open the Command Palette (`⇧⌘P`) and run:

   ```
   Dev Containers: Reopen in Container
   ```

3. VS Code will build the image (first run only) and attach. The integrated terminal opens directly inside `/workspace`.

4. The following VS Code extensions are auto-installed inside the container:
   - Docker
   - ESLint
   - Prettier

### Rebuild after Dockerfile changes

```
Dev Containers: Rebuild Container
```

---

## Option 2 — Docker Compose + docker exec

**Prerequisites:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine + Compose plugin).

### Start the container

```bash
cd .devcontainer

# Build the image and start the container in the background
docker compose up -d --build
```

On subsequent runs (no Dockerfile changes):

```bash
docker compose up -d
```

### Enter the workspace

```bash
docker compose exec dev zsh
```

You are now inside `/workspace` as `root`. Files you create there are persisted in `.devcontainer/workspace/` on the host.

### Stop the container

```bash
docker compose down
```

### Useful aliases

```bash
# Shortcut to jump into the container from anywhere
alias devshell='docker compose -f /path/to/base-devcontainer/.devcontainer/docker-compose.yml exec dev zsh'
```

### Rebuild after Dockerfile changes

```bash
docker compose build --no-cache
docker compose up -d
```

---

## ⚠️ Docker socket security note

By default, `docker-compose.yml` mounts the host Docker socket into the container:

```yaml
- /var/run/docker.sock:/var/run/docker.sock
```

**Why it's there:** it lets you run `docker` and `docker compose` commands from inside the container to spin up sibling containers (databases, services, etc.) without installing a full Docker daemon inside the container itself. This pattern is called Docker-out-of-Docker (DooD).

**The risk:** anything running inside the container — including AI tools like `claude` or `codex` — can use this socket to control the host Docker daemon with full privileges. In practice this means a sufficiently crafty prompt or a malicious package could escape the container and get root access on your host machine.

**Concrete attack surface:**

```bash
# Any process inside the container can do this:
docker run --rm -v /:/host alpine chroot /host sh
# → root shell on the host
```

### Disabling the Docker socket

If you don't need to run Docker commands from inside the container, remove the socket mount.

**`docker-compose.yml`** — delete the socket line:

```yaml
volumes:
  - ./workspace:/workspace
  - ./claude-settings.json:/workspace/.claude/settings.json
  - ./codex-config.toml:/workspace/.codex/config.toml
  # - /var/run/docker.sock:/var/run/docker.sock  ← remove this line
```

**`devcontainer.json`** — no changes needed; the socket is only declared in the Compose file.

After saving, rebuild:

```bash
# Option 1
Dev Containers: Rebuild Container

# Option 2
docker compose down && docker compose up -d --build
```

With the socket removed, `docker` commands will fail inside the container, but all AI tools and development workflows remain fully functional.

---

## Project files

Put your code in `.devcontainer/workspace/`. It is `.gitignore`-d by default so you can manage it as its own independent repository.

```
base-devcontainer/
└── .devcontainer/
    ├── Dockerfile          # Container image definition
    ├── docker-compose.yml  # Compose config (used by both options)
    ├── devcontainer.json   # VS Code Dev Container config
    ├── claude-settings.json  # Claude Code permissions (mounted into container)
    ├── codex-config.toml   # Codex CLI config (mounted into container)
    └── workspace/          # ← your project files live here
```

## AI tools inside the container

| Tool | Command | Config |
|------|---------|--------|
| Claude Code | `claude` | `/workspace/.claude/settings.json` (from `claude-settings.json`) |
| OpenAI Codex | `codex` | `/workspace/.codex/config.toml` (from `codex-config.toml`) |
| opencode | `opencode` | — |
| rtk | `rtk` | wired into Claude Code hook automatically |

All tools run with auto-approval (no confirmation prompts), scoped to `/workspace`.
