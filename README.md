# mcpfy Skills

[![License: MIT](https://img.shields.io/github/license/mcpfy/skills)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/mcpfy/skills?style=social)](https://github.com/mcpfy/skills/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/mcpfy/skills)](https://github.com/mcpfy/skills/commits/main)
[![Skills](https://img.shields.io/badge/skills-2-blue)](#available-skills)

Official skills collection for the [mcpfy-sdk](https://github.com/mcpfyy/mcpfy) framework. Skills are reusable capabilities that enhance Claude's ability to build and work with Model Context Protocol (MCP) servers.

## Table of Contents

- [What are Skills?](#what-are-skills)
- [Available Skills](#available-skills)
  - [mcpfy-server-builder](#mcpfy-server-builder)
  - [mcpfy-webmcp-builder](#mcpfy-webmcp-builder)
- [Installation](#installation)
- [Repository Structure](#repository-structure)
- [Skills Platform Integration](#skills-platform-integration)
- [Contributing](#contributing)
- [Learn More](#learn-more)
- [Support](#support)
- [License](#license)

## What are Skills?

Skills are folders containing instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. Each skill teaches Claude how to complete specific tasks in a repeatable way.

For more information about Agent Skills, visit:
- [agentskills.io](https://agentskills.io) - Official specification
- [skills.sh](https://skills.sh) - Skills platform and leaderboard
- [Anthropic's Skills Guide](https://support.claude.com/en/articles/12512198-creating-custom-skills)

## Available Skills

### mcpfy-server-builder

Scaffold, build, extend, or debug a **standalone MCP server** using the mcpfy-sdk TypeScript package. This skill covers backend server work — not browser-side WebMCP on a website. It provides:
- Quick start with `npx create-mcpfy-app`
- Defining tools, resources, prompts, and MCP App widgets (UI embedded in chat clients via `mcpfy-sdk/widget`)
- MCP client setup and server-side authentication (OAuth/JWT)
- stdio and HTTP transport, plus troubleshooting common mcpfy-sdk mistakes

**Use when**: You need a remote MCP server — scaffolding with `create-mcpfy-app`, adding tools/resources/prompts/MCP App widgets, connecting an MCP client over stdio or HTTP, or configuring OAuth/JWT on the server.

**Not for**: Letting in-browser agents operate your website. Use `mcpfy-webmcp-builder` for WebMCP (`document.modelContext`) on a front end.

**Example prompts:**
- "Scaffold a new MCP server with mcpfy that has a tool for rolling dice"
- "Add OAuth authentication to my mcpfy MCP server"
- "Build an MCP App weather widget for my mcpfy server using `mcpfy-sdk/widget`"
- "My mcpfy widget's `fetch` call is being blocked — help me configure its CSP"
- "Connect an MCP client to my mcpfy server over HTTP and call one of its tools"

### mcpfy-webmcp-builder

Let **in-browser AI agents** operate your website. [WebMCP](https://webmachinelearning.github.io/webmcp/) is a browser API (`document.modelContext`) through which a page publishes tools that an agent in the user's browser can call — using the visitor's signed-in session, without scraping the DOM. This skill guides Claude through adding WebMCP to an existing front-end codebase.

How it works:
1. **Survey** — Claude reads your front end, finds the forms and multi-screen journeys, and checks whether you already run an MCP server (for example one built with `mcpfy-sdk`).
2. **Propose** — it writes up a table of the agent-facing actions it suggests, how each would be delivered, and which ones are risky, then waits for your approval. No code is written before that.
3. **Build** — each approved action ships through one of three routes:
   - *Form annotations*: attributes on your existing `<form>`s
   - *Custom tools*: `registerTool` for a journey that deserves a single purpose-built call
   - *Bridge*: your existing MCP server's tools republished inside the page
4. **Test** — the tools are listed and executed in a real browser through Chrome DevTools MCP, and each result is checked against what you approved.
5. **Review** — a safety pass covers honest annotations, confirmation for irreversible actions, server-side authorization and untrusted output.

Also included: runtime guards for browsers without WebMCP, polyfill guidance, and recipes for vanilla JS, React, Next.js, Vue and Angular.

**Use when**: Adding WebMCP to a website — annotated forms, `registerTool` page tools, or bridging an existing MCP server's tools into the browser tab so agents can act on the site.

**Not for**: Building or deploying a remote MCP server, MCP App chat widgets, or server-side OAuth setup. Use `mcpfy-server-builder` for those.

**Good to know**: WebMCP support is still experimental (origin trials and flags in Chromium-based browsers). The skill has Claude check current support before making promises, and every approach leaves your normal UI working in browsers without it.

**Example prompts:**
- "Let browser agents book appointments on our site — what should we expose through WebMCP?"
- "Turn the contact and quote forms in this repo into agent-callable tools"
- "Our mcpfy MCP server is live; bridge its tools so agents visiting our website can call them"
- "Replace our four-step signup with a single WebMCP tool and show me how to test it"

## Installation

### Claude Code

Register this repository as a marketplace:

```bash
/plugin marketplace add mcpfy/skills
```

Then install skills:

```bash
# Install all skills
/plugin install all-skills@mcpfy

# Or install individual skills
/plugin install mcpfy-server-builder@mcpfy
/plugin install mcpfy-webmcp-builder@mcpfy
```

Or browse and install via the UI:
1. Run `/plugin marketplace add mcpfy/skills`
2. Select `Browse and install plugins`
3. Select `mcpfy`
4. Choose a skill to install
5. Select `Install now`

### Via skills.sh

```bash
npx skills add mcpfy/skills
```

### Claude.ai

1. Go to your Claude.ai project settings
2. Navigate to Skills section
3. Upload the `skills/` folder or individual skill folders
4. Skills will be available in your conversations

### Cursor / Windsurf

1. Clone this repository
2. Reference skills in your `.cursorrules` or project settings
3. Skills will be available when working in the IDE

## Repository Structure

```
.
├── .claude-plugin/       # Claude Code marketplace configuration
├── skills/               # Individual skills
│   ├── mcpfy-server-builder/
│   │   ├── SKILL.md      # Skill definition
│   │   ├── LICENSE.txt   # Skill license
│   │   ├── assets/       # Bundled template files
│   │   └── references/   # Reference docs loaded on demand
│   └── mcpfy-webmcp-builder/
│       ├── SKILL.md      # Skill definition
│       ├── LICENSE.txt   # Skill license
│       └── references/   # Approach guide, forms, custom tools, runtime, bridge, testing, safety
├── spec/                 # Agent Skills specification reference
│   └── README.md
├── LICENSE               # Repository license (MIT)
└── README.md             # This file
```

## Skills Platform Integration

This repository is compatible with:
- **Claude Code**: Via `.claude-plugin/marketplace.json`
- **skills.sh**: Via `npx skills add mcpfy/skills`
- **Claude.ai**: Manual upload of skill folders
- **Cursor/Windsurf**: Local skill references

## Contributing

Skills follow the [Agent Skills specification](https://agentskills.io). For the `SKILL.md` format and frontmatter rules used in this repository, see [spec/README.md](spec/README.md).

To add or improve a skill:

1. Fork the repository and create a branch.
2. Add a folder under `skills/<skill-name>/` with at minimum a `SKILL.md`.
3. Use [skills/mcpfy-server-builder/SKILL.md](skills/mcpfy-server-builder/SKILL.md) as the reference layout — it shows the expected YAML frontmatter (`name`, `description`), staged workflow, on-demand `references/` docs, bundled `assets/`, and guardrails.
4. Add a `LICENSE.txt` in the skill folder only if the skill uses a different license than the repo (MIT).
5. Register new skills in [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) so they appear in the Claude Code marketplace.
6. Test the skill end-to-end with your target agent before opening a pull request.
7. Open a PR describing what the skill does and which example prompts you used to validate it.

Doc fixes and reference improvements to existing skills do not require marketplace changes.

## Learn More

- **Agent Skills spec (this repo)**: [spec/README.md](spec/README.md) — `SKILL.md` frontmatter, required fields, and layout conventions used here
- **Agent Skills (official)**: [agentskills.io](https://agentskills.io)
- **Skills platform**: [skills.sh](https://skills.sh) — install with `npx skills add mcpfy/skills`
- **mcpfy-sdk**: [github.com/mcpfyy/mcpfy](https://github.com/mcpfyy/mcpfy)
- **MCP Protocol**: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **WebMCP (browser API)**: [webmachinelearning.github.io/webmcp](https://webmachinelearning.github.io/webmcp/)

## Support

- **Skills issues**: [github.com/mcpfy/skills/issues](https://github.com/mcpfy/skills/issues) — bugs or gaps in these skills
- **mcpfy-sdk issues**: [github.com/mcpfyy/mcpfy/issues](https://github.com/mcpfyy/mcpfy/issues) — SDK bugs or feature requests
- **Email**: [team@mcpfy.com](mailto:team@mcpfy.com)

These skills run entirely locally against your own project — they don't call any mcpfy-owned service, collect data, or require an account.

## License

This repository is licensed under the MIT License. See [LICENSE](LICENSE) for details.

Individual skills may have their own licenses specified in their respective `LICENSE.txt` files.
