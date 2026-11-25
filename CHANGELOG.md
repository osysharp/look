# Osysharp.Look changelog

## 0.15.0
- `ConsoleBarChart` / `ConsoleLineChart` (and `ConsoleChartCard(points:)`) show a readout of each point (PLATFORM-709):
  pointing at a bar or point shows a crosshair and a card with its label and each series' value after its name
  ("24 Nov", "CPU 3.2 h"); the plot is one keyboard stop (arrows, Home, End, Escape), and a live region says the same
  words ("24 Nov, CPU 3.2 h"); a tap shows the tapped point, not the first. New parameters: `names` (the series' names
  the readout says) and `per` (what one point covers, "day": the stop is then "<title> by day"). A bar is a list item
  named by its words. A gap reads "not measured".
- Two units: `"ms"` (a duration, said in ms, s, min or h, the axis in round steps of the largest it reaches: 19.5 h
  gives 0–20 h) and `"bytes"` (binary: B, KiB, MiB, GiB, TiB). `ConsoleChartCard` takes `height`.
- Tests: `tests/BarChart.test.osy`, page `/dev/bar-chart`.

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
