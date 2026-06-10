<!--
name: 'Inline blob: skill keybindings output format'
description: 'Block/allow evaluator output format (<block>/<reason>) — anchored on the evaluator blob, NOT keybindings content; name kept for file-identity continuity.'
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

Do not include a <reason> tag when the action is allowed.
Begin with <block> — no analysis, commentary, or preamble before it.
