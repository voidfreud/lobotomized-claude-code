<!--
name: 'Inline blob: skill doctor validation'
description: '/doctor validation messages for keybindings.json.'
inlineBlobAnchor: '[$\w]+=\["## Validation with /doctor",'
inlineBlobKind: 'array'
injectionGate: 'keybindings-help skill (loaded with /doctor section)'
ccVersion: '2.1.138'
inlineBlobRawPassthrough: 'true'
-->
"## Validation with /doctor","",'The `/doctor` command includes a "Keybinding Configuration Issues" section that validates `~/.claude/keybindings.json`.',"","### Common Issues and Fixes","",m4q(["Issue","Cause","Fix"],[['`keybindings.json must have a "bindings" array`',"Missing wrapper object",'Wrap bindings in `{ "bindings": [...] }`'],['`"bindings" must be an array`',"`bindings` is not an array",'Set `"bindings"` to an array: `[{ context: ..., bindings: ... }]`'],['`Unknown context "X"`',"Typo or invalid context name","Use exact context names from the Available Contexts table"],['`Duplicate key "X" in Y bindings`',"Same key defined twice in one context","Remove the duplicate; JSON uses only the last value"],['`"X" may not work: ...`',"Key conflicts with terminal/OS reserved shortcut","Choose a different key (see Reserved Shortcuts section)"],['`Could not parse keystroke "X"`',"Invalid key syntax","Check syntax: use `+` between modifiers, valid key names"],['`Invalid action for "X"`',"Action value is not a string or null",'Actions must be strings like `"app:help"` or `null` to unbind']]),"","### Example /doctor Output","","```","Keybinding Configuration Issues","Location: ~/.claude/keybindings.json",'  └ [Error] Unknown context "chat"',"    → Valid contexts: Global, Chat, Autocomplete, ...",'  └ [Warning] "ctrl+c" may not work: Terminal interrupt (SIGINT)',"```","","**Errors** prevent bindings from working and must be fixed. **Warnings** indicate potential conflicts but the binding may still work."
