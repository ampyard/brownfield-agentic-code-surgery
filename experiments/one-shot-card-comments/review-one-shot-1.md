# PR #33 Review — Card Comments Feature

## Issues Found

---

**1. Critical — Publish errors silently swallowed** (`CardComments.svelte`)

Both `submitComment` and `submitReply` use `try/finally` without a `catch`. If `publishComment` throws (e.g. network error), the user sees nothing. The rest of the app uses `toastStore` to surface errors. A `catch` block with `toastStore.addToast(...)` is needed.

---

**2. Critical — Memory leak: `unsubKanban` never cleaned up** (`CardComments.svelte`)

```ts
const unsubKanban = kanbanStore.subscribe(state => {
    comments = state.comments.get(card.id) || [];
});
```

The unsubscribe function is never called — there's no `onDestroy(() => unsubKanban())`. Every time a card detail panel opens and closes, a subscription accumulates in the store. Compare with `CardDetails.svelte` which correctly calls `unsubKanban()` inside `onDestroy`.

---

**3. Major — No tests added**

The two new store functions (`loadCommentsForCard`, `publishComment`) and the component helpers (`getTopLevelComments`, `getReplies`) are entirely untested, despite the project having existing test files.

---

**4. Minor — `card.id` used as comment Map key is unstable** (`kanban.ts`)

Cards are replaceable events — when a card is edited and re-published, its `id` changes. The comment Map entry under the old `id` becomes stale and new comments go to a different key. Using `card.dTag` (stable across versions) as the key is more robust and consistent with how the rest of the store deduplicates cards.

---

**5. Minor — `isSubmitting` shared across all forms** (`CardComments.svelte`)

A single flag locks both the top-level comment input and all reply inputs simultaneously. This is likely unintentional.

---

**6. Minor — Raw hex pubkey shown instead of display name** (`CardComments.svelte`)

```svelte
<span class="comment-pubkey">{comment.pubkey.slice(0, 8)}…</span>
```

Everywhere else in the app, `getUserDisplayName()` is used to resolve NIP-05 names or display npubs. This is inconsistent with the established UX pattern.

---

## Summary Table

| Aspect | Status | Key Issue |
|---|---|---|
| Tested | ❌ | No tests added |
| Testable | ⚠️ | Consistent with existing patterns; helpers not exported |
| Provably Correct | ❌ | Publish errors silently swallowed; unstable Map key |
| Maintainable | ❌ | `unsubKanban` subscription leak; inconsistent pubkey display |
| Documented | ✅ | NIP-100.md updated correctly |
