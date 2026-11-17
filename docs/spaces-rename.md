# Spaces (double-click to rename)

Herdr's own Spaces sidebar list, with one addition: double-clicking a space
opens the same "rename space" prompt the tab bar opens when you double-click a
tab. Everything else (colored avatars, worktrees nested under their checkout,
machine groups, git branch and ahead/behind after the name, drag to reorder,
right-click menu) is the shipped `spaces` mod as it is.

![Spaces (double-click to rename)](preview.png)

## What you see

Left to right, top to bottom, inside the sidebar block it replaces:

- A 31px header row: "Spaces" in 11.5px dim text at the left (hidden when the
  sidebar is narrower than 160px), a 26px square "+" button at the right that
  makes a new space.
- The list, 8px side padding, scrollable, always showing its scrollbar.
- One row per space, 30px tall (28px for a worktree row), 8px side padding,
  rounded 8. The active space has the theme's `selected` background; others
  get `hover` on hover.
- In a row: an 18px rounded square avatar in the space's stored color (one of
  six strong hues, `#22a99a #e3814f #5b8def #8b7cf6 #3fae6a #d9649a`) holding
  the first letter of the name in bold white 10.5px; the name in `foreground`,
  13px, truncated at 150px; then right-aligned in 11.5px dim text only what is
  unusual about its git: repo name when it differs from the space name, branch
  when it is not master/main and not the space name, `↑n` in green, `↓n` in
  red.
- A worktree space sits under its main checkout, indented 22px with a branch
  icon in place of the avatar.
- When more than this computer is known, a 26px machine header per machine
  (monitor icon for this computer, server icon for others, name, a green or
  dim 6px dot for connected or not, a chevron that folds the group, a "+" to
  make a space on that machine). An empty connected group shows "New space
  there".
- Narrower than 160px, rows collapse to their 1-based number and a status
  glyph (`●` for blocked or done, `·` otherwise).

## How it behaves

- **Click** a row: selects that space (`space.select`).
- **Double-click** a row: opens the rename prompt for that space
  (`space.rename {id}` with no name, so the app asks). The first click of the
  pair selects the space as usual, so the prompt opens on the selected space.
  The field starts with the name you gave the space, empty when its name is
  automatic. Enter saves; an empty name goes back to Herdr's automatic name
  (its folder).
  Not in the collapsed numeric rail.
- **Right-click** a row: the shipped `menu` mod's space popup (rename, close,
  move and so on).
- **Drag** a row and release on another row of the same machine: moves it
  there (`space.move`). While dragging, the source row is at half opacity and
  a 2px accent line marks the drop edge. Releasing elsewhere does nothing.
- Agent status is not shown here (it lives in the agents list); the collapsed
  rail shows the space's own status glyph.
- Git is re-read when a space is new or its first terminal changed folder,
  when a command or agent turn ended in it, when its machine comes back, and
  every 60s for visible repos while the window is in front. Local repos are
  also watched for changes made outside Herdr.

## Built from

- `mods/spaces-rename/main.js`, a copy of Herdr 0.1.11's shipped
  `mods/spaces/main.js` (recorded in `mod.json` as `cloned_from`), plus one
  line in `row()`'s left `on_mouse_down`:
  `if (e.click_count === 2) run("space.rename", { id: w.id });`
  The handler takes the event (`(e) => …`) to read `click_count`.
- `mod.json`: `{ "place": "none", "title": "Spaces" }` plus `cloned_from`.
  It is meant to replace the `spaces` block in a layout, not to appear on its
  own.
- `herdr` functions: `spaces`, `machines`, `panes`, `run`, `openOverlay`,
  `exec`, `hc`, `theme`, `shared`, `setShared`, `watchFolder`, `window`,
  `layout`.
- Commands: `space.select`, `space.rename`, `space.move`, `space.new`.
- Shipped mods it uses: `menu` (right-click popup, `props.tab: "space"`),
  `workspace-chrome` for its icons (`plus`, `branch`, `monitor`, `server`,
  `chevron-right`, `chevron-down` SVGs).
- Programs it runs: `git status --porcelain=v2 --branch`, `git rev-parse
  --show-toplevel`, `git rev-parse --git-common-dir` in each space's folder
  (on that space's machine); `hc machine ls --json` every 15s.
- Shared storage: reads and writes `space.colors` (space id → color name).
  Local storage: `machines.folded`.

## Watch out

- The mod does nothing until a layout points a block at it. Clone the main
  layout (`layout.clone {"name":"main","as":"my-main"}`) and change the
  `spaces` block's `content.mod` to `spaces-rename`; don't edit Herdr's
  `main.json` (it is replaced on every start).
- Double-click and drag share `on_mouse_down`. Don't move the rename into
  `on_click`: the row has no `on_click`, and adding one would fight the
  mouse-up logic that decides between select and move.
- A copy of a shipped mod goes stale: this one is 0.1.11's spaces. Later
  Herdr versions add to the shipped mod (0.1.13 added reconnecting a machine
  whose core was rebuilt and a different rail layout). To refresh, re-clone
  the current shipped `spaces` and re-apply the one line.
- `runs programs` and `reaches the network` only in the sense the shipped mod
  does: git in your repos (possibly over ssh on another machine) and `hc`.
  Nothing else; it reads no files outside its folder itself.

## Done when

- `herdr-app check ~/.config/herdr-app/mods/spaces-rename` passes and no
  `.error` file appears in the home.
- With a layout block set to `spaces-rename`, the sidebar looks exactly like
  Herdr's Spaces list.
- Double-clicking a space opens a "Rename space" prompt (its field holds the
  name you gave, or is empty for an automatic name); Enter renames it, Escape
  leaves it alone.
- A single click still selects, right-click still opens the menu, and dragging
  a row onto another still reorders.
