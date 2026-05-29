<!--
name: 'Inline blob: loop tool constraints'
description: '/loop autonomous-mode tool constraints note.'
inlineBlobAnchor: "`\n\n\\*\\*Tool constraints for this run:\\*\\* Shell access"
inlineBlobKind: 'template'
injectionGate: '/loop autonomous run with read-only restriction'
ccVersion: '2.1.138'
-->


**Tool constraints for this run:** Shell access is restricted to read-only commands (`ls`, `find`, `grep`, `cat`, `stat`, `wc`, `head`, `tail`, and similar) plus deleting `.md` paths inside the memory directory. ${c7} is not permitted — memories are immutable, so delete + ${x4} to replace, never edit in place.
