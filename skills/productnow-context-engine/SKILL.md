---
name: productnow-context-engine
description: Uses ProductNow, the context engine for people and AI agents, to search shared organizational context, persist specs and decisions as documents, and coordinate reviews. Use when the user asks about product context, org knowledge, decisions, specs, RFCs, PRDs, or wants to create or update ProductNow documents.
---

# ProductNow context engine

Treat ProductNow as the team's shared source of truth. This chat is ephemeral. Ground answers in ProductNow evidence, then persist what should outlive the session.

## Connect

MCP tools are provided by this plugin (`https://api.productnow-prod.com/mcp`, OAuth). If a tool is missing, the user may still need to complete browser sign-in.

## When to use which tool

| Situation | What to do |
|---|---|
| Factual question about the org | `search_knowledge_warehouse` → cite excerpts; `get_document` only if excerpts are insufficient |
| Narrow search to a named folder | `search_folders` → pass `folderId` as `destinationFolderId` |
| Narrow by author or date range | `search_knowledge_warehouse` with `creatorNames` and/or `updatedAfter` / `updatedBefore` |
| List docs matching filters, no topic | `search_knowledge_warehouse` with filters only (no `query`) |
| How ProductNow itself works | `search_knowledge_warehouse` with `searchScope: "product_help"` |
| New spec, RFC, PRD, or decision | Search first → if none exists, `create_document` |
| Resume prior work | Search → `get_document` |
| User references a local file or plan | Read the file; pass contents as `context` in `create_document` |
| Iterate on a draft | `post_document_chat_message` (optionally `switch_document_chat_edit_mode`) |
| Ready for team review | `move_document_to_review` |
| Teammate comments | `list_document_versions` → `get_document_threads` → `get_thread_messages` |
| Leave feedback on a review doc | `create_document_comment` (review status only) |
| Reply in a thread | `reply_to_thread` |
| Prototype / user feedback | `get_document_feedback` |
| Organize work | `search_folders` / `list_folders` → `create_folder` |
| Shareable knowledge pack | `curate_knowledge_pack` when advertised |
| Embed prototype or media | `import_prototype` or `upload_media` |

## Workflows

### Grounded answers

1. Call `search_knowledge_warehouse` with the user's question. Always pass `query` when they name a topic, even if filters are also present.
2. Resolve folder names with `search_folders`. Use `creatorNames` (`"me"` is the calling user) and `updatedAfter` / `updatedBefore` (`YYYY-MM-DD`, UTC) when asked. Compute relative ranges yourself.
3. Omit `query` only when they ask which documents match creator or date filters (listing mode: no excerpts).
4. Treat returned sources as relevant; cite documents you rely on.
5. Call `get_document` only when excerpts are genuinely insufficient.

### Persist and iterate

1. Search before creating anything new.
2. `create_document` with `name`, a full `prompt` outline (numbered sections), supporting `context`, and `folderId` from folder search/list.
3. `get_document` before editing. Do not assume chat memory is current.
4. Iterate with `post_document_chat_message`. Use automatic edit mode only when the user wants fast iteration.
5. `move_document_to_review` when the draft is ready for teammates.

### Team coordination

1. Canonical content: `list_document_versions` with status `published`.
2. Discussion: `get_document_threads` and `get_thread_messages` before proposing changes to a review doc.
3. `create_document_comment` with `quotedText` to anchor feedback (review versions only).
4. `reply_to_thread` to continue a conversation.
5. `get_document_feedback` before rewriting from prototype/user input.

## Anti-patterns

- Do not keep specs or decisions only in chat.
- Do not create duplicate docs — search first.
- Do not call `get_document` for every search hit.
- Do not drop `query` just because filters are present.
- Do not edit review docs without reading threads.
- Do not comment on draft versions.
- Do not guess document IDs — search or list versions first.

## Chat vs ProductNow

- **Chat:** ephemeral reasoning, clarifying questions, local file edits
- **ProductNow:** anything the team should see, search, comment on, or act on later
