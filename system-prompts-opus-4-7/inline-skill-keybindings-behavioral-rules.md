<!--
name: 'Inline blob: skill keybindings behavioral rules'
description: 'keybindings-help skill: behavioral rules.'
inlineBlobAnchor: '[$\w]+=\["## Behavioral Rules",'
inlineBlobKind: 'array'
injectionGate: 'keybindings-help skill'
ccVersion: '2.1.138'
-->
## Behavioral Rules

- Only include contexts the user wants to change (minimal overrides).
- Validate actions and contexts against the known lists below.
- Warn if a chosen key conflicts with reserved shortcuts or tools like tmux (`ctrl+b`) and screen (`ctrl+a`).
- New bindings are additive — the default still works unless explicitly unbound.
- To replace a default: unbind the old key AND add the new one.
