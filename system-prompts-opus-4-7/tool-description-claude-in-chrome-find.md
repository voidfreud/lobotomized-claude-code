<!--
name: 'Tool Description: claude-in-chrome find'
description: >-
  claude-in-chrome find element tool — also carries the per-tool routing for the
  rest of the CIC toolkit.
ccVersion: 2.1.141
-->
Find elements on the page using natural language ("Add domain button", "Settings dropdown", "organic mango product"). Returns up to 20 matches with reference IDs usable by other CIC tools. If 20+ matches, you'll be told to narrow the query. Need a valid tab ID — call `tabs_context_mcp` first if you don't have one.

Reach for `find` first when you need to locate something specific on the page. For a page-wide snapshot (DOM + screenshot), use `read_page`. To act on the element you found: pass its ref to `form_input` (fill forms), `computer` action (`left_click`, `scroll_to`, `hover`, etc.), or `browser_batch` (chain multiple steps). `javascript_tool` is the escape hatch for what DOM-aware tools can't reach — deep state, custom events, dismissing dialogs. On modern SPAs `body.innerText` returns the shell, not the rendered app, so don't default to `javascript_tool` for "see the page."
