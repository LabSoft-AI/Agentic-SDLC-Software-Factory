# Pi Harness

Pi ([pi.dev](https://pi.dev)) is the second supported harness, next to Kiro CLI
and OpenCode. Pi itself is minimal: it ships no named agents. A small **preset
extension** adds them, so a Pi agent config is a JSON preset plus the prompt it
loads.

The World monorepo runs its whole agent fleet this way. Its `.pi/` folder keeps
agent definitions in `.pi/presets.json`, reuses the Kiro prompts and resources
in place, and adds a permission gate in an `ls-agents` extension. The course
uses the same preset model. The public Pi preset example supports a model, a
tool list, and an inline prompt; the World extension adds instruction files,
resources, context commands, MCP servers, subagents, and an allow/ask/deny gate.

## Install

Pi runs on Node 24. Keep the course's Node 20 install as the default and switch
with `nvm`:

```bash
nvm install 24
nvm use 24
npm install -g --ignore-scripts @earendil-works/pi-coding-agent@1.0.4
pi --version   # 1.0.4
```

`--ignore-scripts` is intentional: Pi does not need post-install scripts. The
World setup targets Pi 1.0.4; do not run `pi update` without rerunning the
harness tests.

Pi can use any provider. World uses Amazon Bedrock:

```bash
aws sso login --profile <your-sso-profile>
export AWS_PROFILE=<your-sso-profile>
export AWS_REGION=eu-central-1
```

Without Bedrock, run `/login` inside Pi and pick the provider you have. Use the
matching `provider` and `model` in the preset.

## First run

From your course repository:

```bash
nvm use 24
pi
```

The first time, Pi asks whether to trust the project. Run `/trust` once so Pi
loads `.pi/` (settings, extensions, skills, and prompts). In print, JSON, and
RPC modes the trust prompt does not appear, so run an interactive `/trust`
first or the project resources are skipped.

## Add preset support

The smallest setup is Pi's public preset extension. It ships with the npm
package:

```bash
mkdir -p .pi/extensions
cp "$(npm root -g)/@earendil-works/pi-coding-agent/examples/extensions/preset.ts" \
  .pi/extensions/preset.ts
```

If the installed package does not include the example, download it from the
pinned Pi release:

```bash
curl -fsSL \
  https://raw.githubusercontent.com/earendil-works/pi/v1.0.4/packages/coding-agent/examples/extensions/preset.ts \
  -o .pi/extensions/preset.ts
```

If you have access to the World monorepo, copy its full extension instead:

```bash
cp -R /path/to/world/.pi/extensions/ls-agents .pi/extensions/
```

The World extension understands every field in the tables below. The public
example understands only `provider`, `model`, `thinkingLevel`, `tools`, and
`instructions`.

Optional: set the default provider and model in `.pi/settings.json`:

```json
{
  "defaultProvider": "amazon-bedrock",
  "defaultModel": "global.anthropic.claude-sonnet-4-6"
}
```

## Define an agent

Presets live in `.pi/presets.json` at the repository root. Each key is a preset
name. The public example uses an inline prompt:

```json
{
  "git-agent": {
    "provider": "amazon-bedrock",
    "model": "global.anthropic.claude-sonnet-4-6",
    "tools": ["read", "grep", "find", "ls", "bash", "edit", "write"],
    "instructions": "You are a Git workflow automation agent. Use `gh` for all GitHub operations and follow the issue, branch, commit, and PR conventions from Module 1."
  }
}
```

Put the full prompt from the module's Kiro starter in `instructions` (the
`prompt` field of `modules/module-1-git/starter/git-agent.json`). JSON strings
use `\n` for line breaks.

With the World extension, keep the prompt in a Markdown file and add resources
and permissions:

```json
{
  "git-agent": {
    "provider": "amazon-bedrock",
    "model": "global.anthropic.claude-sonnet-4-6",
    "tools": ["read", "grep", "find", "ls", "bash", "edit", "write"],
    "instructionsFiles": [".pi/agents/git-agent.md"],
    "policy": {
      "allowTools": ["read", "grep", "find", "ls", "bash"],
      "bash": {
        "allow": [
          "^(?!.*--output)git (status|diff|log|show)\\b",
          "^gh (issue|pr) (list|view)\\b"
        ]
      }
    }
  }
}
```

Start the agent:

```bash
pi --preset git-agent        # start in the preset
# or, inside a session:
/preset git-agent            # switch preset
/preset                      # pick from a list
/session                     # tokens, context, and session stats
```

The World extension also adds `/agent swap <name>` (the Kiro habit), `/usage`,
and `/context`.

The preset's model is the default. A `--model <id>` on the command line or
`/model` inside the session wins over the preset.

## What maps from Kiro

| Kiro | Pi public preset | Pi World `ls-agents` |
|------|------------------|----------------------|
| `.kiro/agents/<name>.json` | `.pi/presets.json` entry | Same |
| `prompt` (inline) | `instructions` string | `instructionsFiles` |
| `prompt: "file://..."` | Inline the file into `instructions` | `instructionsFiles` |
| `tools` | `tools` | `tools` |
| `resources` | No auto-load; name the files in `instructions` | `resourceGlobs` |
| `allowedTools` | No gate; `tools` limits what the agent can use | `policy.allowTools` |
| `hooks.postToolUse` | No per-agent hook; put the command in the prompt or a pre-commit hook | Not ported; `contextCommands` run Kiro `userPromptSubmit` hooks before each prompt |
| `mcpServers` | Project `.pi/mcp.json` or a Pi extension | Per-preset `mcpServers` |

The Pi tool names for the course starters are:

| Kiro tool | Pi tool |
|-----------|---------|
| `fs_read` | `read`, `grep`, `find`, `ls` |
| `fs_write` | `edit`, `write` |
| `shell` | `bash` |

## Module starters

The course ships Kiro JSON and OpenCode Markdown starters. For Pi, use the same
prompt and rules from the Kiro starter and turn them into a preset:

1. Take the `prompt` value from `starter/<name>-agent.json`.
2. Add a `<name>-agent` entry to `.pi/presets.json` with the mapped tools.
3. Use `instructions` with the public extension, or save the prompt as
   `.pi/agents/<name>-agent.md` and use `instructionsFiles` with the World
   extension.

Module 5 needs two adjustments:

- The prompt lives in `tdd-bdd-prompt.md`. Inline its contents into
  `instructions`, or keep it as an instruction file with the World extension.
- `resources` and the `postToolUse` hook have no direct equivalent in the
  public extension. Tell the agent to read `docs/user-stories/**/*.md`, and put
  the exact pytest command in the prompt or a git pre-commit hook.

This is the same trade-off as the [OpenCode harness](opencode.md): the prompt,
tools, and task stay the same; only the file format and the harness features
change.

## Permissions and safety

The public preset extension only sets the model, tools, and prompt. Pi does not
ask for approval before every tool call and does not sandbox tool execution.
Run it with the permissions of a dedicated user or inside a container, and
review the diffs it produces. See [Pi security](https://pi.dev/docs/latest/security)
for the trust model and isolation options.

The World `ls-agents` extension adds a gate in front of every tool call:

- **allow** - runs without prompting (read-only tools and the preset's allowed
  `git`/`gh` commands).
- **ask** - Pi shows `Allow? [Yes/No]`. Declining, or running without an
  interactive UI, blocks the call.
- **deny** - blocked immediately. Destructive git is always denied: force push,
  `reset --hard`, `--no-verify`, and `SKIP=`.

The gate also always asks for commands whose first word is `rm`, `sudo`,
`chmod`, `chown`, `dd`, `mkfs`, `kill`, `pkill`, `shutdown`, or `reboot`;
destructive `git clean`, `restore`, `rm`, `stash drop`/`clear`,
`branch -D`, `checkout --`; `find -delete`/`-exec`; and any chained command
(`&&`, `;`, `|`, `$(...)`, `>`). Writes under `.pi/` and `.git/` always ask.

The always-ask list is a speed bump, not a sandbox. A wrapper such as
`bash -c`, `env`, or `python3 -c` can still run something indirectly when the
agent is allowed to run arbitrary commands.

## What the World harness adds

- **Context commands** - a preset's `contextCommands` run before every prompt
  and their stdout is added to the context (5 s timeout).
- **Reference files** - resolved `resourceGlobs` are embedded in the system
  prompt when they total 200 KB or less; larger sets are listed and the agent
  reads them on demand.
- **Subagents** - a preset can start another preset in an isolated context. The
  child asks for every approval in the parent session and cannot start its own
  subagent.
- **/commit** - delegates to the `ls-git` subagent, commits the working tree,
  takes the issue number from the branch context, and never pushes.
- **Skills** - Kiro skills load in place and appear as `/skill:<name>`.
- **MCP per preset** - a preset's MCP servers are registered when you enter it
  and unregistered when you leave.
- **Web fetch** - the `fetch` MCP server is added for agents that used
  `web_fetch` in Kiro.

## Differences to keep in mind

- Pi needs Node 24, while the rest of the course runs on Node 20+.
- Named agents require a preset extension; plain Pi has no `--preset` flag.
- Pi has no language-server support, so Kiro's LSP is not available.
- Pi has no `web_search`; add a fetch MCP server if you need to read web pages,
  or use the `aws` CLI through `bash`.
- `postToolUse` hooks are not ported to the World harness. Use a prompt rule, a
  git pre-commit hook, or a Pi extension for a deterministic sensor.
- The public preset extension has no permission gate and no resource auto-load.
  Use the World extension if you need those features.
- The World extension protects `.pi/` and `.git/`: writes there always ask.

## Troubleshooting

- **`pi` fails with `enableCompileCache` or another `node:module` error.** You
  are on Node 20. Run `nvm use 24` and try again.
- **AWS errors or an expired token.** Re-run
  `aws sso login --profile <your-sso-profile>` and confirm `AWS_PROFILE` and
  `AWS_REGION` are exported in the current shell.
- **Project resources are skipped.** Run an interactive `/trust` once before
  using print, JSON, or RPC mode.
- **`Unknown preset`.** Check that `.pi/presets.json` exists at the repository
  root, the name matches a key, and the preset extension loaded.
- **A preset fails to apply.** With the World extension, an MCP server that
  cannot register aborts the switch and leaves the previous preset active.

## References

- [Pi documentation](https://pi.dev/docs/latest)
- [Pi extensions](https://pi.dev/docs/latest/extensions)
- [Pi preset example](https://github.com/earendil-works/pi/blob/v1.0.4/packages/coding-agent/examples/extensions/preset.ts)
- [Pi security](https://pi.dev/docs/latest/security)
- [Kiro CLI custom agents](https://kiro.dev/docs/cli/custom-agents/configuration-reference/)
