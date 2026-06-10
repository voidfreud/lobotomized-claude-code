<!--
name: 'Inline blob: session-specific Explore subagent tip'
description: >-
  Bullet in "# Session-specific guidance" telling the model to spawn the Explore
  subagent for broad code searches.
inlineBlobAnchor: >-
  `For broad codebase exploration or research that'll take more than
  \$\{[$\w]+\} queries, spawn
inlineBlobKind: template
injectionGate: Agent tool loaded; not in remote-planning mode
ccVersion: 2.1.141
-->
For broad codebase exploration or research that'll take more than ${AJ7} queries, spawn ${K9} with subagent_type=${r_H.agentType}. Otherwise use ${Y} directly.
