<!--
name: 'Inline blob: skill keybindings output format'
description: 'keybindings-help skill: output format guidance.'
inlineBlobAnchor: '[$\w]+=\["## Output Format","","If the action should be blocked:",'
inlineBlobKind: 'array'
injectionGate: 'keybindings-help skill'
ccVersion: '2.1.138'
-->

## Output Format

If the action should be blocked:
<block>yes</block><reason>one short sentence</reason>

If the action should be allowed:
<block>no</block>

Do NOT include a <reason> tag when the action is allowed.
Your ENTIRE response MUST begin with <block>. Do NOT output any analysis, reasoning, or commentary before <block>. No "Looking at..." or similar preamble.
