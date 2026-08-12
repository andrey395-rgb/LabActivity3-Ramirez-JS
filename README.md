# Lab Activity 3 — Kanban Board

**Week:** 3 (JavaScript Engine & Dynamic DOM)  
**Skills:** DOM create/update/remove, event delegation, in-memory state  

## Goal

Build an interactive Kanban board with three columns (**To Do**, **Doing**, **Done**).  
Users can **add**, **move**, **edit**, and **delete** task cards — all in the browser.  
No database and no page reloads.

## Setup

1. Open `lab_3_kanban_board/index.html` in a browser (`file://` is fine).
2. Open DevTools → Console. Stub warnings mean `app.js` loaded.
3. Implement the `// TODO` sections in `app.js` only (HTML/CSS are ready).

## State shape

```js
state.tasks = [
  { id: "t1", title: "Read the lab README", status: "todo" }, // "todo" | "doing" | "done"
];
```

Always change `state` first, then call `render()` so the UI matches your data.

## DOM hooks

| Selector | Role |
|----------|------|
| `#task-form`, `#task-title` | Add a new card |
| `#board` | Parent for click delegation |
| `[data-column-body="todo\|doing\|done"]` | Where cards are appended |
| `[data-count="…"]` | Column count badges |
| `button[data-action]` | `move`, `edit`, or `delete` |
| `data-id`, `data-status` | Task id and target column for moves |

## Steps

### 1. Implement `render()`

1. Clear each `[data-column-body]`.
2. For every task in `state.tasks`, build a `.card`:
   - Title in an `<h3>`
   - Action buttons (Move → To Do / Doing / Done, Edit, Delete) with the `data-*` attributes listed in `app.js`
3. Append each card into the column that matches `task.status`.
4. If a column has no cards, show `<p class="empty">No tasks</p>`.
5. Update each `[data-count]` to the number of tasks in that status.
6. Refresh the page — seed cards (`t1`, `t2`, `t3`) should appear in the right columns.

### 2. Add tasks

1. In the form submit TODO: `addTask(title)`, clear the input, `render()`.
2. Implement `addTask`: push `{ id: uid(), title, status: "todo" }`.
3. Check: new cards land in **To Do**.

### 3. Move tasks (event delegation)

1. Finish the `#board` click handler — when `action === "move"`, call `moveTask(id, status)` then `render()`.
2. Implement `moveTask`: find the task, set `status` if it is `"todo"`, `"doing"`, or `"done"`.
3. Check: a card can move between all three columns; counts update.

### 4. Edit titles

1. On `action === "edit"`, `prompt()` for a new title (prefill with the current title).
2. If the user cancels or enters only spaces, do nothing.
3. Implement `editTask` to save a non-empty title.
4. Check: after edit, moving the card still shows the new title (state was updated, not only the DOM once).

### 5. Delete with confirmation

1. Implement `deleteTask`: `confirm("Delete this task?")`, then filter the task out.
2. Call it from the board handler when `action === "delete"`, then `render()`.
3. Check: Cancel keeps the card; OK removes it and may show the empty message.

## Tips & tricks

### Work in a safe order

1. Get `render()` working with the seed data first — you should see three cards before you touch add/move/edit/delete.
2. Then wire **add**, then **move**, then **edit**, then **delete**. Each feature is easier to test when the board already paints correctly.
3. After every state change, call `render()`. If the UI looks wrong, log `state.tasks` in the Console and compare it to what you see on screen.

### Debugging in DevTools

- **Console:** temporarily add `console.log("tasks", state.tasks)` at the end of `render()` to see your data after each action.
- **Elements panel:** inspect a card button and confirm `data-action`, `data-id`, and `data-status` are set the way you expect.
- **Breakpoints:** click the line number next to your board click handler, then click a card button — step through to see which `action` fired.
- If nothing happens on click, you probably forgot `render()` after updating state, or the button is missing `data-action`.

### DOM patterns that help

- Use **one** click listener on `#board` (already started). Do **not** attach a new listener to every button inside `render()` — that causes duplicate handlers and hard-to-debug bugs.
- Clear a column with `columnBody.innerHTML = ""` at the start of each full re-render, then rebuild cards from `state`.
- Prefer `textContent` for titles (not `innerHTML`) so characters like `<` don’t break markup.
- Build cards with `document.createElement("div")`, set `className = "card"`, then `appendChild` — easier to read than a giant HTML string.
- For counts: `state.tasks.filter((t) => t.status === "todo").length` (repeat for `doing` / `done`).

### Event delegation cheatsheet

```js
const btn = e.target.closest("button[data-action]");
if (!btn) return; // clicked empty column space, not a button
const { action, id, status } = btn.dataset;
```

- `e.target` might be the text inside the button; `closest(...)` finds the actual button.
- `btn.dataset.id` reads `data-id`; `btn.dataset.status` reads `data-status`.

### Common mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Page reloads when you add a task | Form submitted normally | Keep `e.preventDefault()` on submit |
| Cards disappear after add | Forgot `render()` or cleared state by mistake | Call `render()` after `addTask` |
| Click does nothing | Missing `data-action` / wrong listener target | Inspect the button; click must bubble to `#board` |
| Edit works once, then title resets | You changed the DOM only, not `state` | Update `state.tasks`, then `render()` |
| Duplicate cards on every click | Listeners added inside `render()` | Only use the one `#board` listener |
| Delete removes the wrong card | Wrong `id` on the button | Set `data-id="{task.id}"` when creating the button |
| Empty column looks broken | No empty state | Append `<p class="empty">No tasks</p>` when count is 0 |

### Small quality upgrades (optional)

- Hide or skip the “Move → …” button for the column the card is **already** in (less clutter).
- Trim titles with `.trim()` before saving; ignore empty edits.
- After a successful add, focus the input again: `input.focus()`.
- Keep move labels short on small screens, e.g. `→ To Do`, `→ Doing`, `→ Done`.

### Mental model (remember this)

```
User action → update state.tasks → render() → DOM matches state
```

Never treat the DOM as the source of truth. The array is the source of truth; `render()` is just the painter.

## Acceptance checklist

- [ ] Seed tasks render into the correct columns  
- [ ] Can add a task (appears in To Do)  
- [ ] Can move a task between To Do / Doing / Done  
- [ ] Column counts stay correct  
- [ ] Can edit a title; it survives re-render / move  
- [ ] Delete asks for confirmation and removes the task  
- [ ] Empty columns show “No tasks”  

## Submission

Follow your instructor’s instructions (usually a GitHub repo URL or zipped `lab_3_kanban_board` folder).
