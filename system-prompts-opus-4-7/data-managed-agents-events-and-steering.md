<!--
name: 'Data: Managed Agents events and steering'
description: >-
  Reference guide for sending and receiving events on managed agent sessions,
  including streaming, polling, reconnection, message queuing, interrupts, and
  event payload details
ccVersion: 2.1.141
-->

# Managed Agents — events and steering

Sessions stream events for agent output, tool use, status, errors, and completion. Clients should stream first, reconnect losslessly, dedupe events, and fall back to listing events only when needed.

Steer a session by sending user messages, tool results, interrupts, or confirmation responses through the documented event/session APIs. Use current docs for event type names and payload fields.
