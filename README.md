# Tessl Docker Sandboxes Kit

Docker Sandboxes kit that installs the [Tessl CLI](https://tessl.io), authenticates it via your stored Tessl API key, and wires the Tessl MCP server into the sandboxed agent so the agent can search and install skills from the Tessl registry.

## Quick start

1. **Create a Tessl API key** (run on the host, in any shell where you're logged in to Tessl):

   ```bash
   tessl api-key create
   ```

2. **Store it as a sandbox secret.** The key lands in your OS keychain; the real value never enters the sandbox.

   ```bash
   sbx secret set -g tessl
   # paste the key when prompted
   ```

3. **Run a sandbox with the kit attached.** Replace `.` with the workspace path you want.

   ```bash
   sbx run --kit git+https://github.com/shelajev/tessl-sbx-kit.git claude .
   ```

That's it. Inside the sandbox the agent has `tessl` on PATH, an `.mcp.json` wiring Tessl's MCP server into the agent, and an initialized `tessl.json`. Try `tessl whoami` to confirm auth, or ask the agent to "search the Tessl registry".

## Named sandbox

For a persistent sandbox you can re-attach to:

```bash
sbx create --name tessl-current \
  --kit git+https://github.com/shelajev/tessl-sbx-kit.git claude .

sbx run tessl-current
```

For custom kits, pass `--kit` again when re-running an existing sandbox if `sbx` doesn't resolve it automatically.

## How it works

The kit declares a `tessl` service in `spec.yaml`. The Docker Sandboxes proxy reads your stored `tessl` secret on the host and substitutes `Authorization: Bearer <token>` into outbound requests to `api.tessl.io`. Inside the sandbox, `TESSL_TOKEN` is set to a sentinel value (`proxy-managed`) so the CLI sees a non-empty env var without ever seeing the real key.

Install (runs once at sandbox creation, as the agent user):

- `curl -fsSL https://get.tessl.io | sh` — installs `tessl` to `~/.local/bin` (already on PATH)

Startup (runs each boot, idempotent):

- `tessl init --agent claude-code --agent codex --agent copilot --agent gemini --agent agents` — writes per-agent MCP/config files; only the running agent's file is read
- Appends a short Tessl usage hint to `CLAUDE.md` so the agent discovers the CLI

## Network policy

The kit allows only `*.tessl.io` and `tessl.io`. Skills sourced from GitHub (`tessl install <github-url>`) won't work unless you fork the kit and add the GitHub domains to `network.allowedDomains`.

## Local clone

If you clone this repo, `run.sh` runs a named sandbox using the local kit path:

```bash
./run.sh tessl-current
```

Use any sandbox name as the first argument:

```bash
./run.sh my-sandbox
```

## License

Apache 2.0. See [LICENSE](LICENSE).
