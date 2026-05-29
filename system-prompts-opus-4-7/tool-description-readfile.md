<!--
name: 'Tool Description: ReadFile'
description: Tool description for reading files
ccVersion: 2.1.128
variables:
  - MAX_LINES_CONSTANT
  - CONDITIONAL_LENGTH_NOTE
  - CAT_DASH_N_NOTE
  - READ_FULL_FILE_NOTE
  - CAN_READ_PDF_FILES_FN
  - ADDITIONAL_READ_NOTE
-->
Reads a file from the local filesystem.

- file_path must be absolute, not relative.
- Reads up to ${MAX_LINES_CONSTANT} lines from the start by default.
- Read files in full. Only use offset/limit when the file exceeds 10,000 lines — chunking smaller files produces partial context and forces re-reads. A second chunked read of the same file is always wrong; read it whole instead.
- Results are returned using cat -n format, with line numbers starting at 1
- Reads images (PNG/JPG/etc.) — they render visually.${CAN_READ_PDF_FILES_FN()?`
- Reads PDFs. For PDFs >10 pages, the pages parameter is required (e.g. pages: "1-5"). Max 20 pages per call.`:""}
- Reads Jupyter notebooks (.ipynb) — returns all cells with outputs.
- Files only, not directories — list dirs with the shell tool.
- Empty files return a system-reminder warning instead of contents.
- Read multiple files in parallel when independent — one message, multiple Read calls.${ADDITIONAL_READ_NOTE}
