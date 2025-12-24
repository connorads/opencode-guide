# OpenCode

[OpenCode](https://opencode.ai) is an open-source AI coding agent for the terminal. Think Claude Code, Codex, Gemini CLI, or GitHub Copilot CLI - but open source, so you only need to learn and customise one tool.

It works with your existing subscriptions - Claude Pro/Max, GitHub Copilot, ChatGPT Plus, or Gemini - instead of paying for yet another API. Built-in LSP support means it understands your code like an IDE - with real-time type checking and diagnostics.

## Install

```sh
brew install opencode
```

Or with mise:

```sh
mise use -g github:sst/opencode
```

## Setup

### Connect a provider

```sh
opencode
```

Then run `/connect` and select your provider. OpenCode supports 75+ providers including:

| Provider | Auth Method |
|----------|-------------|
| **Anthropic** | Claude Pro/Max subscription (OAuth) or API key |
| **GitHub Copilot** | Existing Copilot subscription |
| **OpenAI** | API key from platform.openai.com |
| **ChatGPT Plus/Pro** | OAuth via [opencode-openai-codex-auth](https://github.com/numman-ali/opencode-openai-codex-auth) plugin |
| **Gemini** | Google account via [opencode-gemini-auth](https://github.com/jenslys/opencode-gemini-auth) plugin |
| **OpenCode Zen** | [Curated models](https://opencode.ai/docs/zen/) from the OpenCode team (includes free models) |

For a full list of providers, see the [providers docs](https://opencode.ai/docs/providers/).

For ChatGPT or Gemini, add the respective plugins to your config (`~/.config/opencode/opencode.json`).

Then run `opencode auth login` and follow the prompts.

#### Gemini

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["opencode-gemini-auth@1.2.0"]
}
```

#### ChatGPT Plus/Pro (Codex OAuth plugin)

This setup is a bit more involved (you must adapt the plugin’s full config file). Easiest path: ask OpenCode to do it for you.

```sh
opencode run "Webfetch https://github.com/numman-ali/opencode-openai-codex-auth/ and follow the 'Agent Instructions (Install / Update)' steps, you can use Webfetch to get the contents instead of cloning the repo."
```

## LSP Support

OpenCode has built-in LSP support - automatically loading the right Language Server Protocol servers for your project. The LLM gets real-time type checking, syntax validation, and diagnostics - so it writes better code with fewer errors.

**Supported languages:** TypeScript, JavaScript, Python, ESLint, Go, Rust, C/C++, Ruby, Elixir, PHP, Java, and more.

See the full list of [supported LSP servers](https://opencode.ai/docs/lsp/) and configuration options.

## Usage

### Modes

OpenCode has two modes, shown in the bottom-right corner:

| Mode | Description |
|------|-------------|
| **Build** | Can read and modify files (default) |
| **Plan** | Read-only - suggests changes without executing |

Press **Tab** to switch between modes.

### File references

Use `@` to fuzzy-search and reference files:

```
How does auth work in @src/api/auth.ts?
```

### Run shell commands

Prefix with `!` to run a command and add output to the conversation:

```
!npm test
```

### Useful commands

| Command | Keybind | Description |
|---------|---------|-------------|
| `/models` | `ctrl+x m` | Switch models |
| `/new` | `ctrl+x n` | Start new session |
| `/mcp` | - | Toggle MCP servers |
| `/undo` | `ctrl+x u` | Undo last message (reverts file changes too) |
| `/redo` | `ctrl+x r` | Redo undone message |
| `/sessions` | `ctrl+x l` | List/switch sessions |
| `/share` | `ctrl+x s` | Share conversation |
| `/review` | - | Built-in code review command |
| `/help` | `ctrl+x h` | Show all commands |

You can also add your own [custom commands](https://opencode.ai/docs/commands/).

### Initialise for a project

```
/init
```

Creates an `AGENTS.md` file that helps OpenCode understand your codebase. Commit this to your repo.

### Agents

OpenCode supports subagents for specialised tasks - similar to Claude Code. You can also define your own custom agents. See the [agents docs](https://opencode.ai/docs/agents/).

### Skills

OpenCode supports [agent skills](https://agentskills.io/): reusable `SKILL.md` definitions you can share in a repo or keep in your global config. For setup details, see the [skills docs](https://opencode.ai/docs/skills/).

## Tips

- **Drag and drop images** into the terminal to add them to your prompt
- **Use Plan mode first** for complex features - review the plan, then switch to Build mode
- **Be specific** - give context and examples like you would to a junior developer
- Press `ctrl+x e` to open your `$EDITOR` for longer prompts

## Coming from Claude Code?

### Plan mode works differently

In Claude Code, Plan mode suggests changes and asks "Do you want to execute?" - you press Yes and it builds.

In OpenCode, Plan mode is strictly read-only. To execute a plan:

1. Press **Tab** to switch to Build mode
2. Send a message like "Go ahead" or "Do it"

### Skills are discovered differently

If you're coming from Claude, OpenCode can load repo-local Claude-compatible skills from `.claude/skills/<name>/SKILL.md`, but it does not load Claude's global skills folder (`~/.claude/skills`). Copy any global Claude skills into your repo (`.claude/skills/...`) or into OpenCode's global skills folder (`~/.opencode/skill/<name>/SKILL.md`). See the [skills docs](https://opencode.ai/docs/skills/) for supported locations.

### Permissions are more permissive by default

OpenCode allows most operations without asking. To replicate Claude Code's confirmation prompts, add to your config ([permissions docs](https://opencode.ai/docs/permissions/)):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "bash": "ask",
    "edit": "ask"
  }
}
```

You can also use wildcards for granular control:

```json
{
  "permission": {
    "bash": {
      "*": "ask",
      "git status": "allow",
      "git diff *": "allow",
      "npm run build *": "allow",
      "npm run test *": "allow"
    }
  }
}
```

### MCP servers are configured in the config file

Add servers directly to `opencode.json` ([MCP docs](https://opencode.ai/docs/mcp-servers/)).

- `enabled` controls startup of the MCP server.
- `oauth` configures OAuth - use `opencode mcp auth <name>` to authenticate.
- `tools` (with globs) controls availability to agents.

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "chrome-devtools": {
      "type": "local",
      "command": ["npx", "-y", "chrome-devtools-mcp@latest"],
      "enabled": false,
    },
    "sentry": {
      "type": "remote",
      "url": "https://mcp.sentry.dev/mcp",
      "enabled": false,
      "oauth": {}
    }
  },
  "tools": {
    // Disable globally
    "chrome-devtools*": false,
    "sentry*": false
  },
  "agent": {
    "my-debug-agent": {
      "tools": {
        // Enable for this agent
        "chrome-devtools*": true,
        "sentry*": true
      }
    }
  }
}
```

### Hooks work through plugins

Claude Code has a `hooks` config for running scripts at lifecycle events. OpenCode uses plugins instead - more powerful but requires JavaScript:

Create `.opencode/plugin/my-hooks.js` (project) or `~/.config/opencode/plugin/` (global):

```javascript
export const AutoFormat = async ({ $ }) => {
  return {
    "tool.execute.after": async (input) => {
      if (input.tool === "edit") {
        await $`prettier --write ${input.input.filePath}`
      }
    }
  }
}
```

Available hooks include `tool.execute.before`, `tool.execute.after`, and events like `session.idle`, `file.edited`, etc. See the [plugins docs](https://opencode.ai/docs/plugins/).

### No multi-select questions

Claude Code sometimes presents multiple questions with checkboxes. OpenCode uses plain text responses - just answer in your message.

### Config file locations

- **Global**: `~/.config/opencode/opencode.json`
- **Project**: `.opencode.json` in project root
