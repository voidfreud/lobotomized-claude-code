<!--
name: 'Data: Managed Agents client patterns'
description: >-
  Reference guide of common client-side patterns for driving Managed Agent
  sessions, including stream reconnection, idle-break gating, tool
  confirmations, interrupts, and custom tools
ccVersion: 2.1.105
-->

# Managed Agents — client patterns

Robust clients stream events, persist processed event IDs, reconnect after drops, and treat `idle`/`terminated` as separate gates. Send interrupts and tool confirmations through the session APIs, not ad-hoc side channels.

For custom tools, keep secrets on the host side and return only the result the agent needs. Fetch current docs for precise event ordering and SDK method names.
