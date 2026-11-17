# Browser tooling (Claude in Chrome)

Use only for the browser shortcuts and account actions. Work in the user's signed-in Chrome.

- Open your own tabs and close them when done. Keep the Goodreads tab on its page while data lives in page globals: globals vanish when a tab navigates or reloads. Use a second tab for Libby. Copy important data out of the page (into chat or a file) as soon as you have it.
- Tool output that looks like cookies or query strings is blocked ("Cookie/query string data"). Return plain text separated by ` | `, avoid `=`, `=>` and key=value shapes, and chunk to under about 1500 characters.
- Tabs can be replaced mid-run (user closes one, tool family changes). Re-read tab context and use the new tab id; ids from earlier sessions are invalid.
- Read element positions from a fresh screenshot every time. Zoom regions must lie inside the current viewport.
- `navigate` cannot run inside a batched action list.
- If the browser tools are missing, fall back to file intake and hand the user the steps and links.
