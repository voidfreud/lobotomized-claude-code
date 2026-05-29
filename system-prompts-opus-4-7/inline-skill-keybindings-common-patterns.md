<!--
name: 'Inline blob: skill keybindings common patterns'
description: 'keybindings-help skill: common patterns.'
inlineBlobAnchor: '[$\w]+=\["## Common Patterns",'
inlineBlobKind: 'array'
injectionGate: 'keybindings-help skill'
ccVersion: '2.1.139'
inlineBlobRawPassthrough: 'true'
-->
"## Common Patterns","","### Rebind a key","To change the external editor shortcut from `ctrl+g` to `ctrl+e`:","```json","{\n  \"context\": \"Chat\",\n  \"bindings\": {\n    \"ctrl+g\": null,\n    \"ctrl+e\": \"chat:externalEditor\"\n  }\n}","```","","### Add a chord binding","```json","{\n  \"context\": \"Global\",\n  \"bindings\": {\n    \"ctrl+k ctrl+t\": \"app:toggleTodos\"\n  }\n}","```"
