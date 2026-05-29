<!--
name: 'Data: Managed Agents webhooks'
description: >-
  Reference documentation for Managed Agents webhooks, including endpoint
  registration, signature verification, payload envelopes, supported event
  types, delivery behavior, and pitfalls
ccVersion: 2.1.132
-->

# Managed Agents — webhooks

Webhooks notify your HTTPS endpoint when Managed Agents resources change. Treat payloads as thin event envelopes: verify the signature, dedupe deliveries, then retrieve current resource state through the API.

Use webhooks for async orchestration instead of long polling. Fetch current docs for supported event types and signature details.
