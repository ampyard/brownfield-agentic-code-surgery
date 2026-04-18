# PR #32 — Card Comments Review

## Summary of changes
The PR adds:
- `CardComment` interface + `loadCommentsForCard` + `publishComment` to `kanban.ts`
- Comments UI section in `CardDetails.svelte`
- Unit tests for both the store functions and the component
- `NIP-100.md` documentation update
- `plan/plan.md`, `plan/seams-and-dependencies.md`, and `card-comments.md` committed to the repo

---

## Medium Issues

**1. Silent error handling in the component**

`handlePostComment` and `handlePostReply` log to the console but show no user-visible feedback. The rest of `CardDetails.svelte` uses `toastStore.addToast(...)` for user-facing errors. These handlers should do the same:

```ts
async function handlePostComment() {
    ...
    } catch (e) {
        console.error('Failed to post comment:', e); // no toast
    }
}
```

**2. `card.id` is the board d-tag for boards but the event hash for cards**

The PR correctly uses `card.id` as `cardEventId` in the `e` tag. For cards, `card.id` is set to `event.id` (the Nostr event hash) at lines 281, 403, and 542 of `kanban.ts`. This is correct — but the mapping is inconsistent with boards (line 166: `id: dTag ? dTag[1] : event.id`). The implementation is correct for comments; worth noting for future readers.

**3. Incomplete `mockKanbanState` in `CardDetails.test.ts`**

The mock provides only `{ myBoards: [], maintainingBoards: [], cards: new Map() }`, missing `boards`, `loading`, `currentUser`, and `error`. The component likely accesses `$kanbanStore.boards` in other sections of the template. If the component renders those sections before the comments section, Svelte may throw a type error when iterating `undefined`. The tests may pass currently because the rendering fails silently or the parts that access `boards` are unreachable in the test environment, but the mock is fragile.

---

## Minor Issues

**4. Author display shows raw pubkey truncation**

```svelte
<span class="comment-author">{comment.pubkey.slice(0, 8)}…</span>
```

The plan specified `<UserAvatar pubkey={comment.pubkey} />` and `getUserDisplayName`. The `UserAvatar` component and `getUserDisplayName` are already imported in the component. Showing a truncated pubkey is unhelpful UX — 8 hex characters aren't meaningful to users.

**5. Plan files committed to the repo**

`plan/plan.md` and `plan/seams-and-dependencies.md` are AI agent working artefacts. They add noise to the project history and don't belong in the source tree. `card-comments.md` (the feature requirements) is also committed and would normally live in an issue tracker, not source control.

**6. Missing newline at end of `CardDetails.svelte`**

The patch explicitly shows `\ No newline at end of file` at the bottom of the file. This is a cosmetic but persistent diff noise issue.

**7. No textarea `maxlength`**

Neither the comment nor reply textarea has a `maxlength` attribute. A very long comment would be published as-is. A practical cap (e.g., 2000 characters) should be enforced in both the UI and in `publishComment`.

**8. No guard in `loadCommentsForCard`/`publishComment` before NDK is initialized**

Both functions use `ndk` (the closure variable) without checking `hasNDK()`. This is consistent with existing store functions, but if `loadComments()` fires before `init()` is called, the component will silently fail. The existing pattern should at least be documented, or a defensive check added.

---

## What is done well

| Aspect | Assessment |
|--------|------------|
| **Testability** | `createKanbanStore` is now exported, enabling isolated unit tests. The seam analysis is excellent and the test architecture follows the `toast.test.ts` pattern already in the repo. |
| **Tested** | `kanban.test.ts` covers all tag structure cases for `publishComment` and all mapping cases for `loadCommentsForCard`. `CardDetails.test.ts` covers render states, auth gates, post/reply flows, and cancel. Good coverage. |
| **Maintainability** | Purely additive — no existing function modified. `CardComment` interface is clean. Functions have single responsibilities. CSS follows the existing `.section`/`background: #f5f5f5` style. |
| **Documented** | `NIP-100.md` updated correctly. Plan docs are thorough even if they shouldn't be committed. |
| **Provably correct** | Tag assembly logic for `a`, `e`, and `p` is correct. Sort order by `created_at` is correct. `parentType` derivation from the `reply` marker is correct. `card.id` correctly references the Nostr event hash. The one-level nesting model is implemented faithfully. |

