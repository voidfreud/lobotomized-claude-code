<!--
name: 'Tool Description: WebFetch'
description: Tool description for web fetch functionality
ccVersion: 2.1.14
-->

Fetches a URL, converts HTML to markdown, processes it with a small fast model using your prompt, and returns the response. Read-only.

- If an MCP web-fetch tool is available, prefer it — fewer restrictions.
- HTTP auto-upgrades to HTTPS. 15-minute cache for repeated URLs.
- On a cross-host redirect, the tool reports the redirect URL — re-fetch with that URL.
- For GitHub URLs, prefer \`gh\` via Bash (e.g. \`gh pr view\`, \`gh issue view\`, \`gh api\`).
- Fetch independent URLs in parallel — one message, multiple WebFetch calls.
