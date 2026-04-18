User: start

GitHub Copilot: Read [](file:///d%3A/source-codes/kanbanstr/plan/plan.md#1-1), lines 1 to 100

Read [](file:///d%3A/source-codes/kanbanstr/plan/plan.md#50-50), lines 50 to 100

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/stores/kanban.ts#1-1), lines 1 to 80

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/stores/kanban.ts#80-80), lines 80 to 160

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/ndk/index.ts#1-1), lines 1 to 60

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/stores/kanban.ts#150-150), lines 150 to 400

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/ndk/index.ts#60-60), lines 60 to 120

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/stores/toast.test.ts)

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/components/Toast.test.ts)

Read [](file:///d%3A/source-codes/kanbanstr/vite.config.ts)

Read [](file:///d%3A/source-codes/kanbanstr/package.json)

Searched for regex `return {|export const kanbanStore` (`**/src/lib/stores/kanban.ts`), 18 results

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/stores/kanban.ts#1365-1365), lines 1365 to 1394

Now I have enough context. Let me create the seams and dependencies map. 

Read [](file:///d%3A/source-codes/kanbanstr/src/lib/stores/kanban.ts#50-50), lines 50 to 680

Now I have enough context to write the full map. Let me create the output file. 

Created [](file:///d%3A/source-codes/kanbanstr/plan/seams-and-dependencies.md)

Here's the testing landscape map for the card comments feature:

---

## Classes to Test

| Unit | What to verify |
|---|---|
| `loadCommentsForCard()` | filter construction, event→`CardComment` mapping, sort order, `parentId`/`parentType` logic |
| `publishComment()` | tag assembly (`a`, `e card`, `e reply`, `p`), correct delegation to `ndkInstance.publishEvent` |
| `CardDetails.svelte` (comments section) | loading state, empty state, comment rendering, reply toggle, post actions |

---

## Seams

**Seam 1 — `init(ndk)` closure injection (strong ✅)**  
`createKanbanStore` captures `ndk` as a closure variable set via `init()`. Tests can call `createKanbanStore()` directly and inject a mock NDK — covering `ndk.fetchEvents` and `new NDKEvent(ndk)`. This is exactly how toast.test.ts handles `createToastStore`.

**Seam 2 — `ndkInstance` module import (weak ⚠️)**  
`publishComment` will call `ndkInstance.publishEvent()`. Because `ndkInstance` is a module-level singleton, it requires `vi.mock('../ndk', ...)` to intercept — standard Vitest module mocking.

**Seam 3 — Svelte component props (strong ✅)**  
`@testing-library/svelte` injects `boardPubkey`, `card`, `readOnly`, `currentUser` at render time. Store calls are intercepted via `vi.mock` on the store module.

---

## Key Obstacles

- `ndk.fetchEvents` makes real relay calls → mock via Seam 1
- `ndkInstance.publishEvent` is a singleton → mock via `vi.mock`
- `createKanbanStore` is **not currently exported** — the only source change needed to unlock unit testing is adding `export` to that factory function

---

## One Thing Blocking Tests

`createKanbanStore` must be exported from kanban.ts so tests can call it directly (same pattern as `createToastStore` in `toast.ts`).

The full map is saved at seams-and-dependencies.md. Use the **break** agent next to export the factory and set up the dependency mocking strategy.

Made changes.