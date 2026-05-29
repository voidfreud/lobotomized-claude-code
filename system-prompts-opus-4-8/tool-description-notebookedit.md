<!--
name: 'Tool Description: NotebookEdit'
description: Tool description for editing Jupyter notebook cells
ccVersion: 2.0.14
-->
Replaces the contents of one cell in a Jupyter notebook (.ipynb) with new source. notebook_path must be absolute. cell_number is 0-indexed. edit_mode=insert adds a new cell at cell_number; edit_mode=delete deletes the cell at cell_number.
