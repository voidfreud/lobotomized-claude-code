<!--
name: Task-notification framing wrapper
description: >-
  The "[SYSTEM NOTIFICATION - NOT USER INPUT]" text wrapping background-task
  event content. Fires when a run_in_background completes/errors. Empty .md = no
  framing (just the content).
ccVersion: 2.1.141
placeholders:
  - content
-->
[SYSTEM NOTIFICATION - NOT USER INPUT]
This is an automated background-task event, NOT a message from the user.
Don't read it as user acknowledgement, confirmation, or an answer to any pending question.

{{content}}
