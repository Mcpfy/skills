# mcpfy Skills

Official skills collection for the [mcpfy-sdk](https://github.com/mcpfyy/mcpfy) framework. Skills are reusable capabilities that enhance Claude's ability to build and work with Model Context Protocol (MCP) servers.

## What are Skills?

Skills are folders containing instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. Each skill teaches Claude how to complete specific tasks in a repeatable way.

For more information about Agent Skills, visit:
- [agentskills.io](https://agentskills.io) - Official specification
- [skills.sh](https://skills.sh) - Skills platform and leaderboard
- [Anthropic's Skills Guide](https://support.claude.com/en/articles/12512198-creating-custom-skills)

## Available Skills

### mcpfy-server-builder

Scaffold, build, extend, or debug an MCP server using the mcpfy-sdk TypeScript package. This skill provides:
- Quick start with `npx create-mcpfy-app`
- Defining tools, resources, prompts, and widgets
- Client setup and authentication (OAuth/JWT)
- Building MCP App widgets with `mcpfy-sdk/widget`
- Troubleshooting common issues

**Use when**: Creating MCP servers, defining tools/resources/prompts/widgets, connecting an mcpfy client, setting up auth, or working with the mcpfy-sdk framework.

**Example prompts:**
- "Scaffold a new MCP server with mcpfy that has a tool for rolling dice"
- "Add OAuth authentication to my mcpfy MCP server"
- "Build a weather widget for my mcpfy server using `mcpfy-sdk/widget`"
- "My mcpfy widget's `fetch` call is being blocked — help me configure its CSP"
- "Connect an MCP client to my mcpfy server over HTTP and call one of its tools"

### mcpfy-webmcp-builder

Make a website callable by in-browser AI agents with WebMCP. The skill surveys your project, proposes which site actions to expose and waits for your approval, then builds them as annotated forms, custom `registerTool` tools, or a bridge from an existing MCP server (for example one built with `mcpfy-sdk`). This skill provides:
- A proposal step that requires your sign-off before any code is written
- Form annotations that turn existing `<form>`s into agent tools
- Custom tools via `document.modelContext.registerTool`
- A bridge that republishes an existing MCP server inside the page
- Runtime guards, polyfill guidance, and recipes for React, Next.js, Vue and Angular
- Browser-based testing with Chrome DevTools MCP and a safety review checklist

**Use when**: Making a site WebMCP-compatible, annotating forms as agent tools, registering page tools with `registerTool`, or putting an existing MCP server on a website with `webmcp-proxy`.

**Example prompts:**
- "Let browser agents book appointments on our site — what should we expose through WebMCP?"
- "Turn the contact and quote forms in this repo into agent-callable tools"
- "Our mcpfy MCP server is live; make its tools available to agents visiting our website"
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

Skills follow the [Agent Skills specification](https://agentskills.io). When contributing:

1. Fork this repository
2. Create a new skill folder under `skills/`
3. Include a `SKILL.md` file with proper frontmatter
4. Add a `LICENSE.txt` if using a different license
5. Test thoroughly with Claude
6. Submit a pull request

## Learn More

- **mcpfy-sdk GitHub**: [github.com/mcpfyy/mcpfy](https://github.com/mcpfyy/mcpfy)
- **MCP Protocol**: [modelcontextprotocol.io](https://modelcontextprotocol.io)

## Support

- **Issues**: [github.com/mcpfy/skills/issues](https://github.com/mcpfy/skills/issues)
- **Email**: [team@mcpfy.com](mailto:team@mcpfy.com)

These skills run entirely locally against your own project — they don't call any mcpfy-owned service, collect data, or require an account.

## License

This repository is licensed under the MIT License. See [LICENSE](LICENSE) for details.

Individual skills may have their own licenses specified in their respective `LICENSE.txt` files.
