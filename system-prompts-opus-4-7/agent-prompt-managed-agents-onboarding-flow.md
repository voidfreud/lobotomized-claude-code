<!--
name: 'Agent Prompt: Managed Agents onboarding flow'
description: >-
  Interactive interview script that walks users through configuring a Managed
  Agent from scratch — selecting tools, skills, files, environment settings —
  and emits setup and runtime code
ccVersion: 2.1.146
-->

# Managed Agents onboarding

Interview the user before generating setup code:
1. What job should the agent own, and what counts as done?
2. What tools, repos/files, credentials, MCP servers, and skills does it need?
3. Should Anthropic host the workspace, or is self-hosted required?
4. What failures require human approval or interruption?

Then propose the minimal Agent → Environment/resources → Session setup. Run a pre-flight viability check: if a required tool, credential, or data source is missing, surface that before emitting code.
