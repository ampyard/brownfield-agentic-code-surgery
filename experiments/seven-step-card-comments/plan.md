# Plan: Card Comments Feature

## 1. Feature Description

Add threaded comment support visible only inside the card detail view. Comments are stored as Nostr events (kind `30303`) and loaded when the card detail is opened. Users can post top-level comments on a card and reply to existing comments. No change to the board/card summary views.

---

## 2. Change Points

### `src/lib/stores/kanban.ts`

| What | Detail |
|------|--------|
| New `CardComment` interface | Fields: `id`, `pubkey`, `content`, `created_at`, `parentId` (from `e` tag), `isReply` |
| New `loadComments(boardPubkey, cardDTag)` | Fetches `kinds: [30303]`, `#a: ['30302:<boardPubkey>:<cardDTag>']`, parses events into `CardComment[]`, builds thread tree |
| New `addComment(cardEventId, boardPubkey, cardDTag, content, parentCommentEventId?)` | Publishes a kind `30303` event with `content`, `['a', '30302:<boardPubkey>:<cardDTag>']`, and `['e', '<cardEventId>', 'card']` for top-level or `['e', '<parentCommentEventId>', 'reply']` for replies |
| Export both functions via the `kanbanStore` return object | — |

### `src/lib/components/CardDetails.svelte`

| What | Detail |
|------|--------|
| New script variables | `comments: CardComment[]`, `loadingComments`, `newCommentText`, `replyingToId: string \| null`, `replyText`, `isSubmittingComment` |
| `onMount` addition | Call `kanbanStore.loadComments(boardPubkey, card.dTag)` and store result |
| New `<div class="section">` block | Rendered after the Clone section, before the `<footer>`. Shows the threaded comment list and the new-comment input |
| Thread rendering | Top-level comments first; each top-level comment has a Reply button that shows an inline reply input; replies indented beneath parent |
| Add-comment form | Textarea + Submit button; disabled when `readOnly` or not logged in |

---

## 3. Event Schema (kind 30303)

```
{
  kind: 30303,
  content: "<comment text>",
  tags: [
    ['a', '30302:<boardPubkey>:<cardDTag>'],   // card reference
    ['e', '<cardEventId>', 'card']             // for top-level comments
    // OR
    ['e', '<parentCommentEventId>', 'reply']  // for replies
  ]
}
```

Loading filter:
```
{ kinds: [30303], '#a': ['30302:<boardPubkey>:<cardDTag>'] }
```

Threading logic: group by `e` tag — if the `e` tag has marker `'card'`, it is a top-level comment; if it has marker `'reply'`, it is a reply to the event with that id.

---

## 4. Impact Analysis

- **No existing code is modified in a breaking way.** All changes are additive.
- `kanbanStore` gains two new exported functions; the rest of the store is untouched.
- `CardDetails.svelte` gains a new section at the bottom; all existing sections are untouched.
- Board list, board view, and card summary (Card.svelte) are **not touched**.
- No migration needed — kind 30303 is new and independent.

---

## 5. Risk Assessment

| Risk | Likelihood | Mitigation |
|------|-----------|------------|
| User publishes a comment without being logged in | Low | Guard `addComment` with `currentUser` check (same pattern used in card save) |
| Comments from unknown authors showing up | Low | No author filtering needed — comments are public by design |
| Large number of comments causing performance issues | Low | No pagination required for v1; relay limits will naturally cap results |
| Reply threading broken by missing parent events | Medium | If parent id not found, render comment as top-level (graceful fallback) |

---

## 6. Success Criteria

- Saving a new comment publishes a kind `30303` event visible in relay explorers with the correct `a` and `e` tags.
- Replying to a comment publishes a kind `30303` event with `['e', '<parentId>', 'reply']`.
- Opening card details loads and displays existing comments threaded correctly.
- Closing and reopening card details re-fetches comments.
- Board view and card summary are unchanged.
- Read-only / logged-out users can see comments but cannot post.

---

## 7. Implementation Order

1. Add `CardComment` interface and `loadComments` / `addComment` to `kanban.ts`
2. Add comments section (display + input) to `CardDetails.svelte`
3. Wire up load on `onMount` and submit on button click
4. Manual test: post comment → reply → reopen card details
