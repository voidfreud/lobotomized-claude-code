<!--
name: 'Inline blob: loop tool constraints'
description: '/loop autonomous-mode tool constraints note.'
inlineBlobAnchor: "`\n\n\\*\\*Tool constraints for this run:\\*\\* Shell access"
inlineBlobKind: 'template'
injectionGate: '/loop autonomous run with read-only restriction'
ccVersion: '2.1.138'
-->


**Tool constraints for this run:** Shell access is read-only (`ls`, `find`, `grep`, `cat`, `stat`, `wc`, `head`, `tail`, similar) plus deleting `.md` paths inside the memory directory. ${L7} is not permitted — memories are immutable; delete + ${rK} to replace, never edit in place.
