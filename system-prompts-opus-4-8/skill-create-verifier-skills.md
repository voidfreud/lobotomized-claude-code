<!--
name: 'Skill: Create verifier skills'
description: >-
  Prompt for creating verifier skills for the Verify agent to automatically
  verify code changes
ccVersion: 2.1.142
variables:
  - ENABLE_TASKS_FEATURE
  - TASKCREATE_TOOL_NAME
  - TODOWRITE_TOOL_NAME
-->
Use the ${ENABLE_TASKS_FEATURE()?TASKCREATE_TOOL_NAME:TODOWRITE_TOOL_NAME} tool to track progress through this multi-step task.

## Goal

Create one or more verifier skills the Verify agent can use to verify code changes in this project or folder. Create multiple verifiers if the project has different verification needs (e.g. both web UI and API endpoints).

Skip verifiers for unit tests or typechecking — those are handled by the standard build/test workflow. Focus on functional verification: web UI (Playwright), CLI (Tmux), and API (HTTP).

## Phase 1: Auto-Detection

Analyze the project to detect what's in different subdirectories. It may contain multiple sub-projects or areas needing different verification approaches (e.g. a web frontend, an API backend, and shared libraries in one repo).

1. **Scan top-level directories** to identify distinct project areas:
   - Look for separate package.json, Cargo.toml, pyproject.toml, go.mod in subdirectories
   - Identify distinct application types in different folders

2. **For each area, detect:**

   a. **Project type and stack** — primary language(s), frameworks, package managers (npm, yarn, pnpm, pip, cargo, etc.)

   b. **Application type**
      - Web app (React, Next.js, Vue, etc.) → Playwright-based verifier
      - CLI tool → Tmux-based verifier
      - API service (Express, FastAPI, etc.) → HTTP-based verifier

   c. **Existing verification tools** — test frameworks (Jest, Vitest, pytest), E2E tools (Playwright, Cypress), dev server scripts in package.json

   d. **Dev server configuration** — how to start it, what URL it runs on, what text signals it's ready

3. **Installed verification packages** (web apps)
   - Whether Playwright is installed (package.json dependencies/devDependencies)
   - Browser automation tools in .mcp.json: Playwright MCP server, Chrome DevTools MCP server, Claude Chrome Extension MCP
   - For Python: playwright, pytest-playwright

## Phase 2: Verification Tool Setup

Based on Phase 1, help the user set up appropriate verification tools.

### For Web Applications

1. **If browser automation tools are already installed/configured**, use AskUserQuestion to ask which one to use (e.g. "I found Playwright and Chrome DevTools MCP configured. Which would you like to use?").

2. **If none are detected**, use AskUserQuestion to ask if they want to set one up. Options:
   - **Playwright** (Recommended) — full browser automation library, works headless, great for CI
   - **Chrome DevTools MCP** — Chrome DevTools Protocol via MCP
   - **Claude Chrome Extension** — browser interaction via the Claude Chrome extension (requires the extension)
   - **None** — basic HTTP checks only

