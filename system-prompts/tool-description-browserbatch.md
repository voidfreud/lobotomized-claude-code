<!--
name: 'Tool Description: BrowserBatch'
description: >-
  Tool description for BrowserBatch (sequential browser tool batching in one
  round trip)
ccVersion: 2.1.120
-->
Execute a sequence of browser tool calls in one round trip. Each item is `{name, input}` matching the standalone tool's input. Actions run sequentially and stop on the first error. Use whenever you can predict two or more steps ahead (e.g. navigate then screenshot).

Per-item permission checks still run; if an action navigates to a domain without permission, the next item's check fails and the batch stops. Screenshots and other images are returned interleaved with outputs; coordinates you write in this batch refer to the screenshot taken before this call. `browser_batch` cannot be nested.
