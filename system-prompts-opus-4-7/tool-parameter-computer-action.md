<!--
name: 'Tool Parameter: Computer action'
description: Action parameter options for the Chrome browser computer tool
ccVersion: 2.0.71
-->
Beyond standard click/type/screenshot/scroll/key:
- `triple_click` — select a paragraph or line.
- `scroll_to` — scroll an element into view by reference ID from `read_page` or `find`. Prefer over coordinate `scroll` when you have a ref.
- `zoom` — screenshot a region for closer inspection of dense UI.
- `left_click_drag` — drag from start to end coordinates (sliders, file drag, range selectors).
- `hover` — reveal tooltips, dropdown menus, hover states without clicking.
- `wait` — explicit pause in seconds; rarely needed since most tools auto-wait.
