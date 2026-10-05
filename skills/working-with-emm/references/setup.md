# Setup Guide

## The user asks to "run my Emm setup" (Emm already connected)

Setup on a connected account means personalising it, and it comes before any
agent run:

1. Call `instruction_settings()`. If it says About you is still empty, start
   the interview right away (the user asked for it; don't ask whether to).
   Follow Step 4 of the setup guide at `status().links.setup_guide`: open
   questions one at a time, read the whole setup back, then save it once with
   `instruction_settings_update`.
2. Then the first `agent_run`, and the schedule from Step 5 of that guide.

The rest of this file is for connecting Emm: first use, or after losing
credentials. Skip it if memory tools are already working.

## Requirements

- **OpenClaw users**: mcporter must be installed (`npm install -g mcporter`)

## Option A — Platform-managed MCP (Claude.ai, ChatGPT, etc.)

The user does this in their app's settings — hand them the values below, don't
add the server yourself even if you have the capability to:

- **MCP endpoint:** `https://ai.actingweb.io/mcp`
- **Auth:** OAuth 2.0 (Google, GitHub, or Apple)

Follow your platform's guide for adding an MCP server, then have the user sign
in when prompted.

## Option B — mcporter (CLI agents, OpenClaw, custom setups)

These commands are for the user to run in their own terminal. Even with shell
access, don't run them on the user's behalf — OAuth login and MCP-config
writes are the user's actions here, same as Option A's browser sign-in.

**1. Have the user install mcporter**

```bash
npm install -g mcporter
```

**2. Have the user register the Emm AI server**

```bash
mcporter config add emm https://ai.actingweb.io/mcp --auth oauth
```

**3. Have the user authenticate**

```bash
mcporter auth emm --log-level debug
```

This opens a browser for Google OAuth. If it succeeds, skip to step 4.

**If authentication fails** (common in headless or GUI-less environments, where
the session closes before the browser callback completes): the debug output
prints a line like
> `If the browser did not open, visit https://...`

Have the user open that URL in a browser on the same machine and finish
signing in there while the command is still waiting. If the machine has no
browser, have the user run `mcporter auth emm` on a machine that has one, or
connect through a platform-managed client instead (Option A). Don't complete
the OAuth flow yourself: signing in is the user's action.

**4. Have the user verify**

```bash
mcporter list emm --schema
```

Should list the available memory tools. You're done.

## Emm unreachable, or a tool call fails with an auth error

Work through in order:

1. **Is the server connected at all?** Call `status()`. If it errors or the
   tool isn't in your loaded tool list, the connector isn't configured or
   isn't enabled for this conversation — see Option A or B above.
2. **Is the auth still valid?** An expired or revoked OAuth session fails
   tool calls with an auth error even though the connector still shows as
   configured. Ask the user to re-run the platform's connect/authenticate step
   (Option A) or `mcporter auth emm --log-level debug` (Option B).
3. **Test the connector independently of your current task.** `mcporter list
   emm --schema` (Option B) or a bare `status()` call (either option) isolates
   whether the problem is the connection itself or something about the
   specific call you were making.
4. **Tool names are case-sensitive.** `Memory_Search` or `memorySearch` will
   not resolve — use the exact name from your loaded tool list
   (`memory_search`, lowercase with underscores).

If all four check out and the call still fails, see
[error handling during a run](mission-control.md#error-handling-during-a-run)
for the structured-envelope codes.
