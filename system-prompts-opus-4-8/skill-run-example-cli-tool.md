<!--
name: 'Skill: run example — CLI tool'
description: >-
  Bundled example doc (examples/cli.md) for the run skill: installing, invoking,
  and testing a command-line tool
ccVersion: 2.1.145
-->
# Example: CLI tool

CLIs usually have no background process, ports, or lifecycle. The skill covers **installation**, **representative invocations**, and **testing**.

## What matters

- **How to get the binary on \`PATH\`.** Installed globally? Run via \`npx\`/\`uv run\`? Built to \`./target/release/foo\`? Be explicit.
- **Two or three example invocations** covering the main use cases, with expected output so a reader can tell it worked.
- **Exit codes** if they're meaningful (e.g. linter returns 1 on findings).
- **Stdin behavior** if the tool reads from stdin.

## Example snippet

> ---
> name: run-mytool
> description: Build, install, and run mytool. Use when asked to run mytool, test it, or verify it's installed correctly.
> ---
>
> ## Setup
>
> \`\`\`bash
> pip install -e .
> \`\`\`
>
> This puts \`mytool\` on PATH. Verify:
>
> \`\`\`bash
> mytool --version
> # → mytool 0.3.1
> \`\`\`
>
> ## Run
>
> Process a single file:
>
> \`\`\`bash
> mytool process input.json
> # → Processed 42 records, wrote output.json
> \`\`\`
>
> Read from stdin, write to stdout:
>
> \`\`\`bash
> cat input.json | mytool process -
> \`\`\`
>
> Lint a directory (exits non-zero on problems):
>
> \`\`\`bash
> mytool lint ./src
> echo $?  # 0 if clean, 1 if issues found
> \`\`\`
>
> ## Test
>
> \`\`\`bash
> pytest
> \`\`\`

## Keep it short

A CLI run skill can be compact — \`--help\` covers the full flag list. Show enough that an agent can build it, confirm it works, and run the tests.
