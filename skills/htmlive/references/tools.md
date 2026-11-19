# HTMLive MCP tools

These are the tools the server registers. Names and arguments match the implementation. Do not invent a preview, update, or confirm tool.

The public MCP address, when the tools are not already connected, is `https://htmlive.app/mcp`.

## Call these

`list_templates`

- Arguments: none.
- With a user token: every copy on that account that has not been deleted. `source` is `personal`. `scope` is `mine`.
- Without a token: the public official catalog. The `note` says so.
- Each item has `id`, `name`, `description`, `summary` when stored, `category`, and `source`.

`select_template`

- Argument: optional `returnUrl`.
- Pass `returnUrl` only when the client gave that exact https permalink. Do not guess. The server allowlist is empty, so nothing is stored and the page does not redirect.
- Requires a user token. Otherwise nothing is selected.
- Open result: `status` `pending`, `sessionId`, `selectionUrl`, `expiresAt`, `delivery` `poll`, `pushed` false.
- `declined` or `cancelled` means nothing was selected.

`get_template_selection`

- Arguments: `sessionId` from `select_template`. Optional `waitMs`. Omit `waitMs`. Do not pass `0`. The default holds the call open. The cap is 50 seconds.
- `status` `selected` includes `templateId` and `templateName`.
- `status` `pending` includes `selectionUrl` and `retry`. Call again immediately with the same `sessionId`.
- `cancelled` stores nothing. An expired session says it expired.
- `pushed` is false. This cannot wake a turn that has already finished.

`get_template`

- Argument: `template`, a name, a stable id from `list_templates`, or a `/templates/<id>` link on this same origin. There is no `templateUrl` argument.
- The link only identifies the template. It has no token and does not authorize the read. Do not fetch it. A different host is not this template. Without this caller's own token, or when the id is not one of this account's copies, stop. Nothing is read, and do not select another template.
- Opening that URL in a browser shows a public note. The note has no template name, guide, stylesheet, or skeleton. It does not install MCP and does not resume a chat.
- `includeCss` is ignored. The stylesheet is always `assets/style.css`.
- Result: `id`, `name`, `skillMd`, `files`. Each file is `{ path, bytes, content }`. `content` is the full text.
- Paths: `SKILL.md`, `assets/style.css`, `assets/skeleton.html`.
- An unknown name is an error and lists the current name and id pairs. Two matches are an error and nothing is selected.

`create_page`

- `html` is required, up to 2MB, full document or fragment.
- Optional `title`. Inserted only when the HTML has no `<title>`.
- Optional `template`, a name or id. Copies that template's stylesheet onto the HTML. It does not publish the template by itself.
- Result includes `shareUrl` and `editKey`. `owned` is true only for a verified token. The result does not include an email or user id.
- Missing or blank `html` does not publish.

`duplicate_template`

- Argument: `sourceId`, an official id such as `weekly`, or that official template's name.
- Requires a user token. Adds one official template into the account. The new row has a new id and the official name plus 副本.
- A personal id writes nothing. An existing copy is not overwritten. The official template is not changed.

`delete_template`

- Argument: `id`, this account's own copy. Not an official catalog id.
- Requires a user token. Result is `{ ok: true, id }` when it deletes that copy.
- Published pages stay. The official catalog stays.

## Do not call

`confirm_template_selection` is for the host's card. Arguments would be `sessionId` and `templateId`. Hosts that honor app-only visibility hide it from the model. The model path is `select_template` plus `get_template_selection`.

These names have stopped and write nothing:

- `update_page`
- `get_page`
- `rename_page`
- `duplicate_page`
- `delete_page`
- `set_page_sharing`
- `claim_page`
- `update_template`
- `update_template_notes`
- `rename_template`
- `set_template_enabled`
- `update_template_style`
- `create_template`
- `import_template`

There is no tool whose name is `preview_page`, `preview_template`, `update_html`, or `confirm_publish`.
