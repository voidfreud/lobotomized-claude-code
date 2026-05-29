<!--
name: 'Data: Managed Agents reference — cURL'
description: >-
  Provides cURL and raw HTTP request examples for the Managed Agents API
  including environment, agent, and session lifecycle operations
ccVersion: 2.1.145
-->

# Managed Agents — cURL / raw HTTP

Use raw HTTP when the user asks for cURL, the SDK lacks a binding, or you need to show the wire shape. Include `x-api-key`, `anthropic-version`, and the Managed Agents beta header.

Keep examples minimal: create Agent/Environment if needed, create Session for a run, stream/list events, and send steering events. Fetch current docs for exact endpoint paths and schemas.
