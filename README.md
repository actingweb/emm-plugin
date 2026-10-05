# Emm AI — Claude Plugin

A Claude plugin that connects to **Emm AI**, your mission control for AI agents. It works in Claude Code, Claude Cowork, the Claude apps and claude.ai. Emm AI gives every AI session you run shared, persistent context:

- **Memory** — preferences, decisions, family/health/travel context, and anything else you ask Emm to remember.
- **Standing instructions** — documents in your account that you write and edit (how you like to work, project rules, do/don't lists); Claude reads them when a task calls for them, and its system prompt and your messages come first.
- **Output wiki** — structured artifacts Emm generates with you (recipes, notes, summaries) that you can read back any time.

**MCP server:** `https://ai.actingweb.io/mcp`

The plugin sends data only to this MCP server, and only when Claude calls one of its tools.

---

## Example Prompts

- "Remember that I'm vegetarian and allergic to walnuts." Claude saves it to your Emm memory.
- "What did we decide about the Q3 roadmap? Check my Emm memory." Claude searches your memories before answering.
- "Draft a reply to Ana in my usual style." Claude reads your standing instructions on tone and language first.
- "Run my morning tasks." (`/emm:run`) Claude runs your recurring agent cycle and files the results in your output wiki.
- "Save this plan to my Emm wiki." Claude stores the document so any later session can read it back.

---

## What This Plugin Does

Installing this plugin gives Claude:

- **A skill bundle** — teaches Claude to use your Emm AI account across every pillar (memory, standing instructions, output wiki, recurring agent runs).
- **21 slash commands** — explicit controls for each pillar: memory (`/emm:memory-*`), the wiki (`/emm:output-*`, `/emm:dashboard`), standing instructions (`/emm:instructions-*`), the recurring cycle (`/emm:run`, `/emm:task`), connections (`/emm:connections`), and the personalized help guide (`/emm:help`).
- **MCP server auto-configuration** — `.mcp.json` connects Claude to `https://ai.actingweb.io/mcp` automatically, no manual setup needed.

---

## Installation

The plugin ships as `emm-plugin.zip`. The zip root is `emm/`, which becomes the Claude command namespace.

### Claude Cowork (desktop / web)

1. Open the **Customize** panel (gear icon → Customize, or `⌘ ,` then "Customize").
2. In the left sidebar under **Personal plugins**, click the **`+`** button.
3. In the menu that appears, choose **Create plugin** and select the downloaded `emm-plugin.zip` file from disk. (Cowork uploads the bundle and registers it as a personal plugin.)
4. Toggle the plugin on. The skill and slash commands become available immediately.

In Cowork, slash commands are matched by fuzzy/word search across plugin descriptions — type `/run`, `/dashboard`, `/memory`, `/instructions`, etc. and pick from the suggestions. The `emm:` namespace is shown as the plugin attribution but you don't need to type it.

### Claude Code (CLI)

```bash
# Unzip, then install
unzip emm-plugin.zip
claude plugin install ./emm
```

Slash commands are namespaced as `/emm:run`, `/emm:dashboard`, etc.

### Manual install (any host)

After unzipping `emm-plugin.zip`, copy individual components:

```bash
# Skill bundle (Claude reads this for capability guidance)
cp -r emm/skills/working-with-emm ~/.claude/skills/

# Slash commands — when copied outside a plugin folder, the namespace
# prefix is dropped: /emm:run becomes /run.
cp -r emm/commands/* ~/.claude/commands/

# MCP server config
cp emm/.mcp.json .mcp.json
```

---

## Authentication

Emm AI uses standard OAuth 2.1 with PKCE. The first time Claude calls the server, it gets a sign-in challenge and starts the flow; no API keys are involved:

1. Claude registers itself with the server (dynamic client registration) and opens Emm's authorization page in your browser.
2. Sign in with Google, GitHub or Apple, or create an account at [ai.actingweb.io](https://ai.actingweb.io).
3. Choose the access level on that page:
   - **Read/Write** (the default) lets Claude save and change your memories, wiki and settings.
   - **Read-Only** lets it search and read them only.
4. Claude stores the token itself; you won't be prompted again until you revoke it.

You can see and revoke every connected client in the app under My Connections.

Standing instructions have an extra gate: an AI can change them only during an approval window that you open in the app.

---

## Command Reference

Commands are prefixed with `/emm:` in Claude Code (the namespace comes from the `emm/` zip root directory):

### Memory

| Command | Description | Arguments |
|---|---|---|
| `/emm:memory-search [query]` | Search memories by keyword or meaning | Search query |
| `/emm:memory-save [content]` | Save specific content to memory | Content to save |
| `/emm:memory-types` | List memory categories with item counts | None |
| `/emm:memory-delete-type [category]` | Delete an empty custom memory category | Category short name (e.g. `recipes`) |
| `/emm:memory-new [days]` | Show recently added memories (default: 7 days) | Number of days |
| `/emm:memory-context [topic]` | Load memories relevant to a topic | Topic |
| `/emm:memory-extract` | Suggest things worth remembering from the current conversation, and save the ones you confirm | None |
| `/emm:memory-news [source]` | Generate a personalized news summary | Source (optional) |
| `/emm:memory-move [ids → category]` | Move (recategorise) one or more memories; references are rewritten automatically | IDs + target category |
| `/emm:memory-delete [ids]` | Delete one or more memories by ID (use `preview` first) | One or more `type:id` |

### Output wiki

| Command | Description | Arguments |
|---|---|---|
| `/emm:output-search [query]` | Search the output wiki (drafts, research, plans, logs) | Search query |
| `/emm:output-new [hint]` | Capture a new output — Claude walks you through `output_create` | Optional hint |
| `/emm:output-delete [ids]` | Delete one or more wiki items by ID (supports batch) | One or more `category:id` |
| `/emm:output-move [ids → destination]` | Move (re-folder, rename or recategorise) one or more wiki documents; the id stays the same | IDs + folder, slug or category |
| `/emm:dashboard` | Load the current actions dashboard | None |

### Standing instructions

| Command | Description | Arguments |
|---|---|---|
| `/emm:instructions-show [name]` | List installed instructions (no arg) or load one by name | Optional short name |
| `/emm:instructions-edit [name]` | Edit an instruction — guides the approval + save flow | Short name |

### Cycle & queue

| Command | Description | Arguments |
|---|---|---|
| `/emm:run` | Run the recurring agent cycle (`agent_run`) | None |
| `/emm:task [topic]` | Drain the one-off task queue (`work_on_task`) | Optional topic |

### Connections & help

| Command | Description | Arguments |
|---|---|---|
| `/emm:connections` | List trusted connections, shared memory, and remote actions | None |
| `/emm:help` | Personalized account guide (`how_to_use`) | None |

Claude can also read your standing instructions and the output wiki through the MCP server when a task calls for them. Use the slash commands above when you want to explicitly trigger a pillar or fast-track a workflow. Manage everything from your [Emm AI dashboard](https://ai.actingweb.io).

**About `/emm:memory-extract`:** it runs only when you ask for it. Claude reads the current conversation, lists what it would save and where, and saves only the items you confirm. It skips transient chatter, secrets, and anything you asked it not to keep.

---

## Privacy

Your data is stored by Emm AI (`https://ai.actingweb.io`) in the EU, on AWS.

- **Other people** see it only if you connect with them in the app and grant them access to specific memory categories. Nothing is public.
- **Your connected AI agents** see your memories. A custom memory category is shared with all of them by default; you can make one "Creator only" in Settings → Memory & Sharing.
- **AI processing** in Emm runs on AWS Bedrock, which does not train its models on your data.

[Privacy Policy](https://ai.actingweb.io/privacy) · [Terms of Service](https://ai.actingweb.io/terms)

---

## Support

- [Dashboard](https://ai.actingweb.io) — Manage your memories, instructions, and output wiki
- [Feedback](https://ai.actingweb.io) — Use the in-app feedback form for bug reports and feature requests
- [Connecting Claude](https://ai.actingweb.io/connect-claude) — Setup guide for every Claude surface
- [Support](https://ai.actingweb.io/support) — Contact, billing and account deletion
- **Security:** report vulnerabilities to [security@actingweb.io](mailto:security@actingweb.io) (also published at [/.well-known/security.txt](https://ai.actingweb.io/.well-known/security.txt))
