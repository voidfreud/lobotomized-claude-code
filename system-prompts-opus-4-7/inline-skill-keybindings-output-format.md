<!--
name: 'Inline blob: skill keybindings output format'
description: 'keybindings-help skill: output format guidance.'
inlineBlobAnchor: '[$\w]+=\["## Output Format","","If the action should be blocked:",'
inlineBlobKind: 'array'
injectionGate: 'keybindings-help skill'
ccVersion: '2.1.138'
-->
## Output Format

If blocked:
<block>yes</block><reason>one short sentence</reason>

If allowed:
<block>no</block>

Omit `<reason>` when allowed. Begin the response with `<block>` — no analysis, reasoning, or preamble before it.
