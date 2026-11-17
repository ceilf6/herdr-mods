# Tabs (quick close)

Herdr's own tab bar, with one change: the x on the active tab closes it at
once, even when it is the space's last tab. Herdr's shipped tab bar asks
"Close space?" in that case, because closing the last tab closes the space
and ends its terminals. This copy passes `confirmed: true`, so there is no
dialog. Everything else (tab pills with the program's icon and status, drag
to reorder, double-click to rename, right-click menu, the + menu, the Files
and Changes panel buttons, window buttons where the OS draws none) is the
shipped `tabs` mod as it is.

![Tabs (quick close)](preview.png)

## What you see

Left to right, in the 42px block at the top of the work area, on the theme's
`chrome` background with a 1px `border` line underneath, 10px side padding
(more on the left on macOS without a system title bar, to clear the window
buttons):

- One pill per tab of the active space, 29px tall, 10px side padding,
  rounded 7, 12.8px text. The active tab has the `raised` background,
  `foreground` text and a `border` outline; the others `chrome`, `muted`
  text and an invisible outline. Hover shows `hover`.
- In a pill: the program's icon at 14px when it has one (an agent's if the
  tab runs one, else its first pane's), the name (truncated at 170px; a tab
  nobody named shows the program's name, "Terminal" for a shell), " Z" after
  it when zoomed, a 12px spinning ring in `working` color while an agent
  works, a 6px dot in `blocked` or `done` color for those states, and, on
  the active tab only, a 12px x icon in `dim`.
- After the pills: a 26px "+" button, then "Files" and "Changes" buttons
  (14px icon and label, `selected` background when that panel is open),
  and on Linux and Windows the minimize, maximize and close window buttons.

## How it behaves

- **Click a pill**: selects that tab. **Drag** a pill onto another and
  release: moves it there. **Double-click**: opens the rename prompt.
  **Right-click**: the shipped `menu` mod's tab popup. **Scroll wheel**
  over the bar: previous or next tab.
- **Click the x** on the active tab: closes that tab (`tab.close {id,
  confirmed: true}`). If it was the space's last tab, the space closes with
  it and its terminals end, immediately, without the "Close space?" dialog.
  The keyboard shortcut (`cmd-w` on macOS, `ctrl-shift-w` elsewhere) and the
  right-click menu's Close still ask; they are not this mod's.
- **+**: opens the shipped `new-menu` popup (a terminal or any installed
  agent). **Files / Changes**: show that panel in the right-hand `panel`
  stack or hide it when it is already shown; the choice is remembered per
  space in the mod's local storage and restored on switching spaces.
- Window buttons appear only where the OS draws none (`window().buttons`),
  with vector icons on Windows and glyphs elsewhere.

## Built from

- `mods/tabs-quick-close/main.js`, a copy of Herdr 0.1.11's shipped
  `mods/tabs/main.js` (recorded in `mod.json` as `cloned_from`), with the
  active tab's x calling `run("tab.close", { id: c.id, confirmed: true })`
  instead of `run("tab.close", { id: c.id })`.
- `mod.json`: `{ "place": "none", "title": "Tabs" }` plus `cloned_from`. It
  replaces the `tabs` block of a layout; it does not dock on its own.
- `win-min.svg`, `win-max.svg`, `win-restore.svg`, `win-close.svg`: the
  Windows window-button icons, referenced as `mods/tabs-quick-close/...`.
- `herdr` functions: `spaces`, `panes`, `run`, `openOverlay`, `theme`,
  `window`, `shared`, `layout`, `settings`.
- Commands: `tab.close`, `tab.select`, `tab.move`, `tab.rename`,
  `tab.prev`, `tab.next`, `layout.show`, `layout.hide`, `layout.select`,
  `window.minimize`, `window.maximize`, `window.close`.
- Shipped mods it uses: `menu` (right-click popup, `props.tab: "tab"`),
  `new-menu` (the + popup), and icons from `pane` (close), `workspace-chrome`
  (spinner), `files` and `changes`.
- Shared storage: reads `workspace.panels` (which panels the welcome
  enabled). Local storage: `panels` (space id → open panel or null).
- Layout ids it expects, as in the `main` layout: `sidebar`, `panel`,
  `panel-body` (a stack with `files` and `changes` children).

## Watch out

- This removes a safety net on purpose: one click on the x of a space's
  only tab kills every terminal in that space, agents included, with no way
  back. Don't add it for someone who didn't ask for that.
- Nothing shows until a layout points its `tabs` block at this mod. Clone
  the main layout (`layout.clone {"name":"main","as":"my-main"}`) and set
  that block's `content.mod` to `tabs-quick-close`; don't edit Herdr's
  `main.json` (it is replaced on every start).
- `mod.clone` rewrote the icon paths to `mods/tabs-quick-close/`; a copy
  made by hand under another name must do the same, or the Windows window
  buttons lose their icons.
- The shortcut and the menu still confirm. For the shortcut, a `keys.json`
  binding with arguments does it: `"cmd-w": {"command": "tab.close", "args":
  {"confirmed": true}}`. Editing keys.json makes the whole file the
  person's: later Herdr versions won't add new default shortcuts to it.
- A copy of a shipped mod goes stale: this one is 0.1.11's tabs. To refresh,
  re-clone the current shipped `tabs` and re-apply the one argument.
- It runs no programs and reaches no network; it reads nothing outside its
  folder besides the shipped icons above.

## Done when

- `herdr-app check ~/.config/herdr-app/mods/tabs-quick-close` passes and no
  `.error` file appears in the home.
- With a layout's tabs block set to `tabs-quick-close`, the bar looks
  exactly like Herdr's.
- In a space with one tab, clicking the active tab's x closes the space at
  once: no "Close space?" dialog, the space leaves the sidebar.
- In a space with two tabs, the x closes only that tab, as before.
- Click, drag, double-click, right-click, +, Files and Changes behave as in
  Herdr's tab bar.