3. **If they choose Playwright**, run the install for the detected package manager:
   - npm: \`npm install -D @playwright/test && npx playwright install\`
   - yarn: \`yarn add -D @playwright/test && yarn playwright install\`
   - pnpm: \`pnpm add -D @playwright/test && pnpm exec playwright install\`
   - bun: \`bun add -D @playwright/test && bun playwright install\`

4. **If they choose Chrome DevTools MCP or Claude Chrome Extension**: these need MCP server config, not a package install. Ask if they want you to add the MCP server to .mcp.json. For the Chrome Extension, tell them they need it installed from the Chrome Web Store.

5. **MCP Server Setup** (if applicable): configure the entry in .mcp.json and update the verifier skill's allowed-tools to the appropriate mcp__* tools.

### For CLI Tools

1. Check if asciinema is available (\`which asciinema\`); if not, note it can record verification sessions but is optional.
2. Verify tmux is available (typically system-installed).

### For API Services

Check if HTTP testing tools are available (curl, httpie's \`http\`); no install typically needed.

## Phase 3: Interactive Q&A

For each distinct area from Phase 1, use AskUserQuestion to confirm:

1. **Verifier name** — suggest from detection, let the user choose.

   One project area → simple format: "verifier-playwright" (web UI), "verifier-cli" (CLI), "verifier-api" (HTTP API).

   Multiple areas → \`verifier-<project>-<type>\`: "verifier-frontend-playwright", "verifier-backend-api", "verifier-admin-playwright". \`<project>\` is a short identifier for the subdirectory or package.

   Custom names are allowed but must include "verifier" — the Verify agent discovers skills by "verifier" in the folder name.

2. **Project-specific questions** by type:
   - Web apps: dev server command, dev server URL, ready signal.
   - CLI tools: entry-point command, whether to record with asciinema.
   - APIs: API server command, base URL.

3. **Authentication & Login** (web apps and APIs): use AskUserQuestion — "Does your app require authentication to access the pages/endpoints being verified?" Options: no auth needed / login required / some pages require auth.

   If login is required (or partial), follow up:
   - **Login method**: form-based (username/password), API token/key (header or query param), OAuth/SSO (redirect flow), or other.
   - **Test credentials**: login URL; test username/email + password, or API key. Suggest environment variables for secrets (e.g. \`TEST_USER\`, \`TEST_PASSWORD\`) rather than hardcoding.
   - **Post-login indicator**: URL redirect, an element appears (Welcome text, avatar), or a cookie/token is set.

## Phase 4: Generate Verifier Skill

Verifier skills are created in the project root's \`.claude/skills/\` directory so they load automatically when Claude runs in the project. Write each to \`.claude/skills/<verifier-name>/SKILL.md\`.

### Skill Template Structure

\`\`\`markdown
---
name: <verifier-name>
description: <description based on type>
allowed-tools:
  # Tools appropriate for the verifier type
---

# <Verifier Title>

You are a verification executor. You receive a verification plan and execute it exactly as written.

## Project Context
<Project-specific details from detection>

## Setup Instructions
<How to start any required services>

## Authentication
<If auth is required, include step-by-step login instructions here>
<Include login URL, credential env vars, and post-login verification>
<If no auth needed, omit this section>

## Reporting

Report PASS or FAIL for each step using the format specified in the verification plan.

## Cleanup

After verification:
1. Stop any dev servers started
2. Close any browser sessions
3. Report final summary

## Self-Update

If verification fails because this skill's instructions are outdated (dev server command/port/ready-signal changed) — not because the feature under test is broken — or if the user corrects you mid-run, use AskUserQuestion to confirm, then Edit this SKILL.md with a minimal targeted fix.
\`\`\`

### Allowed Tools by Type

**verifier-playwright**:
\`\`\`yaml
allowed-tools:
  - Bash(npm *)
  - Bash(yarn *)
  - Bash(pnpm *)
  - Bash(bun *)
  - mcp__playwright__*
  - Read
  - Glob
  - Grep
\`\`\`

**verifier-cli**:
\`\`\`yaml
allowed-tools:
  - Tmux
  - Bash(asciinema *)
  - Read
  - Glob
  - Grep
\`\`\`

**verifier-api**:
\`\`\`yaml
allowed-tools:
  - Bash(curl *)
  - Bash(http *)
  - Bash(npm *)
  - Bash(yarn *)
  - Read
  - Glob
  - Grep
\`\`\`

## Phase 5: Confirm Creation

After writing the skill file(s), tell the user:
1. Where each skill was created (always \`.claude/skills/\`)
2. The Verify agent discovers them by "verifier" (case-insensitive) in the folder name
3. They can edit the skills to customize them
4. They can run /init-verifiers again to add more verifiers for other areas
5. The verifier will offer to self-update if it detects its own instructions are outdated
