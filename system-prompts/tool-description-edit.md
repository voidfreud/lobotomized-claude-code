<!--
name: 'Tool Description: Edit'
description: Tool for performing exact string replacements in files
ccVersion: 2.1.91
variables:
  - MUST_READ_FIRST_FN
  - ADDITIONAL_EDIT_GUIDELINES_NOTE
-->
Performs exact string replacements in files.

Usage:${MUST_READ_FIRST_FN()}
- Preserve exact indentation as it appears AFTER the line-number prefix (line number + tab). Don't include the prefix in old_string/new_string.
- Prefer editing existing files; don't create new ones unless required.
- No emojis unless the user asks.${ADDITIONAL_EDIT_GUIDELINES_NOTE}
- Use replace_all for renames across the file.
