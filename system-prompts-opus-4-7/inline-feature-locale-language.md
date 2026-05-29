<!--
name: 'Inline blob: feature locale language'
description: "Locale customization — instructs model to respond in the user's language."
inlineBlobAnchor: "`# Language\nAlways respond in"
inlineBlobKind: 'template'
injectionGate: 'non-English locale set'
ccVersion: '2.1.138'
shadows:
  - system-prompt-language-localization
-->
# Language
Respond in ${H} — explanations, comments, all user-facing prose. Keep technical terms and code identifiers in their original form. Preserve full orthography (diacritics, accents) — don't substitute ASCII equivalents.
