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
  Every tick is said in the unit the largest value reaches, the one the ticks are rounded in: 59 min reads 0–60 min,
  not 0, 0.33, 0.67, 1 h (`ChartScaleColumn(most:)`). The Charts kit's `Axis(unit:)` (1.12.0) follows the same rule.
- One readout at a time: the keyboard's (or a tap's) card gives way while the pointer is over the plot. Its band now
  declares a `variants` block, without which the rule hiding that card matched nothing and a click then a point at
  another bar showed two cards.
- `ConsoleBarPoint.Said`: what the readout calls a point when its axis label is too short to stand alone. By the hour the
  axis says "14:00" and the readout "24 Nov, 14:00". Empty, the readout says `Label`, as before.
- `ConsoleBarChart`: when the bars are too narrow for every value (a month of days, a day of hours), the latest bar's
  value stands in the band above the plot, from its bar's edge nearer the middle inward. It no longer runs past the
  plot and the card, nor lies over the taller bars beside it, and with several series it says each value once
  ("1,234 · 3"). In a test it is named "<label> latest value".
- The time axis (`ChartTimeAxis`, both charts): the last label ends flush with the plot's right edge and the first with
  its left, and the others are centred under their columns. Each sat at the left of its box before.
- **Renamed:** the chart series tokens are `Colors.LookSeries1` … `LookSeries7` (were `Series1` … `Series7`). The Charts
  kit declares `Series1` … `Series7` with its own values, so an app importing both kits had two owners for one leaf and a
  Look chart could draw in the Charts kit's colours. An app that named `Colors.SeriesN` for the Look's colour names
  `Colors.LookSeriesN` now; one that declared the seven leaves only to settle that collision can drop them.
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
