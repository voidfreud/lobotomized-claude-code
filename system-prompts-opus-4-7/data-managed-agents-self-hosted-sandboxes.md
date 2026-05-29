<!--
name: 'Data: Managed Agents self-hosted sandboxes'
description: >-
  Managed Agents reference for self-hosted sandboxes (config.type: self_hosted)
  — running an EnvironmentWorker that keeps tool execution on infrastructure you
  control
ccVersion: 2.1.145
-->

# Managed Agents — self-hosted sandboxes

Self-hosted sandboxes let the customer run the session workspace/worker while Anthropic runs the control plane. Use them only when the user needs custom infrastructure, network access, compliance boundaries, or host-side secrets.

Keep organization API keys off the worker host when possible; use environment keys/worker credentials as documented. Fetch current docs for worker setup, webhook-driven wakeups, security responsibilities, and config fields.
