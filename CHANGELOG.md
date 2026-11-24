# Osysharp.Look changelog

## 0.14.0
- `MoreMenu(console: true, wide:)`: the trigger is the kit's own `ConsoleButton`, so a console header's More matches its
  `ConsoleButton` neighbours in height, line and focus ring (naming as the `console` flag on the dropdowns). Menu 38 below.
  Backdrop, Escape and item-choice closing are unchanged. Tests: `tests/MoreMenuTrigger.test.osy`, page
  `/dev/controls/more-menu-console`.

## 0.13.1
- `MoreMenu`: an open menu now closes on a press anywhere outside it, through `onToggle`, so the page's `open` stays in step.
  It draws the kit's `Backdrop`, the mechanism `LookSelect`, `LookDate` and the shell menus use. Escape (keyboard back on the
  button) and choosing an item (the page's action sets `open = false`) close it as before; all three are tested.
- The trigger is, and stays, a `LookButton` (outlined): a test holds its height equal to a neighbouring outlined button
  closed, and open and focused. Tests: `tests/MoreMenuTrigger.test.osy`, page `/dev/controls/more-menu`.
