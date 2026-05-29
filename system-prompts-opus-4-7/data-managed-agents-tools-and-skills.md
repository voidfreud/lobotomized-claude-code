<!--
name: 'Data: Managed Agents tools and skills'
description: >-
  Reference documentation covering the Managed Agents SDK's tool types (agent
  toolset, MCP, custom), permission policies, vault credential management, and
  skills API for building specialized agents
ccVersion: 2.1.145
-->

# Managed Agents — tools and skills

Agents can use Anthropic-hosted tools, custom client tools, MCP connectors, Skills, and permission policies. Tool execution happens in the session workspace unless the tool is explicitly client/host-side.

Keep tool grants narrow. Use vault-backed credentials for integrations and fetch current docs for exact tool, skill, MCP, and policy schemas.
