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
For exploration spanning >${H87} queries, spawn ${J7} subagent_type=${Ke.agentType}. For simpler searches, use ${A} directly.
