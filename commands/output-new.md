---
description: Create a new output in the Emm AI wiki (drafts, research, plans, dashboards, …).
---

Help me capture a new output in the Emm AI wiki. Arguments: $ARGUMENTS

Guide me through the `output_create` MCP tool:

1. Confirm a **category** — default options are `email`, `news`, `research`, `task`, `log`, `improvement`, `actions`, `space`. If none fits, propose a new category slug (Emm AI mints new categories on first `output_create`).
2. Propose a **slug** (lowercase, kebab-case) and a **title**.
3. Write a one-line **`short_description`** that will surface in list views.
4. Confirm the **body** (valid Markdown; H1, then H2/H3 sub-sections; YAML frontmatter at top for metadata when relevant).
5. Run `output_create` and report the resulting `category:id` plus a Wiki link.

If I want a styled HTML report saved as designed (read-only, beta accounts), use `output_publish_html` instead: one self-contained HTML document with an inline `<style>`; scripts (inline or from an `https:` CDN) and inline handlers run in a sandboxed frame that cannot make requests beyond loading those scripts and styles (never put private data in their URLs), so keep the numbers readable without them; no remote images or stylesheets. Text I will keep editing stays Markdown.
