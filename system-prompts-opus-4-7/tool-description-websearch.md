<!--
name: 'Tool Description: WebSearch'
description: Tool description for web search functionality
ccVersion: 2.1.120
variables:
  - CURRENT_MONTH_YEAR
-->
Searches the web for up-to-date info beyond Claude's knowledge cutoff. Returns results as markdown-linked blocks.

After answering, list the relevant URLs you used in a `Sources:` section as markdown hyperlinks: `[Title](URL)`.

- Domain include/exclude filtering supported.
- US-only.
- Use the current year (${CURRENT_MONTH_YEAR}) in queries — don't search for last year's data when asked for "latest".
- Run independent searches for different queries in parallel — one message, multiple WebSearch calls.
