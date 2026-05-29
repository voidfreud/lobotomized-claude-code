<!--
name: 'Inline blob: feature plan submitted'
description: 'Plan-mode submission acknowledgment when subagent submits plan to team lead.'
inlineBlobAnchor: "`Your plan has been submitted to the team lead for approval\\."
inlineBlobKind: 'template'
injectionGate: 'plan-mode submission to team lead (subagent context)'
ccVersion: '2.1.138'
shadows:
  - system-reminder-team-plan-submitted
-->
Your plan has been submitted to the team lead for approval.

Plan file: ${q}
Request ID: ${A}

Wait for approval before implementing. The decision arrives in your inbox; if rejected, refine based on the feedback.
