---
name: productnow-context-engine
description: Uses ProductNow, the company brain for people and AI agents, to search shared organizational context, remember facts and decisions, persist specs as documents, and coordinate reviews. Use when the user asks about product context, org knowledge, decisions, specs, RFCs, PRDs, says "remember this" or "save this", or wants to create or update ProductNow documents.
---

# ProductNow company brain

Treat ProductNow as the team's shared source of truth for its product, customers, and team. This chat is ephemeral. Recall from ProductNow before answering, cite what you used, and remember what should outlive the session.

## Connect

MCP tools are provided by this plugin (`https://api.productnow-prod.com/mcp`, OAuth). If a tool is missing, the user may still need to complete browser sign-in.

## When to use which tool

| Situation | What to do |
|---|---|
| Factual question about the org, product, customers, or team | `search` → answer from excerpts and cite title + URL; `fetch` only if excerpts fall short |
| Narrow search to a named folder | `search` with `resultType: "folders"` (or `fetch_folder`) → pass `folderId` to `search` |
| Narrow by author or date range | `search` with `creatorNames` and/or `updatedAfter` / `updatedBefore` |
| List docs matching filters, no topic | `search` with filters only (no `query`) |
| How ProductNow itself works | `search` with `searchScope: "product_help"` |
| "Remember this" / "save this" / "note that…" | `remember` with the content, then stop |
| Read a doc you have an id for | `fetch` (latest draft), or `list_document_versions` → `fetch` with `documentVersionId` |
| Browse the warehouse | `fetch_folder` (root, one folder, or `deep: true` for a tree) |
| New spec, RFC, PRD, or decision the user outlined | `search` → `fetch_folder` for placement → `create_document` → `get_status` → `fetch` |
| Change a doc the user named | `search` / `fetch` → `edit_document` → `get_status` until `idle` → `fetch` to verify |
| User references a local file or plan | Read the file; pass contents verbatim as `context` in `create_document` |
| Snapshot, team review, or publish | `update_document_status` with `snapshot`, `review`, or `publish` |
| Teammate comments | `fetch` — threads and messages are included |
| Leave feedback on a review doc | `comment_on_document` |
| Reply in a thread | `reply_to_thread` with a `threadId` from `fetch` |
| Organize work | `move`, `rename`, `archive`, `create_folder` |
| Shareable knowledge pack | `curate_knowledge_pack` when advertised |
| Embed image or video | `upload_media` |

## Workflows

### Grounded answers

1. Call `search` with the user's question. Always pass `query` when they name a topic, even if filters are also present.
2. Resolve folder names with `search` (`resultType: "folders"`) or `fetch_folder` and pass `folderId`. Use `creatorNames` (`"me"` is the calling user) and `updatedAfter` / `updatedBefore` (`YYYY-MM-DD`, UTC) when asked. Compute relative ranges yourself.
3. Omit `query` only when they ask which documents match creator or date filters (listing mode: no excerpts).
4. Answer from the excerpts and cite the document titles and URLs you rely on. If the evidence is insufficient or ProductNow is unavailable, say so.
5. Call `fetch` only when excerpts are genuinely insufficient.

### Remember

1. On "remember this", "save this", or "note that…", call `remember` with the fact as `content`, verbatim and self-contained (who, what, when). Add `hint` only if the user named a document, folder, or topic.
2. Stop. Do not search, fetch, choose a folder, or poll. `success` means it is saved.
3. Tell the user it is remembered.

### Persist and iterate (named or new document)

1. `search` before creating anything new.
2. Take `folderId` from the closest related search result, or walk `fetch_folder` from the root. Use the root only when the user asked. `create_folder` under the closest existing folder only when nothing fits.
3. `create_document` with `name`, a full `prompt` outline (numbered sections), verbatim `context`, and `folderId`; or `fetch` the existing doc and then `edit_document` with instructions for its editing agent (name the section, say exactly what to add/replace/remove, quote wording that must survive). The agent cannot see this chat.
4. Poll `get_status` until `status` is `idle`.
5. `fetch` to verify the saved content before reporting success.
6. `update_document_status` (`snapshot` / `review` / `publish`) only when the user asks.

### Team coordination

1. Canonical content: `list_document_versions` with status `PUBLISHED`, then `fetch` that version.
2. Discussion: `fetch` returns every comment thread and message; read them before proposing changes to a review doc.
3. `comment_on_document` with `quotedText` to anchor feedback to specific content.
4. `reply_to_thread` to continue a conversation.

## Anti-patterns

- Do not answer org questions from general knowledge or client memory — `search` first.
- Do not keep specs or decisions only in chat.
- Do not search, fetch, or pick a folder before `remember`, and do not poll after it.
- Do not create duplicate docs — `search` first.
- Do not call `fetch` for every search hit.
- Do not drop `query` just because filters are present.
- Do not write the new document text into `edit_document`'s `message` — write instructions for the editing agent.
- Do not report a write as done before `get_status` is `idle` and `fetch` confirms it.
- Do not pass `DRAFT` / `REVIEW` / `PUBLISHED` to `update_document_status` — it takes `snapshot`, `review`, `publish`.
- Do not use `rename`, `move`, `comment_on_document`, or `reply_to_thread` to change content — only `edit_document` does.
- Do not invent ids — use `documentId`, `folderId`, `documentVersionId`, and `threadId` from earlier tool calls.
- Do not write without the user asking — writes are shared and visible to others.

## Chat vs ProductNow

- **Chat:** ephemeral reasoning, clarifying questions, local file edits
- **ProductNow:** anything the team should see, search, comment on, or act on later
