# Docker sbx setup

Notes on running coding agents in Docker Sandboxes (`sbx`) on this machine. Written 2026-10-18.

## Current state

- `sbx` is installed (winget package `Docker.sbx`) and signed in. The Docker Subscription Service Agreement was accepted on first run.
- Running sandbox: `pi-assistant` (agent `pi`, workspace `C:\Users\lars\assistant`).
- Native pi was uninstalled by deleting `C:\Users\lars\.pi`. Pi now only runs inside sbx.
- No sbx secrets are stored yet (`sbx secret ls` shows none).

## What the license agreement covers

- The "Agentic Platform Services" and "Sandbox" parts apply to sbx.
- Free "personal" accounts are for non-commercial use. Company work may need a paid plan. Docker docs say the `sbx` CLI and local sandboxes are free, including for commercial work.
- Input and output belong to the user. Docker will not train models on them without admin consent.
- Docker owns usage data and may use it in aggregated, de-identified form.
- Every action an AI agent takes counts as the user's own action. Docker disclaims liability for agent actions and output.
- Third-party model providers and MCP servers are covered by their own terms.

## Pi kit (`sbx/pi`)

- Run with `sbx run sbx/pi:latest`. It is an experimental Sandbox Kit v3 workload with pi baked into the image.
- It declares only an Anthropic credential. `--model` and `--provider` do not work with v3 kits.
- The `latest` tag follows the kit. The version tag is rebuilt nightly, so pin a digest for reproducibility: `sbx run "docker.io/sbx/pi@sha256:<digest>"`.
- Credentials and kit content are fixed when the sandbox is created. To change them, run `sbx rm -f <name>` and create it again.
- Do not run `/login` inside the sandbox. It writes a real token into `~/.pi/agent/auth.json` in the VM, where the agent can read it.

## Skills

- Host store: `C:\Users\lars\AppData\Local\DockerSandboxes\sandboxes\state\agent-skills`. List with `sbx skills ls`.
- The store is mounted read-only at `~/.agents/skills` in the sandbox, which pi reads. Use `--skills readwrite` or `--skills off` when creating a sandbox to change that.
- Skills in the store: `grill-me`, `datasolvr-api`, `datasolvr-cli`, `datasolvr-config`. Lars added these himself.
- On 2026-10-18 the other 37 skills, mostly from Matt Pocock's repo (https://github.com/mattpocock/skills), were removed with `sbx skills rm --force`.
- To add skills from a repository: `sbx skills add <git url>`. Superpowers is at https://github.com/obra/superpowers.
- Skills in `C:\Users\lars\.agents\skills` (the AWS skills) are a separate folder and are not part of the sbx store.

## MCP

- The sbx MCP gateway (`sbx mcp add`, `--static-mcp`) supports Claude Code, Codex, Devin, Gemini, Kiro and OpenCode. Pi is not supported.
- Pi has no built-in MCP client. The plan is the `pi-mcp-adapter` extension.
- Local stdio MCP servers run on the host, outside the sandbox.

## Network policy

- Policy is deny-by-default. List with `sbx policy ls`. Check a host with `sbx policy check network <host>`.
- Checked allowed on 2026-10-18: `api.githubcopilot.com`, `api.individual.githubcopilot.com`, `api.github.com`, `github.com`, `registry.npmjs.org`.
- Checked denied: `api.exa.ai`, `mcp.context7.com`. Both need allow rules.
- The global `local-policy` allows 194 network entries. Some are broad wildcards, so review and trim them.

## Goal: pi with Copilot credentials and plugins

Chosen approach: a local v3 kit set made of `sbx/pi` (pinned by digest) plus a mixin kit `pi-lars`, run with `sbx run ./<kit-set> --clone`. Not published anywhere for now.

### Plugins to bake in at build time (`pi install npm:<name>`)

