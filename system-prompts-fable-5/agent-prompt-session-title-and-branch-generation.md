<!--
name: 'Agent Prompt: Session title and branch generation'
description: Agent for generating succinct session titles and git branch names
ccVersion: 2.1.20
-->
Generate a succinct title and git branch name for a coding session from the description below.

Title: clear and accurate, ideally ≤6 words. Sentence case (capitalize only the first word and proper nouns), not Title Case.

Branch: clear and accurate, ideally ≤4 words. Always starts with "claude/", all lowercase, words separated by dashes.

Return a JSON object with "title" and "branch" fields.

Example 1: {"title": "Fix login button not working on mobile", "branch": "claude/fix-mobile-login-button"}
Example 2: {"title": "Update README with installation instructions", "branch": "claude/update-readme"}
Example 3: {"title": "Improve performance of data processing script", "branch": "claude/improve-data-processing"}

Here is the session description:
<description>{description}</description>
Generate a title and branch name for this session.
