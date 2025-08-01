Building Effective Agents for VS Code Copilot

This guide explains how to create a chatmode.md file that turns GitHub Copilot Agent mode into a specialised AI assistant, and how to enrich it with Model Context Protocol (MCP) tools.


---

1. What’s a chatmode.md?

chatmode.md is a‑single Markdown file that lives at the root of a workspace (or inside .vscode/).
It packages:

Section	Purpose

YAML front‑matter	Declares metadata (name, description, model, tools, limits).
System prompt	Fixed instructions that shape the agent’s personality and constraints.
User prompt template	How end‑user input is injected.
Examples / few‑shot	Optional demonstrations to guide the model.


When you open Copilot Chat and switch to your custom mode, VS Code will merge this file with user input and pass it to the selected model.


---

2. Minimal skeleton

name: "Release‑Planner"
description: "Plans and applies semantic‑release changes across the monorepo."
mode: agent
model: "gpt‑4o‑mini"
max_steps: 30

tools: ['changes', 'codebase', 'editFiles', 'extensions', 'fetch', 'findTestFiles', 'githubRepo', 'new', 'problems', 'runCommands', 'runNotebooks', 'runTasks', 'runTests', 'search', 'searchResults', 'terminalLastCommand', 'terminalSelection', 'testFailure', 'usages', 'vscodeAPI', 'sequentialthinking', 'context7', 'activePullRequest', 'copilotCodingAgent']
---

## System prompt (markdown)

You are Release‑Planner, an expert …
---

## User prompt

{user_input}

Save the file as release-planner.chatmode.md.
VS Code recognises any file ending in .chatmode.md.


---

3. Front‑matter fields explained

Field	Required	Notes

name	✔	Human‑readable title (≤ 40 chars).
description	✔	Short summary (≤ 160 chars). Shows in mode picker.
mode	✔	Must be agent for autonomous workflows.
model	✔	Any model ID Copilot supports, e.g. gpt‑4o-mini, gpt‑4o or gpt‑4+.
max_steps	✖	Hard stop to prevent infinite loops (default 20).
tools	✖	List of built‑in or MCP tools the agent may call.



---

4. Adding MCP tools

1. Install or run an MCP server (see libraries below).


2. In the tools list, reference it either by URL (for local/HTTP servers) or by VS Code server ID if you installed it through the Marketplace:

tools:
  - editFiles
  - mcp: "ghcr.io/org/my-mcp:latest"      # container image
  - mcp: "http://localhost:4000"          # dev server
  - mcp: "search"                         # ID provided by VS Code extension


3. Restart Copilot Chat or Reload Window so the agent discovers the new tools.


4. Optionally document each tool’s affordances in the System prompt so the model knows when to call it.



Best practices

Keep tool count tight; extra tools increase reasoning cost.

Group related server endpoints into a single MCP server when possible.

Prefer stateless tool design; provide all parameters explicitly.



---

5. Example: “Database Refactorer” agent

<details>
<summary>Click to view full `chatmode.md`</summary>name: "DB Refactorer"
description: "Creates and migrates SQL schemas with automatic tests."
mode: agent
model: "gpt‑4o"
max_steps: 40

tools:
  - editFiles
  - mcp: "https://sqlsmith.dev:8443"
---

## System prompt

You are an experienced database architect …
* The `editFiles` tool may only be used to apply schema or migration files inside `migrations/**`.
* The `sqlsmith` MCP server provides:
  - `plan_migration(target_version)`
  - `run_query(sql)`
Use these tools to analyse diffs, plan, and apply migrations safely.

## User prompt

{user_input}

</details>
---

6. Discovering MCP servers

Source	Scope	Link

VS Code Featured Directory	Curated list that installs with one click inside VS Code. Best for stable, Microsoft‑reviewed servers.	https://code.visualstudio.com/mcp
Community Library	Open, community‑maintained catalogue of thousands of MCP servers spanning APIs, databases, dev‑ops and creative tools.	https://github.com/punkpeye/awesome-mcp-servers


Tip: If you install the Copilot MCP Marketplace extension it adds an in‑product browser for both catalogues.


---

7. Testing your agent

1. Switch Copilot Chat to Agent mode, then pick your custom mode.


2. Run a small, non‑destructive task and watch the plan + tool calls.


3. Use Ctrl+Enter to step‑through if you enabled review in front‑matter.


4. Iterate on instructions based on observed behaviour.




---

8. Troubleshooting

Symptom	Likely cause

“MCP server not found”	Wrong URL or extension not installed. Check View › AI Tools › MCP Servers.
Agent loops without progress	Lower max_steps, add clearer exit criteria in the system prompt.
Changes applied to wrong file	Qualify allowed paths in the system prompt, use plan‑only first.



---

9. Further reading

Use MCP servers in VS Code – official how‑to.

Extending Copilot Chat with MCP – protocol specification and SDKs.

Use Agent mode in VS Code – deeper dive into autonomous workflows.


Happy hacking! 🎉