| Package | Purpose | Setup needed |
|---|---|---|
| `agent-ste` | Forces Simplified Technical English on writes, commits and replies. Strict. | None |
| `pi-mcp-adapter` | MCP client for pi | Allow MCP hosts, adapter config |
| `@juicesharp/rpiv-web-tools` | `web_search` and `web_fetch` | Provider API key, allow provider host |
| `@juicesharp/rpiv-ask-user-question` | Ask the user questions | None expected |
| `@juicesharp/rpiv-todo` | Todo list | None expected |
| `@juicesharp/rpiv-btw` | Side questions | None expected |

Only the first three were read in detail. The last three were not checked.

### Decisions

- Web search provider: Exa. Key stored on the host with `sbx secret set-custom --host api.exa.ai`. Allow `api.exa.ai`.
- Network policy for `web_fetch`: strict allow list. Fetches to unlisted hosts fail.
- MCP servers: Context7 (`https://mcp.context7.com/mcp`, API key header) and GitHub (`https://api.githubcopilot.com/mcp/`, Bearer token). Endpoints and headers are from memory and not yet verified against current docs.

### Copilot authentication (open question)

What was found in pi's source:

- Pi's `github-copilot` provider reads the `COPILOT_GITHUB_TOKEN` environment variable. It deliberately ignores `GH_TOKEN` and `GITHUB_TOKEN`.
- Its `/login` is a GitHub device flow using client id `Iv1.b507a08c87ecfe98`. It exchanges the long-lived token at `https://api.github.com/copilot_internal/v2/token` for a short-lived Copilot token (`proxy-ep=...` selects the API host, normally `api.individual.githubcopilot.com`).
- Required request headers: `User-Agent: GitHubCopilotChat/0.35.0`, `Editor-Version: vscode/1.107.0`, `Editor-Plugin-Version: copilot-chat/0.35.0`, `Copilot-Integration-Id: vscode-chat`.

Idea being tested: keep the token on the host and let the sbx proxy substitute it.

1. A helper script `C:\Users\lars\.sbx-helpers\copilot-token.ps1` takes the `gh auth token` value, exchanges it for a Copilot token and prints it. `-Check` prints only metadata, never the token.
2. Register it as a custom secret: `sbx secret set-custom --host api.individual.githubcopilot.com --env COPILOT_GITHUB_TOKEN --command 'pwsh -NoProfile -File C:\Users\lars\.sbx-helpers\copilot-token.ps1' --refresh 20m`
3. Create a sandbox and check that `pi -p "hello"` works with the placeholder.

Not yet verified:

- Whether a `gh auth token` value is accepted by the exchange endpoint. Tokens issued to the GitHub CLI app may be rejected. A token from pi's own device flow, done once on the host, may be needed.
- Whether pi accepts a placeholder value for `COPILOT_GITHUB_TOKEN` and still picks the right API host.
- Whether the same token works as a Bearer token for the GitHub MCP server.

Fallbacks: allow a real token inside the sandbox, or use an agent with built-in sbx Copilot support (`copilot` or `opencode`).

### Helper script safety

The helper lives outside the workspace on purpose. sbx docs say secret helpers must stay outside writable sandbox mounts. The first `-Check` run was not approved, so the script has not been run yet.

## Best practices

- Keep credentials on the host (`sbx secret set`). Never put keys inside the sandbox.
- Use `--clone` so the agent works on a private clone, not the working tree. In direct mode check `.git/hooks` and build scripts before running changed code.
- Run `sbx policy ls` and remove broad wildcard domains that are not needed.
- Pin kits by digest. Kit installs run as root, so restrict kit sources.
- Share skills read-only. A read-write store puts all sandboxes in one trust boundary.
- Credential and kit changes only apply to new sandboxes.

## Next steps

1. Run the helper with `-Check` and read the result.
2. Add allow rules for `api.exa.ai` and `mcp.context7.com`.
3. Spike Copilot auth in a throwaway `sbx/pi` sandbox.
4. Write the `pi-lars` mixin (Dockerfile, network policy, credentials, `settings.json`, MCP adapter config).
5. Build the kit set and test with `sbx run`.
6. Send the Context7 and GitHub MCP details for verification.

## Links

- https://docs.docker.com/ai/sandboxes/
- https://hub.docker.com/r/sbx/pi
- https://github.com/docker/sandbox-kit-spec
- https://pi.dev/docs/latest/providers
