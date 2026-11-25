# The Osyrin look — for builders of pages

`osyrin/kits/look` (`Osysharp.Look`, version in `package.osy`) is Osyrin's own look, drawn from the v3 boards
(`designs/factory-ui-v3/` in the platform's operations docs) and the console boards (`designs/console-B/` beside them). Every Osyrin
page — Admin, the org-policy pages, Foundry — is built from it. `use Osysharp.Look@0;` in the manifest, `using
Osysharp.Look;` in the file. The gallery is `/dev/components` in `osyrin/kits/look/tests`, and the page replicas are
`/dev/page-parts/plan`, `/release`, `/slice` and `/effective` (ProductPlan, Release, Slice and canvas4's
PolicyFloorEffective drawn with the package's parts and the boards' sample content). `scripts/design/pixel-budget.sh`
shoots the gallery and the page replicas and diffs each against its board part by part (budgets in
`designs/factory-ui-v3/pixel-budgets/`; Foundry's own `scripts/design/pixel-budget.sh` reads
`components.json` there too, so it stays). The package registers Inter and IBM Plex Mono in its own `osyrin.lock`; an
app or a tests app that uses the look needs no font step.

The app's theme derives from the base theme: `theme Admin : OsyrinV3 { … only what is Admin's own … }`. A package's
tests app does the same (`theme TestTheme : OsyrinV3 { }`).

**Who owns it (owner, 19 Nov 2026: "SPLIT").** The platform owns this kit: this directory is its only source, it is
versioned (`version` in `package.osy`, `0.x` while its parts still move), and its address is
github.com/osysharp/look: `use Osysharp.Look@0;` resolves there (an `Osyrin.*` package publishes in the osysharp
organisation as `osyrin-<name>`, as there is no `osyrin` organisation on GitHub). It is on the platform's publish list
(`publish-list.txt`), so every platform release's kit step (`scripts/release/release-kits.py`) runs its tests app with
the release's binary and publishes it when it has no release yet, its newest release does not run on that release, or its
source changed.
**What a consumer pins:** `use Osysharp.Look@0;` (the current version is 0.4.0, `version` in `package.osy`; `0.x` while the
parts still move, so a consumer takes the newest 0.x the platform has published, and pins the exact version in its lock
file). A consumer inside this repository does not need it: it composes this directory.
It holds
the base every Osyrin product shares — the tokens and themes (`OsyrinV3`, `OsyrinConsole`, `OsyrinLook`), the console
shell, rail and bars, controls, chips and badges, tables, charts, the page head and bands, panels, states and dialogs.
A part only one product's pages use belongs in that product's own package on top of this one (Foundry's, for its plan,
release, slice and conversation parts), never here. Nothing is copied into this directory from another repository any
more: `scripts/design/sync-look.sh`, which copied Foundry's `packages/Osysharp.Look` here, is deleted.

**How a consumer proposes a change.** Foundry, or any product built on the kit, changes a base part HERE: a branch of
this repository that edits the part, its gallery sample or test in `osyrin/kits/look/tests`, and this README's rules
when a rule changes, landed through this repository's landing queue (`scripts/land.sh`). A part that needs a new
platform feature lands with that feature. The consumer adopts the change by moving its `Osysharp.Look` version once a
release has published it to github.com/osysharp/look (until then, by moving its pinned copy), and never edits a copy of a base part in its own package: a copy is how the two sides drifted
apart before the split (86 of its files differed on 19 Nov, 8 of them changed on both sides). A change that would alter how Admin draws runs Admin's pixel tests and
`scripts/design/pixel-budget.sh console` before it lands, as every console change does (below).

## Console pages

**Console pages against their boards (plan item G13).** A console page — Admin's, org-policy's — is measured against its
console board in `designs/console-B/` by numbers, not by eye: `scripts/design/pixel-budget.sh console`
(the budgets in `console-B/budgets/`, the measuring in `scripts/design/console-metrics.mjs`), on this checkout's local Admin
loop (`scripts/admin-local.sh up --build`, `seed --demo`, `probe`; `recompile` after an edit). **A slice that changes a
console page or a console part runs it before it lands** and puts its output in its RESULT beside the side-by-side shots.
It fails when a page's title, description, band, tabs, panels' side space and gaps, stat cells, table head and cells,
controls or badges are further from the board than the budget's stated tolerance, and on any word that runs out of its
box (at 1024 too: `node scripts/design/console-metrics.mjs <board> <route> --width 1024`). A page that does not draw a part
of its board is reported, not failed. A deliberate difference goes in the budget's `accepted` with its reason — the
owner's 16 px beside the rail (14 Nov 2026) is there; the boards' content (Foundry's sample rows) never is a reason.

**A console table row's detail (owner, 16 Nov 2026).** When a row has more to show, it opens as a row of its own UNDER
the row, across every column — never inside one cell (that stretches the row and floats its other cells). Build it with
`ConsoleTable(…, details: true)` and `ConsoleTableRow(cols, label: name, detail: "Where it is enforced") { slot detail {
… } ConsoleCell { … } … }`: the row starts with the console's disclosure chevron (`ConsoleDisclosureButton`, the board's
22 px grey chevron, Foundry-Product's `.tw`; read as "`detail`, `label`", says `aria-expanded`, Enter and Space work it),
and the opened detail is the row "`label` detail" on the sunken ground, its words in line with the first column. On a
phone it is as wide as the panel shows and stays in view while the table scrolls sideways. Below the Laptop breakpoint, where a table wider than its panel scrolls, its far edge fades (2rem, `EdgeFade.End`, as the tabs' does) so a cut column shows there is more. A row of such a table with
nothing to open says `details: true` and no `detail`. Outside a table — a phone card, a panel's explanation — the worded
`ConsoleDisclosure("How a limit holds an application") { … }` is the console's "› Details" (the v3 `Disclosure` is not
used on a console page). Sample and tests: `/dev/console-detail-rows`, `tests/ConsoleDetailRows.test.osy`.

**A rail group about one thing.** A `ConsoleNavGroup` may carry a `Subject` (`ConsoleRailSubject{Name,Key,State,Tone}`): its
name (12.5/600 rail ink), key (11.5 mono) and state badge are drawn over the group's rows (`ConsoleRailSubjectBlock`), in the rail and in
the phone drawer; a long name or key ends in "…" and shows whole on hover. Admin gives its application rail the application, so the rows
read as about it. (The v3 boards draw no such header; it follows the org switcher's type and spacing.)

**The console's ⌘K search.** A page gives `ConsoleFrame` (or `ConsoleShellBar`) `searchText` and `searchGroups`, and
the top bar shows the Search button and its dialog. The groups come from `ConsoleSearch(searchText)` (`live var found =
ConsoleSearch(searchText);`), which asks every `IConsoleSearchSource` in the app — a package implements it for the rows
it owns (`Group()` the heading, `Search(text)` the rows, read as the caller), ordered by `[Order(n)]`; nothing names a
source. The dialog is a modal (`UiRole.ModalDialog`): opening it, by the button or ⌘K, puts the keyboard in its field, Tab stays
inside, and Escape closes it and gives the keyboard back to what opened it — the platform does this for every modal dialog.
The search is for somebody signed in. Sample and test: `/dev/console-search` (`OrganizationSearch` in
`tests/model/ConsoleSearchPage.osy`), `tests/ConsoleSearch.test.osy`.

**A dashed placeholder strip.** `ConsoleDashedStrip(sentence, label: "", action: false) { … }` is the console's dashed
"nothing here yet / a gap to read" strip: a 16 px glyph, the sentence, and the default slot at its right (under the
sentence on a phone). The glyph says what the strip is — an info mark for a statement (a gap to read, nothing to do), the
up arrow only when `action: true`, i.e. the strip offers something to do and its slot holds the action. It is explicit,
never inferred: a slot cannot be asked whether it is filled, and a statement is the safe default. Read as `label`, else
the sentence.

**The phone's tab strip.** `ConsoleTabs` on a phone scrolls sideways when its tabs are wider than the band: the tab in
view is scrolled into view when the page loads, the strip's far edge fades (2 rem, the board's own fade), and a 2 rem
gutter at its foot and as scroll padding keeps the last tab, and a tab scrolled to, clear of the fade. A tab's `Count`
stands after its words as a badge (warn tone when its `CountTone` is warning). On a desk nothing scrolls and nothing fades.

**Leaving a page with unsaved edits.** The console shell bar's avatar menu and the phone's product switcher ask before
leaving while the page holds unsaved edits (`Navigation.IsDirty`) — keep editing, discard, or save and leave — as the
rail's rows do.

**What needs somebody is one line above the table, never a panel beside it (owner, 16 Nov 2026).** An attention or
status message about a table's rows — "1 address not proven", "3 invoices overdue" — is ONE line between the stat strip
and the table, with its action in the line: `ConsoleDashedStrip(sentence, label: "…", action: true) { ConsoleButton(…) }`
(or a `ConsoleNote` when there is nothing to do). One → "1 address not proven — ann@acme.test · Resend"; several →
"5 addresses not proven — ann@…, bob@…, cy@… and 2 more · Resend all · Show", where Show opens the list one by one under
the line, still above the table. A side column (`PageColumns … slot side`) never holds such a panel: beside a table it
takes the table's room, and the owner's Operators page (owner-operators-1116.png) cut every name and address to three
letters. `scripts/design/console-metrics.mjs` fails a page whose table's panel shares its row with another panel while
narrower than 75 % of the work column with a narrower panel beside it ("a table squeezed by a side panel"). Sample: Admin's `/principals`
(`admin/model/pages/principals.osy`, test `operators_lists_people_only_and_every_figure_is_its_rows`).

**A stat's sub line never splits a count from its word.** `ConsoleStatStrip` draws a stat's `Sub` ("1 down · 1 healthy ·
1 retired") with each part's leading count joined to the word after it by a no-break space (`ConsoleStatSubWords`), in the
cell and in the phone's spanning line. A narrow cell — four states in Admin's Zones figure on a 1280 desk — then breaks the
line between parts or inside a part's words, never as "1 / healthy". A part that starts with no count ("none failing") is
drawn as it is. Write the sub with ordinary spaces: the join is the strip's. A UI test reads a no-break space as the space a
person sees, so `Assert.Visible("1 healthy")` finds it (the driver's `plainText`). Sample: Admin's `/infra`, test
`the_zones_figure_breaks_between_its_counts_never_inside_one` (`admin/tests/infra-zones-list-ui.test.osy`).

## The rules

1. A key is always a `KeyLink` (mono, accent) — or, in its own header, a `KeyChip`. Never plain text.
2. A status is always a `StatusChip(words, tone)` with its dot. Never a bare `Badge`, never coloured text.
3. An object page starts with `ObjectHeader` and `TabBar`; an area page with `AreaHeader`.
4. Content sits in section cards (`SectionCard`, `SmallCard`), never loose on the ground.
5. A table lives inside a card: `CardTable` + `CardTableRow`, or `TableCard` in one call. Never the kit's `DataGrid`
   (that is for execution inspection only). A head and its rows get the same `spacing` and `dense`; no part sets a
   table's gap or padding of its own (`TableRhythm` is the one place).
6. Destructive triggers are quiet: `LookButton(kind: ButtonKind.Danger)` (outlined) or `Quiet`, never a filled red.
7. One primary action per header or band: `LookButton(kind: ButtonKind.Primary)`.
8. The numbers and states a person steers by are `StatTile`s under the header (in a `StatRow`), not a sentence.

## The parts

| Part | Props | Board part | Example |
|---|---|---|---|
| `AreaTitle` | `text, tight = true` | Components §2 page title 28/700 | `AreaTitle("Patient Portal 2.4")` |
| `SectionTitle` · `ItemTitle` · `SummaryText` · `DenseText` | `text` | §2: 18/700 · 17/600 · 15/1.55 · 14 | `ItemTitle("Notes")` |
| `MetaLine` | `List<string> facts` | §2 metadata 13 Ink 3, joined " · " | `MetaLine(["Booking", "38 min"])` |
| `Overline` · `Eyebrow` | `text` | table head 11/600 · card eyebrow 12/600 | `Eyebrow("Next")` |
| `KeyText` | `text` | §2 mono 13/600 | `KeyText("v3")` |
| `KeyLink` | `key, to, small` | §3 chips: the key that opens its item | `KeyLink("BOOK-14", "/plan/BOOK-14")` |
| `KeyChip` | `key` | Slice/Release header: the object's own key | `KeyChip("BOOK-12")` |
| `PlainChip` | `words` | neutral fact chip (kit `Badge`, Default) | `PlainChip("Next.js web app")` |
| `RideChip` | `release` | Slice "rides Patient Portal 2.4" (Ink words) | `RideChip("Patient Portal 2.4")` |
| `ReleaseChip` | `release` | §3 "→ Patient Portal 2.4" (kit `Badge`, Info) | `ReleaseChip("Patient Portal 2.4")` |
| `PersonChip` | `name, initials, quiet` | §3 person; `quiet` 13/400 Ink 2 beside a header title | `PersonChip("Anna Berg · lead", "AB", quiet: true)` |
| `ProposedTag` | `label = "Proposed", small` | §3 dashed tag; `small` 11/600 beside a card heading | `ProposedTag(small: true)` |
| `SolidChip` · `OutlineChip` | `words` | PolicyFloor "Hard boundary" (white on ink) · "Exception-eligible" (outline) | `SolidChip("Hard boundary")` |
| `HashBox` | `hash` | PolicyFloor inline hash | `HashBox("4e1a07c")` |
| `StatusChip` · `StatusDot` | `words, StatusTone tone` | §4 status chip; `StatusTone.Floor` draws the board's 12 px lock (`LockMark`) in place of the dot (v3 PolicyFloor "v4 in force", PolicyFloorEffective "floor v4") | `StatusChip("In review", StatusTone.Run)` |
| `LockMark` | — | the lock: two outlined boxes in the floor ink (shackle over body, 1.3 px), not an SVG — a package's SVG does not reach its app | (used by `StatusChip(…, StatusTone.Floor)`) |
| `Meter` | `label, int value, int max, LookTone tone = Run, int threshold (0 none), LookTone pastTone = You, amount, pastWords` | on no board as such, in their language: the label 13/600 and the amount right (12 tabular Ink 2, "312 of 500 GB") over a 6 px pill on Line 2 filled in the tone's dot; at or past `threshold` the fill, a dot and the amount (600, its ink) take `pastTone` (amber: it needs a person), `pastWords` follows ("near the cap"). One quantity against a limit; `ProgressShares` is a group's parts | `Meter("Sessions", 7, 8, threshold: 6, pastWords: "near the cap")` |
| `Legend` · `LegendEntry` | `label = "Legend"`; block = entries · `text, strong`; block = the chip it explains | orgsettings/Policies legend: white on the hairline, radius 10, `10px 14px`, 13 Ink 2, entries wrapping `8px 18px`; an entry is its chip 8 px before its words (`strong`: OnSurface 700) | `Legend("What the chips mean") { LegendEntry("A governor may grant a scoped exception.") { OutlineChip("Exception-eligible"); } }` |
| `KitTone` | `ChipTone` → kit `Tone` | tone for a kit `Badge` | `Badge("Published", tone: KitTone(ChipTone.Ok))` |
| `Breadcrumb` · `CrumbLink` | `List<Crumb>` | §12 breadcrumb | `Breadcrumb([new Crumb { Label = "Settings", To = "/settings" }])` |
| `ObjectHeader` | `title, trail, situation, lead, item, actionsUnder`; `slot chips`; block = actions | §12; `lead`: Product/Release header (chips beside the title, actions at the bottom; `actionsUnder`: always on their own line under the situation, as Release draws them); `item`: Slice header (chips ABOVE the title, actions at the top) | `ObjectHeader("Slot picker …", trail, situation: "…", item: true) { slot chips { KeyChip("BOOK-12"); } LookButton(…); }` |
| `PageColumn` | `label, landmark, ground` | the page frame's main column: grows from 640 to at most 1120 px, parts 18 apart (`landmark`: it is the page's main region). In a row (Foundry's shell) or a column of fixed height (Admin's content region): never shorter than its parts (`min-height: min-content`), so the parent scrolls or grows and no card is squeezed to its head; `ground`: padding on the ground around it, inside the 1120 (Admin: `"20px"`). Proof: `/dev/parts/column`, `/dev/parts/row` | `PageColumn("Policies", landmark: true, ground: "20px") { … }` |
| `PageState` | `PageStateKind kind (NotFound · Forbidden · Error · Offline), title, text`; block = its one action | a whole page that is only a state, when there is no page to sit in (the platform session's placement for Admin): on the ground, centred, 96 px from the top, at most 520 wide; the state card (`StateCard`, an error on the bad line) with the overline ("Not found" · "No access" · "Error" · "Offline"), the title 22/700 as the h1, 14/1.5 Ink 2 text, the action 4 below; the main region, an alert for error and offline. `/dev/parts/not-found` … `/offline` | `PageState(PageStateKind.NotFound, "There is no release 2.9", "…") { LookButton("Open the releases", Go, kind: ButtonKind.Primary); }` |
| `AreaCard` | `title, count`; `slot side`; block = rows | orgsettings/Policies' group cards (the settings boards' area head): head `12px 20px` over a SoftLine rule, title 16/700 (h2), at the right a count (13 Ink 3) or the `side` slot (a chip) — no link; `SectionCard` keeps the page boards' 18/700 head with a link | `AreaCard("Network", count: "2 policies") { … }` |
| `TabBar` | `label, List<TabItem>, current` | §12 tab bar (`TabItem{Label,To,Count,CountTone,Built}`) | `TabBar("BOOK-12 sections", tabs, here)` |
| `AreaHeader` | `title, tabs, current, situation` | area head + tabs | `AreaHeader("Products", tabs, "/products")` |
| (`PageTab`) | — | Admin's `PageTab(label, to, count, countTone)` is `new TabItem { Label, To, Count, CountTone }`: the look needs no part for it | `new TabItem { Label = "Components", To = "/infra/status/components", Count = "2" }` |
| `EmptyPanel` · `ErrorPanel` · `PermissionNote` · `LoadingPanel` | see source; `EmptyPanel(title, body, overline = "Empty", framed = true)` | §11 states. `EmptyPanel` is as tall as what it says (L1; it still fills a grid cell or a row, but takes no share of a tall column — it said `height: 100%`, which left dead space under its button in a page column); `framed: false` is the empty state INSIDE a card (no frame, `10px 20px 20px` under the card's head), with the "Add …" as the card's action (`SectionCard(…, actions: true)`). `/dev/controls/row-menus` | `EmptyPanel("No releases yet", "A release groups …") { LookButton(…); }` |
| `PermissionNote` | `permission, heldBy, mono = false`; block = its link | §11 permission note on the wash; the permission is words in the text face ("policy admin"); `mono: true` only for a code name ("settings.write", key face 13) | `PermissionNote("policy admin", "Org admins — Jonas Lindqvist")` |
| `StateCard` · `StateOverline` · `StateLink` · `SkeletonLine` | `StateKind (Empty · Error · Note · Loading · Dashed)` | §11 parts; `Dashed`: white in a dashed proposed line, `14px 20px` | `StateLink("Try again", Retry, first: true)` |
| `DashedNote` | `title, text, level = 2`; block = a link or chip | orgsettings/Policies "Not yet governable": what is NOT covered, set apart — a dashed proposed line on white, radius 14, `14px 20px`, 8 apart, title 16/700, text 14/1.5 Ink 2. The solid note on the wash stays `StateCard(StateKind.Note)` | `DashedNote("Not yet governable", "Listed so nobody assumes they are covered: …")` |
| `SectionCard` | `title, eyebrow, sub, linkLabel, linkTo, band, actions, clip = false`; `slot sub`, `slot actions` (drawn at the head's right when `actions: true` — Item "Add a part"); since L1 the card does not clip what leaves it (a row's menu, a hover title on its last row); `clip: true` cuts at its edge as before | `band`: ProductPlan release card (eyebrow beside a 20/700 title, 14 Ink 2 line holding a chip, link top right); else Release "Contents" (18/700, link on the baseline) | see the worked examples |
| `StageCard` · `CardTitleLink` | `title, to, eyebrow, sub, lead`; `slot side`, `slot tags`; block = more rows · `title, to, lead` | ProductReleases release cards: `lead` the "Next · planned" card (eyebrow above a 22/700 title link, 14 Ink 3 line, outlined buttons top right, `18px 20px`); else "Then" (one row, space between: eyebrow beside an 18/700 title link, the chip right; the line under it, `16px 20px`) | `StageCard("Patient Portal 2.5", to, eyebrow: "Then", sub: "2 items · 1 done") { slot side { StatusChip("Planned", StatusTone.Wait); } }` |
| `ListCard` · `ListCardRow` | `title, label` · `title, to, List<string> facts`; `slot side` | ProductReleases "Released": 17/700 card title, then rows (16/600 title link over a 13 px meta line, a chip right, `12px 20px` under Line 2), no table head | `ListCard("Released") { ListCardRow("Patient Portal 2.3", to, ["Fri 30 Oct", "9 items"]) { slot side { StatusChip("Released", StatusTone.Ok); } } }` |
| `GoLink` | `label, to, small` | the quiet "›" link, 14/600 (13/600 `small`) | `GoLink("Release plan ›", "/releases/2.4")` |
| `CardFoot` | `text, quiet`; block continues it inline | card foot line | `CardFoot("Also in 2.4: ") { KeyLink("BOOK-10", …); Text(" …"); }` |
| `SmallCard` | `title, kind (Summary 17/600 · Foot 15/600 · Side 16/700), linkLabel, linkTo`; `slot title` | Release 3-up and 4-up cards, Slice side cards | `SmallCard("Waits on · Unlocks", kind: SmallCardKind.Side, linkLabel: "Dependencies ›", linkTo: …) { slot title { ProposedTag(small: true); } … }` |
| `PlainCard` | `label, heading`; `slot heading` | Release facts card, Slice "Right now" (`16px 20px`, gap 10) | `PlainCard("Release facts") { FactsGrid(…) { … } }` |
| `CardGrid` | `min, gap` | `repeat(auto-fit, minmax(min(300px\|250px\|240px, 100%), 1fr))` | `CardGrid("240px") { SmallCard(…); … }` |
| `DecisionBand` | `question, meta`; block = the primary | Release "Approve … for production" | `DecisionBand("Approve 1.8.1", meta: "…") { LookButton(…, kind: ButtonKind.Primary); }` |
| `StatTile` · `StatDot` | `label, value, sub, StatTone tone (None · Ok · Run · Wait · You · Bad · Neutral), to, shareDone, shareTotal, shareTone, mono, emphasis, compact`; `slot sub` | on no board, drawn in their language: the KPI tile under a header — small card `14px 16px` radius 14, the label an 11/600 uppercase overline in Ink 3 after the tone's dot, the value 24/700 tabular (`mono`: the key face at 600; past 16 characters it shrinks to 17 and wraps, never clips), `sub` 13 Ink 2 (the slot continues it: a chip), a 4 px bar at the foot for `shareDone` of `shareTotal`; with `to` the whole tile is one link named "label: value, sub" (wash on hover, focus ring); a tone tints only the dot and the bar, `emphasis` gives the tile the tone's tint and ink. Gallery "Stat tiles", `/dev/stat-tiles` | `StatTile("Items done", "7 of 12", sub: "5 still open", tone: StatTone.Run, to: …, shareDone: 7, shareTotal: 12)` |
| `StatRow` | `label, compact`; block = `StatTile`s | the row of tiles: equal shares of the width, at least 150 px each, 12 apart (`compact`: 10), two to a row on a phone, each as tall as its row. COMPACT (`StatRow(…, compact: true)` with `StatTile(…, compact: true)`, on no board — the tile tightened, for a page that holds a lot without scrolling): about 70 px high where the tile is about 110, `8px 14px`, lines 2 apart, the value 18/700 (15 past 16 characters), the sub line kept with the share bar beside it (40 px, pushed right); same tones, link and wrapping. `/dev/controls/display` | `StatRow("Release at a glance") { StatTile(…); … }` |
| `ProgressShares` | `merged, running, waiting, notStarted, counts, StatusTone waitingTone` | ProductPlan progress column (8 px pill + 12 Ink 3 counts); `waitingTone: StatusTone.You` draws the waiting share amber, where the page's chips say "Waiting on …" in `You` (default: the boards' grey wait dot) | `ProgressShares(2, 1, 1, 0, "2 merged · 1 running · 1 waiting")` |
| `ShareBar` · `ShareFill` | `done, total, StatusTone tone, w` | on no board, drawn from ProgressShares' pill: a small 6 px bar, `done` of `total` in the tone's dot on Line 2, decorative (words beside it say the count). Gallery "Stat tiles" | `ShareBar(5, 8, StatusTone.Ok, w: "120px")` |
| `ProgressRow` | `title, to, sub, done, total, barTone`; `slot side` | a `ListCardRow` with a bar: title link over a 13 px line, a `ShareBar` in a quarter of the room (at most 120 px), the chip right (ProductReleases "Then" stages as rows of one card) | `ListCard("Then") { ProgressRow("Portal 2.5", to, "4 items · 1 done", 1, 4) { slot side { StatusChip("3 not ready", StatusTone.You); } } }` |
| `CountPill` | `count, words, StatusTone tone` | a count that matters as a pill: the number and its words as one text ("3 not ready"), 12/600 tabular, `2px 9px` on the tone's tint (§4's tones, pill ends) | `CountPill(3, "not ready", StatusTone.You)` |
| `LookDate` | `label, Binding<DateOnly?> value, w, state, message, clearable, placeholder, help, console` | `LookField`'s frame around the kit's `DatePicker` (it draws the label and keeps it as its name); the look has no date box of its own. `console` (below): the dropdowns' console face over the same month calendar, "Clear" beside the face | `LookDate("Target date", target, clearable: true)` |
| `StatToneOf` · `InitialsOf` | `StatusTone` · `name` | a tile's tone for a chip's, so both agree · a `PersonChip`'s initials ("Pia Planner" → "PP") | `StatTile("State", shown.Words, tone: StatToneOf(shown.Tone))` |
| `RowTitle` | `title, to` | ProductPlan row title 16/600 | `RowTitle("Show free slots", "/arcs/BOOK-SLOTS")` |
| `LevelTag` · `CheckMark` | `text, lit` · `met` | Slice dependency level, acceptance mark | `CheckMark(true)` |
| `ChainRow` | `level, what, key, keyTo, state, tone, lit, quiet`; block under `what` | Slice "Waits on · Unlocks" row (`64px 1fr`, Line 2 rule, `6px 0`); no key and `quiet` for one the reader may not see | `ChainRow("Waits", "Receipts", key: "ALPHA-4", keyTo: to, state: "Open", tone: StatusTone.Wait) { LookButton("Remove link", Remove, kind: ButtonKind.Quiet, small: true); }` |
| `PlannerMark` | — | the planner's hexagon before a proposal | (used by `CalloutBand(mark: true)`) |
| `CardTable` · `CardTableRow` · `TableGroupBand` | `label, cols, heads, roomy, spacing, dense, minWidth, aligns, sortable, sortBy, sortDescending, onSort(head, descending), stack = true` · `cols, roomy, label, spacing, dense, onPress, open, heads, stack = true`; `slot under` · `eyebrow, title, key, keyTo, innerKey, innerKeyTo, innerTitle, label`; block = the band's right-hand note | table in a card; `spacing` (`TableSpacing`, numbers in `TableRhythm`): `Plain` = Release (head 11/600 ruled, rows `10px 20px`, 12 apart), `Roomy` (= `roomy: true`) = ProductPlan (head 12/600 no rule, rows 12 px), `Settings` = the settings boards (head `7px 20px`, rows `11px 20px`, 14 apart), `Tight` = PolicyFloorEffective (rows `9px 20px`, 10 apart), `Long` = a long list (rows `6px 20px` over a 21 px line, 34 high with text or a chip, head `5px 20px` ruled; on no board at 34 — HostConfigIndex's 36 tightened); `dense` = the compact row, `7px 20px` over a 20 px line, 34–36 high, head `5px 20px` (HostConfigIndex's index draws a 36 px row); `minWidth` (e.g. "900px") = the table scrolls sideways inside its card, the card's head and foot stay; `aligns` = one `TextAlign` per column (a right-aligned head over right-aligned cells: give the cells `textAlign` too); `sortable` heads are buttons with a small arrow (↕, ↑, ↓) and `aria-sort` that REPORT to `onSort` — the caller's query sorts and passes `sortBy`/`sortDescending` back; the band keeps each key with its title when it wraps, and its block is ReleasePlan's right-hand note (12, Ink 2, pushed right: the group's own reason, a quiet action) | see the worked examples; `/dev/controls/tables` |
| `TableCell` | `align`; block = the cell's content | one cell of a row, for a part that takes no `role:` — see "A table is a table" | `TableCell { StatusChip("Done", StatusTone.Ok); }` |
| A table on a phone (L1) | `CardTable`/`TableCard` `stack = true`; `CardTableRow`/`SettingTableRow` `heads, stack = true`; `StackedHead` (`head, row`, the label the rows draw) | on no board (asked by org-policy's OP-P3b shots at 390): below the Phone breakpoint (the table's room under 480 px) a row is ONE column — `CardTableRow`: its first cell at full width, then each other cell under its column's head (11/600 uppercase Ink 4, 6 above the cell, from `heads`, the table's own list); `SettingTableRow`: the name and key, the value and its tag line, then the page's cells under `heads[2…]`; then `under` at full width. The head row is not drawn (it stays, one pixel and clipped, for a reader and `Assert.Cell`), `minWidth` is ignored: nothing scrolls sideways in a card. Without `heads` the cells stack unlabelled, 6 apart. `stack: false` on the table AND its rows keeps the columns (and `minWidth`'s scroll) at every width. A sortable head is not drawn there (sorting waits for a desk). A cell that says `textAlign: Right` stays right-aligned in its full-width line. `/dev/controls/phone-tables` | `TableCard("Keys", cols, heads) { foreach (var k in keys) { CardTableRow(cols, label: k.Name, heads: heads) { … } } }` |
| `TableCard` | `title, cols, heads, sub, linkLabel, linkTo, foot, spacing, dense, minWidth, aligns, sortable, sortBy, sortDescending, onSort, pageSize, total, page, onPage(page), stack = true, actions`; `slot actions` (with `actions: true`, at the head's right beside the link: the card's primary "Add …", L1); block = the rows | a table in a card in one call: `SectionCard` head (title, `sub`, link), `CardTable` (labelled `title + " table"`), the rows, a paging row "1–25 of 312 · Previous · Next" when `pageSize` > 0 (`page` from 1; Previous/Next quiet, disabled at the ends, REPORT the page wanted — the caller's query pages), then `foot` as a `CardFoot` | `/dev/controls/tables` "Hosts" (sorted, paged, dense), "Network" (wide, scrolls at 390) |
| `CardTableRow(onPress, open)` | `onPress`, `open`, `label` (with the row's other props) | a row pressed as a whole (Components §15, a list with a drawer): a click anywhere on the row, or Enter/Space on its ONE focus stop (a button laid over the row, named by `label` — give the first cell's words), calls `onPress`; a link or button in a cell takes its own press and does not open the row; `open: true` = the board's open row, the accent tint (`#EEEBFF`) inside a 1 px accent-line ring (`#CFC8FA`), said as `aria-current` and `aria-selected` (`Assert.Selected(label, true)`). Rows still count (`Assert.Items`) and read by column | `CardTableRow(cols, label: r.Id, onPress: () => Open(r.Id), open: r.Id == open) { … }`; `/dev/controls/display` "Rules" |
| `ClampText` | `text, lines (1 · 2), title, lineHeight, role` | on no board (the boards cut a long line with an ellipsis): text cut after `lines` lines — one line ends in "…", two are cut at the second line's foot (the platform has no line clamp: no "…" there); with `title` (default) hovering shows the whole text in the look's hover title, a dark 12/500 label `6px 8px` under the text, max 360 wide, a `tooltip` named "Whole text" (the platform has no `title` attribute); a screen reader reads the whole text. Inside a `SectionCard`/`TableCard` the title is whole over the card's edge (since L1 the card does not clip; `SectionCard(…, clip: true)` cuts it) | `ClampText(row.Note, role: UiRole.GridCell)`; `ClampText(reason, lines: 2)`; `/dev/controls/display` "People" |
| `RelativeDate` · `RelativeWords` · `ExactWhen` | `DateTime at, DateTime? now, role, end` · `at, now` · `at` | a moment said relative to now in the look's ONE wording (below), the exact time as its hover title (`tooltip` "Exact time", `end`: hung from the right edge in a right-aligned column); `now` defaults to the clock (UTC), a test pins it with `TestClock.Set`. `LookDate` stays the date INPUT; this is the date SHOWN. The wording, `RelativeWords(at, now)`, UTC calendar days: under a minute either way "just now"; under an hour "N min ago" / "in N min"; the same day "N h ago" / "in N h" (whole hours, down); the day before "yesterday", the day after "tomorrow"; else "12 Nov" this year, "12 Nov 2025" another. `ExactWhen`: "Thu 12 Nov 2026, 09:05 UTC". Never another wording ("2 hours ago", "Nov 12") | `RelativeDate(change.At)`; `/dev/controls/display` "Recent changes" |
| `HoverTitle` | `text, name, end` | the look's hover title (the platform has no `title` attribute): a dark 12/500 label under its owner, a `tooltip` named `name`; its owner shows it while hovered (`onPointerEnter`/`onPointerLeave`) and stands `position: relative` | inside `ClampText`, `RelativeDate` |
| `PersonCell` | `name, address, initials, photo, to, cell` | on no board as a cell: `PersonChip`'s circle grown to 28 px (initials, by default `InitialsOf(name)`, or `photo`), the name 13/600 over the address 12 Ink 3, each one line cut with "…" in a narrow column (both still read whole); `to` = name and address one link named by the name; it IS the table cell (`cell: false` outside a table) | `PersonCell(m.Name, m.Email, to: "/people/" + m.Id)`; `/dev/controls/display` "People" |
| `RankNumber` | `int rank` | ProductPlan rank | `RankNumber(1)` |
| `CalloutBand` | `lead, rest, mark`; block = small buttons | ProductPlan planner proposal | `CalloutBand("The planner proposes …", "…", mark: true) { LookButton("Evidence", A, small: true); }` |
| `AmberNote` | `text, indent` | "Will merge after …" | `AmberNote("Will merge after BOOK-11")` |
| `Disclosure` | `summary, ruled, page, trailing, indent`; block = body | "▸ Details" (`indent`: in a row's `under` slot), "Diagnostics ▸" | `Disclosure("Details and predicted token use", ruled: false, indent: true) { … }` |
| `FactsGrid` · `FactCell` | `label` · `term, value, mono` | Slice/Release facts | `FactsGrid("Facts") { FactCell("Candidate", "9c41e7a", mono: true); }` |
| `LookButton` | `label, onPress, kind, small, disabled, name, canPress, whenDenied, wide, announce`; block = a leading glyph, 8 px before the words (`announce` (or the older `name`): the announced name of a button repeated per row, its words first: "Remove link to ALPHA-5"; `canPress`: a server policy, e.g. a function's `CanCall` — where it says no the button is drawn disabled and `whenDenied` is its description, `Assert.DisabledBecause`; `wide`: fills its row) | the page boards' buttons (outlined `10px 14px`, primary `10px 16px`; §3's outlined row is `10px 16px`, the pages win); `ButtonKind.Inline` a quiet 13/600 accent action with no box, inside a table row or a card, so the row keeps its height (Release "Take out"); `ButtonKind.CardLink` the same at 14/600, a card's own action drawn as its link (Release "Open Notes ›") | `LookButton("Take out", TakeOut, kind: ButtonKind.Inline)` |
| `MoreMenu` · `MenuAction` | `label, open, onToggle, menuLabel, announce, small, popover = true, console, wide` · `label, onPress, danger, announce` | Release header "More ▾" and its menu (`console: true`: the trigger is a `ConsoleButton`, for a console header beside `ConsoleButton`s; `wide` as its own; menu 38 below) (240 wide, radius 10, the drawer's lift; items 14/400 `9px 10px`); the page holds `open`; it closes through `onToggle` on a press outside (the kit's `Backdrop`), on Escape (the keyboard returns to the button) and when the page ends an item's action with `open = false`. IN A ROW (L1): `announce` names each row's button apart, its words first ("More for GH_TOKEN"), `small` the 36 px button (menu 42 below); the menu opens over the card's edge — `SectionCard`/`TableCard` no longer clip — so the last row's menu is whole; on a phone `popover: Viewport.AtLeast(Tablet)` draws the button alone and the page draws a `MenuSheet` at its own level. `/dev/controls/row-menus` | `MoreMenu("More ▾", open == k, () => Toggle(k), menuLabel: "Actions for " + k, announce: "More for " + k, small: true, popover: Viewport.AtLeast(Tablet)) { MenuAction("Revoke", …, danger: true, announce: "Revoke " + k); }` |
| `GlyphButton` | `glyph, onPress, name` | Plan board's ↑ ↓ at the right of a row: a 32 × 32 square, 600 13 Ink 2 on white in the control line, radius 8, the wash on hover; `name` is its accessible name ("Move BOOK-SLOTS up"). `/dev/parts/row-controls` | `GlyphButton("↑", Up, "Move BOOK-SLOTS up")` |
| `ChipMenu` · `ChipFace` | `words, open, onToggle, name, menuLabel`; block = `MenuAction`s | Plan board's release chip that is the control ("Next · 2.4": 600 12 Ink on Line 2 in the Ink 2 line, radius 6, `5px 10px`, 32 high; open: the accent line); its menu is `MoreMenu`'s card 38 below the chip's left edge; on a phone only the chip, and the page draws a `MenuSheet`. The page holds `open`. `/dev/parts/row-controls` | `ChipMenu("Next · 2.4", open, Toggle, name: "Release of BOOK-SLOTS: Next · 2.4. Change it") { MenuAction("Not scheduled", Out); }` |
| `MenuSheet` | `label, onClose`; block = `MenuAction`s | a phone's menu: a sheet at the window's foot over a scrim (white, radius 14, `12px 12px 20px`, the dialog's shadow, `label` its 13/600 Ink 3 head; Escape or a press outside closes; the sheet is named `label + " sheet"`, the menu in it `label`, for `Assert.Below(…)` at 390). Render it at the page's level, never inside a card (a card is a containing block; no portal). `/dev/parts/row-controls` at 390 | `if (open && !Layout.AtLeast(Tablet)) { MenuSheet("Release of BOOK-SLOTS", Close) { MenuAction("Not scheduled", Out); } }` |
| `ProblemCard` | `title, chip = "Refused"`; block = what stopped it | ReleaseEditing "Not saved: … changed while you edited" (bad line, `16px 20px`, 18/700 + bad `StatusChip`), an alert | `ProblemCard("2.4 is not released") { Text("…"); }` |
| `FormCard` | `title, subtitle`; `slot actions` (its close); block = field, rows | ReleasePlan's "Add items" card (Line hairline, `14px 18px`, gap 8, title 15/600 sans h3, nothing under the rows) | `FormCard("Add items", subtitle: "to 2.4") { slot actions { LookButton("Done", close, kind: ButtonKind.Inline); } LookInput(…); … }` |
| `EditingCard` | `title`; block = editor, then buttons | ReleaseEditing's "Release" card being edited (2 px accent line, `18px 20px`, gap 12) | `EditingCard("Notes") { MarkdownEditor(…); LookButton("Done writing", …); }` |
| `FieldRow` · `FieldGroupHead` | `label, help, note, changed, was, controlW`; block = the control · `text, first` | SagaArcEdit's field rows (label 14/600 over help 13 Ink 3 left, control right at most `controlW` = 300 px, `12px 20px` under Line 2; `changed` = accent tint 2 + "● Changed · was …"; stacks under 640) and its group heads ("WHAT IT IS", 13/600 uppercase Ink 4) | `FieldRow("Title", help: "Shown everywhere BOOK is named.") { LookInput("Title", title, inRow: true); }` |
| `LookInput` · `LookSelect` · `LookTextArea` · `LookField` | `label, value (Binding<string>), w, state (Normal · Editing · Refused), message` (+ `LookSelect`: `options`, `words`, `placeholder`, `help`, `inRow`, `disabled`, `small` (36 px), `onChanged`, `console` (below); `LookTextArea`: `h`; `LookInput` and `LookTextArea`: `placeholder`; `LookInput`: `mono`, `inRow` = no label of its own, its `FieldRow` draws it, `onInput` each keystroke, `debounceMs` = only after that pause, `onEnter`, `search` = leading glass + a "Clear <label>" × while filled, Escape clears, `type` = the browser's kind of text, "email" for an address) | §3 inputs: 44 high (`LookSelect` is §22's dropdown, as `LookEnumSelect` below, over strings — never the browser's select), `10px 12px`, radius 8, 2 px accent while edited, bad line + 12 px message | `LookInput("Release name", name, w: "220px")` |
| `LookCheckbox` · `LookCheckboxBound` | `label, checked, onToggle, help, disabled` · `label, Binding<bool> value, onToggle, help, disabled` | §22 checkbox: 18 px box radius 4 in the proposed line, the accent with a white tick when ticked, 14 px words 8 after, a 13 px Ink 3 help line, at least 32 high, the focus ring; controlled for a derived state, bound for a field (the press writes it, `onToggle` commits) | `LookCheckbox("Run docs checks", docs, FlipDocs)` |
| `LookSwitch` | `label, on, onToggle, onWord = "On", offWord = "Off", help, inRow, disabled` | §22 switch, for a setting that takes effect at once: 34 × 20 track (accent on, control line off), 16 px knob, its word after; the label 13/600 above (`inRow`: the `FieldRow` draws it); controlled | `FieldRow("Live updates", help: "…") { LookSwitch("Live updates", live, Apply, inRow: true); }` |
| `LookEnumSelect<T>` | `label, Binding<T> value, T[] options = T.Members, w, help, inRow, disabled, onChanged, console` | §22 dropdown over an enum (words from `[Label]`): closed the field frame (44, `0 12px`, radius 8, control line, ▾), open the 2 px accent line and the list under it — as wide as the field, 4 below, radius 8, the menu's lift, rows `8px` radius 6, the chosen one ✓ in the accent ink, the lit one Line 2. ↓/↑ open and move, Enter picks, Escape closes (focus stays), a press outside closes. §22 draws radius 8, not `MoreMenu`'s 10, so it follows §22 | `LookEnumSelect("Runs when", check.Cadence, help: "Which slices the check runs on.")` |
| `LookRefPicker<T>` | `label, Binding<T> value, Binding<string> search, T[] options, Func<T,string> words, placeholder, w, help, inRow, disabled, searchFrom = 8, onChanged, console` | §22's dropdown over ONE PAGE of an entity's rows: the page's query keyed on `search` and bounded (`.Take(20)`), never the table. From `searchFrom` rows (or a search typed) a search box ("Search <label>", 36 high) heads the list, autofocused; ↓/↑/Enter move and pick from it, Escape closes and the box takes the keyboard back; "Nothing matches …" and, past one answer, "More match — keep typing". Without the search the keyboard is `LookEnumSelect`'s | `LookRefPicker("Host", host, find, hosts, h => h.Name, placeholder: "Choose a host")`; `/dev/controls/row-picker` |
| `console` on the three dropdowns, `LookDate` and `LookNumberInput` | `console: true` | A dropdown, a date or a number in a CONSOLE form (beside `ConsoleInput`, `ConsoleButton`, in a `SettingRow` of a console page) is the console's control (spec "Components", Forms): on a desktop the 30 px face, `0 10px`, radius 5, 13 px in the console's control line and surface, its label 12.5/500 as `ConsoleField` draws it and `help` 12 px text-2 (the dropdowns and the date; the number has no `help`); on a phone the 44 px tap target. The list it opens is §22's either way. Without it the face is §22's 44 px field (`LookShell` pages, Foundry) — `LookSelectFace(console:, consoleOpen:)` is the face the dropdowns and the date draw; the number's box is `ConsoleInput`'s (`0 10px`, radius 5, 13 px, its unit 13 px text-2; a value out of range refused under it in the console's critical ink, the box's line critical, the console's accent line while edited). A console form takes no `NumberField` or `DecimalField` (the Ui kit's 44 px fields): `LookNumberInput(console: true)`, `ConsoleDecimalInput` | `LookEnumSelect("Kind", kind, w: "200px", console: true)` beside `ConsoleInput("Currency", …)` |
| A refused console button | `canPress`, `whenDenied` on `ConsoleButton` | A console verb the caller may not use, drawn where it would be (a setting row's Change, a head's action) when hiding it would leave the reader wondering where it went: the dim face `disabled` draws, as tall as its neighbours, and `whenDenied` the reason a screen reader hears and `Assert.DisabledBecause` reads. Where the verb would only tease (an Add on a page the reader can never act on), leave it out and say who may in a line instead | `ConsoleButton("Rename", StartRename, canPress: may, whenDenied: "Only an owner renames it.")` |
| A console field's line is its input's description | `help`, `error` on `ConsoleInput`, `ConsoleTextArea`, `ConsoleDecimalInput`; a console `LookNumberInput`'s refusal | The 12 px line under a console control — its help, or its refusal in the critical ink — is what a screen reader hears after the input's name (`aria-describedby`), and what `Assert.Described(label, words)` reads. Each of these controls draws its OWN line beside its input (`describedBy:` on the input, `id:` on the line): a handle does not cross into `ConsoleField`, and `Slot(id:)` names a slotted control but cannot describe it, so `ConsoleField`'s own `help`/`error` stay for other slotted controls (and are not tied). Words drawn beside an input but untied pass `Assert.Visible` and fail `Assert.Described` | `ConsoleInput("Slug", slug, help: "Lower-case letters, digits and dashes.")` → `Assert.Described("Slug", "Lower-case letters")` |
| `ConsoleTextArea` | `label, Binding<string> value, help, error, placeholder, rows = 3, w = "100%"` | Console-B Forms: a few lines of plain text in a console form (a reason, a note), drawn in `ConsoleField` as `ConsoleInput` is: `rows` lines of 20 px plus `8px 10px`, radius 5, 13 px (the phone's body size below Tablet), the console's surface and control line; with an `error` the line is the critical one and `ConsoleField` says the message. Filled in a test by its label | `ConsoleTextArea("Why", reason, help: "Shown to the organization's owners.")` |
| `ConsoleDecimalInput` | `label, Binding<decimal> value, min, max, prefix, unit, help, error, placeholder, w = "100%", inRow` | Console-B Forms: an amount (a decimal: a daily cap, a price) in a console form, in `ConsoleField` as `ConsoleInput` is — the same box (30 px on a desk, 44 on a phone, `0 10px`, radius 5, 13 px); `prefix` ("$") and `unit` ("tokens") inside the box at its left and right, 13 px text-2. A value outside `min`…`max` is refused under it in the console's critical ink ("At most 100 tokens."), the box's line critical; a refused save's message for the field (`Validation.MessageFor`) lands there by itself, `error` wins. ↑/↓ add or take a whole unit; `inRow`: a `SettingRow` draws its label. A console form takes no `DecimalField` (the Ui kit's 44 px field) | `ConsoleDecimalInput("Tokens / day", cap, min: 0m, unit: "tokens", help: "0 blocks.")` |
| `InlineMessage` · `LookTone` · `LookToneDot` | `text, LookTone tone (Ok · Run · Wait · You · Bad · Neutral), lead` | one line in a form or card: the tone's dot, `lead` strong, the tone's tint and ink, 13/1.45 `8px 12px` radius 8, a status region. On no board as such (§21's result lines' tones). `ToneStrip` stays the four-tone strip under a row | `InlineMessage("Saved 09:41.", LookTone.Ok, lead: "Saved.")` |
| `LookToast` | `message, undoLabel, onUndo, seconds = 5, onDismiss` | on no board: a confirmation at the bottom centre (at a phone's width: at the top, under the top bar, so it never covers the page's foot), white on the accent's deep ink (the editing banner's ground), 14/500 `10px 14px` radius 10, the dialog's shadow, an underlined Undo; owns its timer like the kit's `Toast` (named apart from it), held under the pointer, a status region. Render it always | `LookToast(notice, undoLabel: "Undo", onUndo: Undo, onDismiss: Clear)` |
| `ConfirmDialog` | `title, text, confirmLabel, onConfirm, onCancel, danger, cancelLabel = "Cancel", busy, reasonLabel, Binding<string>? reason, reasonRequired, reasonPlaceholder`; block = the body under `text` (what it touches); with `reasonLabel` + `reason` a `LookTextArea` under the body (`reasonPlaceholder`: an example reason while it is empty), and with `reasonRequired` the confirm button off — "Say why first." — until a reason is typed | §14's dialog values: modal, centred, 440 wide, radius 14, `20px 22px`, 18/700 title, 14/1.5 Ink 2, Cancel outlined then the primary — with `danger` the bad dot filled: the only red filled button of the look. Escape or a press outside is Cancel. The confirm button is disabled ("Working on it…") while any action is in flight and while `busy` (bind the page action: `busy: Retire.Pending`), so a slow action pressed twice runs once. At most 90 % of the window high: the title on top, the reason and the buttons at the bottom stay in view, `text` and the block scroll between them (P2.D19; `/dev/controls/confirm-long`, twelve rows) | `if (asking) { ConfirmDialog("Retire host-3?", "…", "Retire host-3", Retire, Cancel, danger: true); }` |
| `InheritedValue` | see source (`InheritedSetting`) | §5 setting with an inherited value | — |
| `SourceChip` · `SourceCell` · `SmallLink` | `words, SourceTone tone (Inherited · Narrowed · SetHere · Waiting)` · `+ linkLabel, linkTo`; `SourceToneOf(SettingFrom)` | PolicyFloorEffective "From" column, 11/600 `2px 7px` | `SourceCell("Narrowed here", SourceTone.Narrowed, "Product policy ›", to)` |
| (kit) `SettingSwitch` · `SettingChoice` · `SettingNumber` · `SettingAlways` · `SettingRow(name:)` in a `SettingsCard` | the kit's own (`osy docs ui-setting-controls`); a setting every change of which is kept with why passes `reason: why, reasonRequired: true` and a `saveWith` that hands `why` to the server; a reader's row that cannot say where its value comes from passes `chipless: true` | Foundry's settings; this look styles them through its Setting* tokens (`OsyrinV3`) | `SettingNumber("Session size", value: tokens, unit: "tokens", min: 1, max: 10000000, saveWith: Save, reason: why, reasonRequired: true)` |
| `ScopeLine` | `title, summary, help, helpTo, ruled` | §13 / canvas4 scope line over a section list | `ScopeLine("Patient Portal settings", "inherits … · 6 set here", help: "How scopes combine ›", helpTo: to)` |
| `HostChip` · `HostChipRow` (`HostEntry{Host,Mark,Removable}`) | `host, onRemove, mark, blocked` · `label, hosts, onRemove, blocked, note` | PolicyFloorEdit host chips; × only with `onRemove` and `Removable`; `mark` = "added" (accent line); blocked = bad tint in the bad line (on no board) | `HostChipRow("Allowed", hosts, onRemove: Remove)` |
| `HostsCard` · `HostAddField` | `title, tag, editable`; `slot add`, `slot why`, `slot under` · `label, value, onAdd, placeholder, ceiling, problemLead, problem` | PolicyFloorEdit "Outbound hosts" card and "Add a host" | see "A settings page" |
| `SettingTableRow` · `SettingTableGroup` · `SettingTableText` | `name, cols, key, keyTo, tag, tagText, spacing, dense, heads, stack = true`; `slot value`; `slot under`; block = the page's own cells · `label` · `text` | a settings table's row; `spacing` `Settings` by default (14 apart, `11px 20px`, under `CardTable(…, spacing: TableSpacing.Settings)`), `Tight` for PolicyFloorEffective (`minmax(0, 1fr) 82px minmax(0, 1.3fr) 130px`, 10 apart, `9px 20px`); `key` with `keyTo` a `KeyLink`, without it `KeyText` (no dead link); `tag` = the draft tag line under the value; `under` = set under the VALUE column across the rest of the row, 6 below the cells (PolicyFloor's "Where it is enforced" `Disclosure(…, ruled: false)`) | see "A settings page"; `/dev/controls/tables` |
| `CheckValue` · `CheckSentence` · `SentenceLink` | `label, value, outcome, outcomeTone, lead, rest, linkLabel, linkTo, link2Label, link2To` · `lead, rest, ok, links` | PolicyFloorEffective "Check a value against the floor"; the refused sentence under a field | `CheckValue("Check a host", typed, outcome: "Refused", outcomeTone: StatusTone.You, lead: "…", rest: "…")` |
| `ToneStrip` | `text, lead, StripTone tone (Warning · Bad · Ok · Plain), markdown` | orgsettings/Bundle tinted strip, 13/1.45 `8px 10px`. A new part: `AmberNote` stays the fit-width note, `CalloutBand` the planner band. `markdown`: the text is Markdown (open questions, decisions), drawn by the platform's `Markdown` under the lead, in the strip's ink; `/dev/parts/stages` | `ToneStrip("Any edit creates revision 4 …", lead: "Editing clears approvals.")`; `ToneStrip(questions, lead: "Open questions.", markdown: true)` |
| `ApprovalSlot` · `ApprovalRow` | `who, detail, given, agent` · `label`; block = slots, `slot action` = the primary | orgsettings/Bundle signatures row | `ApprovalRow { ApprovalSlot("Elif Demir", detail: "approved 3be1…7d4a · 09:41", given: true); ApprovalSlot("Second approver"); slot action { LookButton(…, kind: ButtonKind.Primary, small: true); } }` |
| `DiffLines` · `FieldDiff` | `List<string> lines ("+ …", "− …", "  …"), label` · `field, was, now, lead` | §6 Diff | `FieldDiff("Runs when", "every slice", "slices that touch app/booking/**")` |
| `DarkTextCard` | `text, label` | orgsettings/EditData "The signed text" (Plex Mono 12/1.6 on Ink) | `DarkTextCard(signedText, label: "The signed text")` |
| `AgentChip` | `words` | PolicyFloorEdit "Prepared by a factory agent" | `AgentChip("Planner agent")` |
| `EditBanner` | `text, onDiscard, onSave, second, discard, save` | the kit's `EditingBanner` with a second line (PolicyFloorEdit, EditData); an empty `discard` draws the save alone (a form whose parts keep themselves as typed) | `EditBanner("Editing draft v6 · revision 4", Discard, Review, second: "Saving clears Olive's approval on revision 3")` |
| `LookNumberInput` | `label, Binding<int> value, min, max, step = 1, unit, w = "100%", state, message, right, inRow, console` | §22's number field with its unit inside ("Never below [10] slots"): the §3 box, 44 high, the unit 14 Ink 3 12 in from its right; no browser steppers — ↑/↓ add or take `step` within the range; a value outside it is refused under the field ("At most 14 slots.", or `message`); `right` right-aligns the digits | `LookNumberInput("Never above", above, min: 1, max: 14, unit: "slots")` |
| `LookChoiceGroup<T>` | `label, Binding<T> value, T[] options = T.Members, Func<T, string>? detail, help, inRow, disabled, onChanged` | §22's radio (20 px ring, 2 px, the accent's with a 10 px dot when chosen, Ink 3's when not, 14 px words 8 after, 32 high); with `detail` each option is §21's option row (15/600 over its 13 Ink 3 second line, `10px 12px` in the soft line, radius 10; chosen: the accent line on the accent wash). `radiogroup` of `radio`s; the group is one Tab stop: ↑/↓ (←/→) move the lit option (a 2 px accent ring), Space or Enter chooses, Home/End; a press chooses | `LookChoiceGroup("Role", role, detail: RoleCarries)` |
| `SegmentedFilter` · `FilterSegment` | `label, List<FilterSegment> segments, selected, Action<string>? onSelect` · `Key, Label, Count, To` | canvas2/Backlog's "Show" strip: white in the control line, radius 10, segments `8px 14px` 40 high 14/500 Ink 2 with a control line between, the chosen 600 accent ink on the accent tint; the count a round 12/600 pill (the board writes "· 12" in the words). A segment with `To` is a link (`aria-current`), so a reload keeps it; without, a button (`aria-pressed`) handing its key to `onSelect`. On a phone it scrolls sideways inside itself | `SegmentedFilter("Show", [new FilterSegment { Key = "all", Label = "All", Count = "12", To = "/backlog/all" }, …], show)` |
| `SecretOnceCard` | `label, secret, onStored` | on no board: `DarkTextCard`'s mono-on-Ink box (13 px, broken anywhere) in a white card (radius 14, `16px 18px`) under `label` 15/600 and "Shown once — … it cannot be shown again." 13 Ink 3; Copy (outlined, "Copied" after; named "Copy <label>") and the one primary "I have stored it". The secret is text only — never a name, an address, a title or a log; `onStored` should forget it. Tests use an obviously fake value | `SecretOnceCard("Deploy key", made.Secret, Forget)` |
| `ConversationTurn` | `who, DateTime at, text, agent, initials, meta, pending, now`; `slot trace`; block = what follows the text | canvas2/PlannerPanel's turns: the head is who (`PersonChip` quiet, or `AgentChip` with `agent`) · `RelativeDate`, 12 Ink 3; `text` is Markdown drawn by the platform's `Markdown` (14/1.5); a person's turn is the bubble (wash, soft line, radius 10, `8px 10px`, at most 300, hung right), an agent's stands full width on the ground, parts 6 apart; `slot trace` between head and text (a `Disclosure(…, ruled: false)` of its calls); `meta` a 12 Ink 3 foot line (tokens, cost); `pending` = "Writing…" run chip, the text streamed, a status region. Named "who, when" for `within:`. `/dev/parts/conversation` | `ConversationTurn("Planner", at, reply, agent: true, meta: "1,840 tokens · $0.04") { slot trace { Disclosure("3 calls", ruled: false) { … } } ChangeList(…) { … } }` |
| `Composer` | `label, Binding<string> value, onSend, busy, placeholder, hint, maxRows = 8, sendLabel = "Send"`; `slot quiet` | PlannerPanel's foot: label 12/600 over a text area (14/1.5, `8px 10px`, radius 8, control line, 64 high) that grows with its text to `maxRows` lines then scrolls; the `hint` 12 Ink 3 left, `slot quiet` then the primary Send (small) right. **Enter sends** (the board's "Enter sends"); empty or `busy` it does nothing and Send is disabled ("Write something first." / "Sending…"). Plain text typed as Markdown: the platform's `MarkdownEditor` edits a stored document by owner address, so an unsent turn cannot use it; the sent turn is rendered with `Markdown`. Not the ReplyComposer kit's `Composer` (a desk's replies, with drafts and macros): an app using both qualifies one | `Composer("Ask, plan or change something", draft, Ask, busy: Ask.Pending, hint: "Enter sends")` |
| `ChangeList` · `ChangeBlock` | `title, label = "Proposed changes"`; `slot actions`; block = `ChangeBlock`s · `kind, ChipTone kindTone, key, keyTo, what, state, StatusTone stateTone, note, label, framed`; block = body; `slot actions` | PlannerPanel "3 changes to 3 items": the list is the accent line, radius 10, its head `8px 10px` on Accent tint 2 (title 12/700 accent ink, Reject all · Accept all right); each block `9px 10px` under a soft rule, gap 6: the kind chip 11/700 `2px 7px` on its tone's tint (Create ok, Update run, Acceptance accent — the board's purple is not a token), the key (small `KeyLink`), `what` 12 Ink 2, `state` a `StatusChip` at the right; then the body (`DiffLines`, a `PartTree`, an edit field), `note` 12 Ink 2, the actions right (small `LookButton`s, `announce`d "Reject: Update on BOOK-14"). `framed`: a block alone, in the accent line. Named "<kind> on <key>" | `ChangeBlock("Update", ChipTone.Run, key: "BOOK-14", keyTo: to, what: "done check") { DiffLines([…]); slot actions { … } }` |
| `ReadinessMarks` · `ReadinessMark` · `MarkState` | `List<ReadinessMark> marks, label = "Readiness", reasons` · `Name, State (Passed · Failed · NotJudged), Reason` | on no board as one row; in SliceDesign's and Brief's language: a 16 px mark (✓ white on the ok dot · ✕ bad ink on the bad tint · a dashed control-line ring) 5 before the name 12/600 (Ink 2 · bad ink · Ink 3), marks `4px 12px` apart, wrapping. A list named `label`, each mark an item read whole ("Tests named: failed — Criterion 2 names no test."); a failed mark's reason is its hover title (`tooltip` "Why <name> failed"); `reasons` also writes the failed reasons under the row, 12 bad ink. `/dev/parts/stages` | `ReadinessMarks([new ReadinessMark { Name = "Tests named", State = MarkState.Failed, Reason = "…" }, …], label: "Readiness of BOOK-26")` |
| `StageStrip` · `StripStage` | `List<StripStage> stages, int current, label = "Stages"` · `Title, What, State, Rule, To` | SliceDesign "Design stages": an ordered list in `repeat(auto-fit, minmax(min(200px, 100%), 1fr))`, 10 apart; each stage a card radius 12, `14px`, gap 8 — a 28 px dot (done ✓ on the ok dot, current its number white on the you ink, later Ink 3 in a 2 px control-line ring) 8 before the title 15/600 (a link with `To`), `What` 13/1.45, `State` 12/600 (ok ink · you ink · Ink 3), the current one's `Rule` as an amber note; done on the wash in the Line hairline, current white in a 2 px you-dot line (`aria-current`), later in a dashed control line. Stages before `current` are done | `StageStrip([new StripStage { Title = "Brief", State = "by Anna Berg · 10 Nov", To = to }, …], 1, label: "Stages of PORTAL-D3")` |
| `CheckStrip` · `StripCheck` | `List<StripCheck> checks, label = "Checks"` · `Title, Mark (MarkState), What, State, Rule` | on no board as such: `StageStrip`'s cards for checks that stand each on its own (outcome · tests named · fits a session · buildable), so a passing check after a failing one is drawn passed, never "later". Same grid and card (radius 12, `14px`, gap 8, 28 px dot, title 15/600, `What` 13/1.45, `State` 12/600); passed: ✓ on the ok dot, the wash in the Line hairline, ok ink; failed: ✕ in the bad ink on the bad tint, white in a 2 px bad line, bad ink; not judged: a dashed control-line ring, white in a dashed control line, Ink 3; `Rule` the amber note on any check not passed. A list named `label`, each check an item named "<title>: passed" / "failed" / "not judged". `/dev/parts/stages` | `CheckStrip([new StripCheck { Title = "Tests named", Mark = MarkState.Failed, What = "…", State = "Fails", Rule = "Name a test on every criterion" }, …], label: "Readiness of BOOK-26")` |
| `SidePanel` | `title, sub, onClose, closeLabel, mark, stick, w = "380px"`; `slot actions`, `slot head`, `slot foot`; block = the body | PlannerPanel's planner panel beside the main column (the page's row `gap 24px`, aligned to the top): white, Line hairline, radius 14, clipped, sticky 20 from the top, at most the window less 40; head `14px 16px 10px` over a Line 2 rule (title 17/600 h2 over `sub` 12 Ink 3, `mark` the hexagon 20 px; `slot actions` then the close `GlyphButton` "›", named "Close <title>", with `onClose`; then `slot head`: what it sees, its tabs); the body `12px 16px`, gap 12, scrolls inside (`stick`: held at its end, for a conversation); `slot foot` `10px 16px 14px` over a Line 2 rule (the composer). A region named `title`; under Tablet a full sheet (fixed, no corners, a modal dialog, "×", Escape closes) — render it at the page's level there. `/dev/parts/conversation` | `SidePanel("Planner", sub: "…", onClose: Hide, mark: true, stick: true) { slot foot { Composer(…); } ConversationTurn(…); }` |
| `PartTree` · `TreePart` | `label, List<TreePart> parts` · `Level, Kind, ChipTone KindTone, Key, Title, Line, To` | Brief "Place as work" (an arc with its first slices) as a tree a `ChangeBlock` can hold: rows `6px 0` under a soft rule, 18 px a level behind a 2 px soft guide; the kind chip 11/600 `1px 6px` radius 5 on its tone's tint, the key mono 12/600 (`KeyLink` with `To`), the title 14; `Line` 12 Ink 3 under it. A list named `label`, items at `aria-level` Level + 1, named "<kind> <key> <title>". Chosen over `CardTable` + `TableGroupBand`: it has levels, not columns, and sits in a 350 px change block where a head and columns do not fit | `PartTree("Proposed parts", [new TreePart { Kind = "Arc", KindTone = ChipTone.Accent, Key = "TRIAGE-DUAL", Title = "Two reviewers" }, new TreePart { Level = 1, Kind = "Slice", Title = "Second reviewer", Line = "140k of 160k" }])` |

`ActionButton` (org-policy's name) is `LookButton`: one primary per band, the rest outlined.

## The app frame

`LookShell(AppChrome chrome, bool padded = true, List<LookSwitcherGroup> switcher = null) { Outlet(retain: 8); }`
(`model/LookShell.osy`) is the frame around every page, as the owner's mock-up draws it
(`docs/designs/foundry/owner-refined-plan-mockup.png`, L2a) and the boards' rail does (canvas1 Navigation, the rail on
every canvas4 page, canvas1 OverviewPhone): the rail (248 px) at the window's left edge, full height and sticky, on the
ground with a 1 px hairline on its right; the page region ("Page") right against it, **no top bar**. `padded` (default)
insets the page `PageInset()` = 24 px (40 below); `padded: false` for an app whose pages stand in the Ui kit's
`WorkspaceFrame`, which pads them 24 px itself (Foundry). It stands in place of the Ui kit's `SidebarShell` and takes
the same `AppChrome`, so an app's layout changes one word. Sample: `/dev/shell`, `/dev/shell/long`, `/dev/shell/groups`
(tests in `tests/Shell.test.osy`).

| The app passes | Drawn as |
|---|---|
| `Product`, `Tagline`, `Mark` | the brand block: a 26 px ink tile with the mark (the first letter of `Product` without one), the name 15/700, a 12 px Ink 3 line |
| `Apps` (`AppLink{Label,To,Icon,Current}`), `Home` | with any, the brand block is a button that opens the list of places, "Home" first, the current one on the tint |
| `switcher` (`LookSwitcherGroup{Label,Apps}`, optional) | more places in that list, after `Apps`, each group under its label 11/600 uppercase Ink 4 and named by it ("Platform", then "Your organisations"); any group makes the brand block a button. Without it the list is `Apps` alone |
| `Nav` (`NavItem{Label,To,Icon,Badge,BadgeTone,Exact,Kind,Children}`) | rows 38 high, `0 10px`, 14 px, a 16 px icon; `Kind = NavKind.Section` is a 12/600 uppercase label over its `Children`; a row with `Children` has a small opener and shows them, set in, while one is in view; `Badge` is the kit's `Badge` at the right |
| `FootNav` | rows under the hairline at the foot, above Collapse |
| `User` (`ShellUser{Name,Secondary,Initials,Menu}`) | the person at the foot (26 px round initials, 13/600 over 12 px Ink 3); with `Menu` (`MenuAction{Label,To or OnPress,Icon,Divided}`) a button, named as the person, that opens the menu above it. A light/dark switch is a `MenuAction` whose action calls `Theme.Toggle()` |

What it does itself: the row in view is lit (the longest matching address, `aria-current`); the Collapse row and the `[`
key make it a 64 px strip of 44 × 40 marks, remembered per browser (below 1100 px it is always the strip); every
section stands open with its rows in view, however long the rail, and a reader folds one by its label, which then shows
how many rows it holds; when the window is shorter than the rail, only the rows between the brand block and the foot
scroll; a row asks before it leaves a page with unsaved edits (the kit's `LeavePrompt`). Below 768 px: a white header (a
44 px "Open navigation" button, the page's `[Title]` 17/700) and the rail as a menu card over the page, every row 44
high and labelled, the person and their menu last, "Close menu" at the top.

Not drawn yet: the Search row with its `/` keycap, a name beside a mark on the strip, the ink count pill of the Inbox row
(a `Badge` in its tone is drawn), the dot on the phone's menu button.

## The signed-out frame

`LookSignedOutFrame(string heading, string headline, Textures artwork, string brand, string lede = "", string sub = "",
string logoUrl = "") { form; slot below { … } }` (`model/LookSignedOutFrame.osy`) is every signed-out screen: one wide
card (75rem, radius 20, the lifted shadow) on a pale lavender page, the artwork panel on its left (58 %) and the form on
warm paper on its right. The panel holds the brand row (a 36 px "O#" tile, or the `logoUrl` image in its place, and
`brand` 22/600), the `headline` 52/700 white, one line per `\n`, and `lede` under it (18 px; empty draws none). The
form column (27rem) holds `heading` 32/700, `sub` muted, the default slot (the form) and `slot below`, an optional box
under the form. The artwork is shaded under the words as the osyrin.com front page shades it: across the left on the
wide panel, fading out over its lit right side. Below 768 px the panel is a 13rem band above the form, shaded from the top.
`artwork` is the look's own picture, `textures/signed_out_hero.jpg`. A package's textures reach the app that uses it
as its icons do (`textures "textures/*.jpg";` in `package.osy`): `Textures.SignedOutHero` is there without the app
carrying the file, an app's own texture of the same name wins, and two packages shipping one name is an error. Its tokens are
the look's: `Colors.SignedOut*`, `Radius.SignedOutCard`, `Length.SignedOut*`, `Shadow.SignedOutLift`,
`FontSize.SignedOutHeading*`. Foundry's `SignedOutFrame` passes its own brand and headline. Sample: `/dev/signed-out`
(`tests/SignedOut.test.osy`).

A page reached from the front page's hero (osyrin.com: Admin's `/apply`) says what the hero says, the way it says it,
with four more arguments, each empty by default and then drawing nothing: `eyebrow`, an 11/600 uppercase line over the
headline; `headlineAccent`, more headline lines after `headline`'s (one per `\n`) in the artwork's second ink,
`Colors.OnSignedOutHeroAccent` (the front page's lavender); `tagline`, a 24/600 closing line under the lede (20 on a
phone) and `taglineAccent`, its last words in that ink. e.g. `LookSignedOutFrame("Apply for the beta", "Every team can
build with AI.", Textures.SignedOutHero, "Osyrin", eyebrow: "Private beta", headlineAccent: "Every app follows your
rules.", lede: "…", tagline: "Software you own.", taglineAccent: "Run for you.") { … }`. Sample: `/dev/signed-out/hero`.

## Chart series

`Colors.LookSeries1` … `LookSeries7`, in the order a chart takes them: `LookSeries1` is the accent (`Colors.Primary`), `LookSeries2`
and `LookSeries3` two lighter shades of it, `LookSeries4`–`LookSeries7` four greys from deep to light. Each stands at 3:1 or more on
the ground and on a panel, light and dark. The status tones are not series: a chart uses them only where a series IS a
state. Tests in `tests/Series.test.osy`.

## Console charts, plan keys and row notes

The console's own charts and plan-item keys, from Foundry's pages (reconciled into this kit before the split). Each
part's full rules are in its source's doc comment.

| Part | Props | What it draws |
|---|---|---|
| `ConsoleBarChart` | `List<ConsoleBarPoint> points, unit = "", stacked = false, last = "", height = 96, label = "Chart", List<ChartReferenceLine> references = null` | bars over time with a value axis of round ticks from 0 (0–100% for `unit` "%"), each said in the unit by `ChartValueSaid` ("50%", "$2", "2 h", "150k"), the top tick with the unit's word ("6 builds"); a bar per point and series in the `ChartSeriesNumber` colours, side by side or `stacked`; a point with no value is a dashed "no data" gap, never a bar of 0; `references` are dashed threshold rules. In a test the axis is the group "<label> scale", the time labels "<label> time" |
| `ConsoleLineChart` | `List<ConsoleBarPoint> points, unit = "", last = "", height = 96, label = "Chart", thin = 5, List<ChartReferenceLine> references = null` | the same scale, gridlines, time axis and series colours as `ConsoleBarChart`, with points joined by a line; each point says its sample size ("80% · n=5"), a sample below `thin` is a hollow point in faint words and a key says so; a gap breaks the line. In a test the plot is the group "<label> line" (each point a list item "16 Nov: 50%, n=2") |
| `ConsoleBarPoint` · `ChartReferenceLine` | `Label, List<double?> Values, List<int?> Samples` · `Label, Value, Critical` | a chart's point (an empty value is a gap) · a threshold at `Value` in the chart's unit, amber, red when `Critical`; the scale reaches the highest one |
| `ConsoleKeyChip` | `key, ConsoleTypeKind kind, level` | a plan item's key as a chip in its level's colours (`ConsoleTypeBadge`'s): saga white on the accent, arc on the soft accent, slice outlined, bit on the neutral ground; 20 high, mono 12/600, at least 48 wide; read as "Saga FOUNDRY-1", so colour is never the only sign of the level |
| `ConsoleKeyChipLink` | `key, to, ConsoleTypeKind kind, level` | `ConsoleKeyChip` in a link that opens the item: one focus stop with a ring; pressed in a test by its key |
| `ConsoleKeyLink` | `key, to` | a key in a line of words that opens the item it names: mono, the console's accent, one line, underlined on hover; `Ui.Click("FOUNDRY-10")` |
| `NoteWords` · `NoteLine` | `words, key = "", to = "", rest = ""` · `of, words, key = "", to = "", rest = ""` | a row's note ("Waits on FOUNDRY-10"): the words, the key as a `ConsoleKeyLink` (plain where `to` is empty) and the words after it · the same on its own line under a row, small, in the quiet ink (never a warning's amber), wrapping between words; the region "Note of <of>". `ConsoleTreeRow`'s `note` draws it |

## The page frame

**A page with a band** (L2a, the mock-up): `PageBand(title, trail, situation, tabs, current, tabsLabel) { slot chips;
slot primary; actions }` first, outside any column — white from the rail to the window's right edge, the breadcrumb,
the 28/700 title, key chips, other actions at the title's right, the `TabBar` on its foot with the one primary action on
the tabs' line (regions "<title> header", "<title> tabs line"); then the body in a `PageColumn`, or `PaneLayout` (below).
**Pages that share tabs share one head.** `ConsolePageHead`, `PageBand` and `AreaHeader` are `[Section]` parts
(`osy docs ui-section`), keyed by their tabs' addresses (`TabItem.To`): when the app moves between two pages whose heads show
the same tabs, the head on screen is kept — the same DOM, patched to the arriving page's title, `current` and slots —
so a tab switch redraws only the body, and the page's test can say `Assert.RedrawsOnly("<the tab body>")`. A head with
no tabs (`ConsolePageHead`, `PageBand`) is keyed by its trail's links instead: the pages under one object or area — an
application's Overview, Access, Logs, each in the rail, not in tabs — share it, the breadcrumb kept and the title
patched. That includes a workspace's top level: pages whose trail has the same links above them (Zones, Plans,
Organizations, an organisation's overview — all under "/") share one head too, patched to each. A head with neither is
built per page. Nothing to write in the page: use the part as it already is.
`StatStrip(label, List<StripStat> stats)` (`StripStat{Label, Value, Sub, Tone, To, Mono}`) is a page's figures as one
white card of equal cells split by inset hairlines (a phone: one under another). `CardTable(…, quietHead: true)` draws
the head as words on white with a hairline under.

**Panes** (L2b, the two-surfaces boards `designer.dc.html`, `plan-part.dc.html`): see "Design and plan parts" below.

Every older page board draws the same frame at 1440 px: 20 px in on the ground, the app's rail (248 px) 24 px before the main
column (`PageColumn`), which grows from 640 px to at most **1120 px** and holds its parts **18 px** apart (a Slice-like page with the
planner tab: 44 px on the right, 24 off). In order: `ObjectHeader` (`lead` for a product or a release, `item` for an
item), `TabBar`, then — when the page has one — the summary row (14 Ink 2 words with a `KeyLink`, its buttons on the
right, wrapping `8px`), a `CalloutBand` or a `DecisionBand`, then the cards. Headings: the page title is the h1, a
card's title an h2, a foot card's an h3. A floor status is `StatusChip(…, StatusTone.Floor)`, not `KitTone(Floor)`
(that one draws Ink 2).

**The object page's two columns** (Slice): a row `gap 18px`, wrapping, aligned to the top — the left column
`flex: 999 1 540px`, the right `1 1 300px`, each a stack of cards **14 px** apart. At 1440 the main column is 1060 wide:
742 left, 300 right.

**Card grids**: `CardGrid(min, gap)` — Release's summary cards `300px`/16, its foot cards `240px`/16, Slice's side cards
`250px`/14. Every card in a row is as tall as the row.

**A row in a card** (`CardTableRow`, `SettingTableRow`): a padded row under a Line 2 rule, its grid of cells on the
table's columns (centred), then its `under` slot 6 below. Its gap and paddings are its table's, from `TableRhythm` —
the head reads the same numbers, so a head sits on its columns without a caller correcting widths: `Plain` `10px 20px`
12 apart, `Roomy` `12px 20px` 12 apart, `Settings` `11px 20px` 14 apart, `Tight` `9px 20px` 10 apart; `dense` `7px 20px`
over a 20 px line, 34–36 high, for a long list (Admin's "a lot of info but not long pages you scroll forever"). In a
`CardTableRow` the `under` slot holds the amber note and the disclosure, each set in 46 px (34 px rank column + 12 px
gap), so they sit under the title; in a `SettingTableRow` it is set under the value column. A title cell is a
`Stack(gap 2)`: a `Row(gap "4px 10px", baseline)` of `KeyLink` + `RowTitle`, over a 13 px Ink 3 line. A chip in a cell
sits in a `Box` so it keeps its own width; a right-aligned column's cell sets `textAlign: TextAlign.Right`, as its head
does by `aligns`. A table wider than its card (`minWidth`) scrolls inside it: a test asserts it with
`Assert.Wider("<title> table", "<title>")` and `Assert.Inside(<a first-column cell>, "<title>")` beside
`Assert.NoOverflow()`, which passes a table that scrolls in its own box and fails a control or words the card cuts off.

**A table is a table** (P2.D15): `CardTable` is a `table` named by its `label` (`TableCard`'s is `title + " table"`),
its head ONE `row` of `columnheader`s (a sortable one says `aria-sort`), each `CardTableRow`/`SettingTableRow` a `row`.
The look cannot see the caller's cells, so **the caller writes them as cells**: each direct child of a row's block is
ONE cell, in the head's order, and says so — `role: UiRole.GridCell` on a `Text`/`Box`/`Stack`/`Row` that is already
there (no wrapper, no pixel moves), or `TableCell { … }` around a part that takes no `role:` (`KeyLink`, `StatusChip`;
`TableCell(align: TextAlign.Right)` for a right-hand column). `SettingTableRow`'s own two cells (name, value) are cells
already; `slot under` is not a cell. A child that is not a cell is not read under its column, and `Assert.Cell` refuses
it. With that, a test asks `Assert.Items("<label>", n)` (the rows a `foreach` drew — the head is not counted) and
`Assert.Cell("<first cell's words>", "<column head>", "<words>", within: "<label>")`.
Group bands (`TableGroupBand`, `SettingTableGroup`) are NOT rows: a band as a row would be one cell under a head of
several, which the driver reports as a broken row and a reader announces as data. A band is a labelled group in the
table. But `Assert.Items` counts every row a `foreach` stamps, so a loop over GROUPS stamps the groups too: loop over
a value with no `Id` (a class without one, as the head does over `HeadCell`) and only the data rows count.
`/dev/controls/tables` "Contents", "Hosts", "Grouped contents"; test
`A_look_table_counts_its_data_rows_and_reads_a_cell_by_column_name`.

**A row's action is announced with its row**: `LookButton("Resolve", Resolve, kind: ButtonKind.Inline, announce:
"Resolve " + alert.Title)` shows "Resolve" and is read, and pressed by a test, as "Resolve Disk full on host-3";
`MenuAction("Remove", Remove, announce: "Remove " + key)` the same in a menu. `announce` defaults to the words; begin
it with them (voice control says what it sees). `LookButton`'s `name` is the older spelling; `announce` wins.

### Worked example: a Plan-tab release card (from `/dev/page-parts/plan`)

```osy
SectionCard("Patient Portal 2.4 · Thu 12 Nov", eyebrow: "Next", sub: "Expected Thu 12 – Mon 16 Nov (80 %) ·",
            linkLabel: "Release plan ›", linkTo: "/releases/2.4", band: true) {
  slot sub { StatusChip("At risk · headroom smaller than the uncertainty", StatusTone.You); }
  Box(px: "20px", pb: "10px") {
    Disclosure("Why this forecast · updated 09:38 by the planner") { … }
    Disclosure("Predicted token use") { … }
  }
  CardTable("Arcs of Patient Portal 2.4", "34px minmax(0, 1fr) 190px 150px", ["#", "Arc · your priority", "Progress", "Release"], roomy: true) {
    CardTableRow("34px minmax(0, 1fr) 190px 150px", roomy: true, label: "BOOK-SLOTS Show free slots") {
      RankNumber(1);
      Stack(gap: "2px", minW: "0") {
        Row(gap: "4px 10px", wrap: Wrapping.Wrap, align: Align.Baseline) { KeyLink("BOOK-SLOTS", to); RowTitle("Show free slots", to); }
        Text("Patients see every free slot …", fontSize: FontSize.Label, color: Colors.Ink3, lineHeight: "normal");
      }
      ProgressShares(2, 1, 1, 0, "2 merged · 1 running · 1 waiting");
      Box { PlainChip("2.4 · set on the arc"); }
      slot under {
        AmberNote("Will merge after UIKIT-8 because …");
        Disclosure("Details and predicted token use", ruled: false, indent: true) { Text("Predicted token use 270k …"); }
      }
    }
  }
  CardFoot("Also in 2.4: ") { KeyLink("BOOK-10", to); Text(" Visit history pages by 25 …", whiteSpace: WhiteSpace.PreWrap); }
}
```

### Worked example: a Release "Contents" card (from `/dev/page-parts/release`)

```osy
SectionCard("Contents · 4 slices", linkLabel: "Open Plan ›", linkTo: "/releases/1.8.1/plan") {
  TableGroupBand("Saga", "Claims Desk upkeep", key: "UPKEEP", keyTo: …, innerKey: "UPKEEP-ROUND", innerKeyTo: …,
                 innerTitle: "Rounding fix (INC-15)");
  CardTable("Contents", "110px minmax(0, 1fr) 200px 90px 100px", ["Slice", "Title", "Why in", "State", "What’s new"]) {
    CardTableRow("110px minmax(0, 1fr) 200px 90px 100px", label: "UPKEEP-41") {
      KeyLink("UPKEEP-41", to);
      Text("Round claim totals half-up in the summary");
      Box { PlainChip("From the arc"); }
      Box { StatusChip("Done", StatusTone.Ok); }
      Text("Entry ✓", fontSize: FontSize.Label);
    }
  }
}
```

The group band comes before the table's head, as the board draws its ONE group. With several groups the head comes
first, once, directly under the card's title, and every band among the rows (as ReleasePlan) — a first band outside
the table put the live Release page's heads under it (P2.D19).

## Design and plan parts (L2b)

Drawn from `docs/designs/foundry/designer.dc.html` and `plan-part.dc.html`; samples `/dev/panes` (the design page) and
`/dev/plan-part` (a plan part); tests `tests/PageBand.test.osy`, `Outline.test.osy`, `Talk.test.osy`, `PlanPart.test.osy`.

| Part | Props | What it draws |
|---|---|---|
| `PaneLayout` | `label, first, second, third = "", height = "", bottom = "40px", secondTitled = true, secondWash = true, measure = "", sheet = "", Action<string>? onSheet, thirdStartsFolded = false`; slots `first`, default, `third`, `firstTools`/`secondTools`/`thirdTools`, `firstFoot`/`secondFoot`/`thirdFoot` | Draw right after `PageBand`, not in a `PageColumn`: one white surface from the band's foot to the window's foot and from the rail to the right edge (`bottom` the page's foot inset: 40 under `LookShell`, 24 in `WorkspaceFrame`), panes split by hairlines, heads one 52 px line (name 17/700 h2, tools, fold «/»), each body scrolling, each foot staying. Side panes 248 and 300 px, fold to a 40 px tab ("Fold/Open <name>"; `thirdStartsFolded` starts the third as its tab, as the plan part's "Ask the planner"), from Laptop a handle with a grip ("Resize <name>", drag or arrows, 200–560), remembered. `secondTitled: false` the middle head holds only its tools; `measure` centres the middle body and foot ("680px"). Phone: the middle only; a side pane is a sheet (dialog "<name>") opened by `PaneSheetButtons(first, third, onOpen)` in the band, the page holding `sheet`/`onSheet`. Regions "<name> pane" |
| `PartLevel` · `LevelMark` | `Saga, Arc, Slice, Bit` · `level, word = false` | the level's mark by SHAPE: saga ◆ ink, arc ring (run dot), slice square (ok dot), bit dash (Ink 3); `word` adds "Arc" 600 Ink 3; without it the mark is a graphic named by its word and shows it on hover |
| `OutlineTree` · `OutlineEntry` | `label, List<OutlineEntry> entries, selected, Action<string> onSelect, List<string>? menu, Action<string,string>? onMenu` · `Id, Depth, Level, Title, Key, Fresh, DependsOn, Count, Tone (StatTone), ToneWords` | the tree, entries in order: guides 14 px a level, open/close ("Open/Close <title>"), the level mark, the title 14/20 up to two lines (Key mono before it), quiet marks right (link mark with `DependsOn` on hover, count, the new dot, a status dot); selected: tint, 3 px accent bar, 600 accent ink; "···" ("More for <title>") on the selected or hovered row opens `menu` ("<words>: <title>"). One Tab stop: ↑/↓, → open / first child, ← close / parent, Home/End, Enter selects. A listbox `label`, rows options "<Level> <Title>" |
| `OpenQuestions` · `OpenQuestion` · `OutlineLegend` | `questions, Action<string> onOpen, label = "Open questions"` · `Id, Text, Level, RowTitle, Choice` · `fresh, depends` | the outline's foot: "Open questions · N" 14/700, each a pressable question over its row's mark and title (" · a choice"); the legend of the new dot and the link mark |
| `TalkTurn` | `who, DateTime at, text, pending, now`; block after the text | a turn as plain text: "You · 12 min ago" 13/600 Ink 3, the Markdown text 15/1.65; no avatar, no bubble. Group "<who>, <when>" |
| `CaptureLine` · `LevelRef` · `SentenceAction` | `label, Action? onMove, Action? onUndo`; block = the sentence · `level, title, Action? onOpen` · `words, onPress, name` | a 6 px accent dot and one 13 Ink 3 sentence written with `Text` and `LevelRef` ("Added under ◯ Arc Waiting list: ■ Slice Expire …"), then "· Move · Undo" ("Move: <label>", "Undo: <label>") |
| `DecisionCard` · `ChoiceOption` | `question, List<ChoiceOption> options, Action<string> onAnswer, Action? onTalk, whoActs, meta, answerLabel = "Answer"` · `Key, Label, WhatHappens, For, Against, Recommended` | v3's template in full: question 17/700, each option a row (ring, label 15/600, "Recommended" chip, what happens, For · / Against ·), the recommended on the accent wash with a 3 px bar, NOTHING preselected, "Who acts: …", the answer disabled ("Choose an answer first.") until a choice, "Talk it through". Region `question`, radio group "Answers: <question>"; ↑/↓ and Space/Enter |
| `TalkComposer` | `label, Binding<string> value, onSend, about, aboutLevel, Action? onClearAbout, clearLabel = "Talk about the whole design", busy, placeholder, maxRows, sendLabel` | white in the accent line, radius 12: the "About: ◯ Arc **Waiting list** ×" chip (group "About <title>"), a frameless 15/1.6 text area named `label`, Send (primary) at the right; Enter sends. Group "<label> composer" |
| `AreaTabs` · `AreaTab` · `DoneMark` | `label, List<AreaTab> areas, current, Action<string> onSelect` · `Key, Label, Done, Count` · `done, size = 16, doneWords, openWords` | the areas as a vertical tab list 188 px (phone: full width): the done mark (filled ✓ ok / hollow you ring), the name 14, the count 13 Ink 3; current on the tint. Tabs "<Label>, agreed/open" |
| `AreasAgreed` | `List<AreaTab> areas, noun = "areas"` | one 28 × 6 segment per area (ok dot / wait tint) and "4 of 6 areas agreed" 14/600; a meter |
| `TrailStep` · `SupersededLine` | `title, done, last = false`; block · `when, was, why` | a decision on a vertical line: 14 px open ring / done check, the question 15/600, the block (chosen 14/1.55, meta, "Answer it ›"); "Supersedes, <when>: ~~was~~ — why" 13 Ink 3 behind a 2 px rule. Group `title` |
| `SummaryBlock` · `SummaryEntry` | `label`; block · `text, name, suggested, editing, Action? onEdit`; block = the editor | "Goal", "Desired outcomes" 14/700 over entries: Markdown 14/1.55 with the wash on hover, a dashed "suggested" chip (its meaning on hover), a quiet "Edit" ("Edit <name>"); while `editing` the page's `MarkdownEditor` stands in its place in the accent line |

## Kit controls

Kit parts that were built to the board and come out as the board under the base theme's tokens — use them directly:
`Badge` (tones; through `PlainChip`/`ReleaseChip`/`KitTone`), `Tabs`/`Tab` (through `TabBar`; §12 measured 0.12 %), and,
measured in the gallery (`components.json`): `ScopeHeader` + `ScopedSetting` (§5, 0.4 %), `EditingBanner` +
`PendingChange` (§5, 0 %), `HistoryRow` (§6, 0 %), `SectionList`/`SectionGroup`/`SectionLink` (§13, 0 %, with the
look's `ScopeLine`), `ImpactPanel`/`ImpactDialog` (§14, 0.8 %, with `Bullets`), `Drawer` (§15, 0.4 %). A one-line
`SectionList` shares §13's row tokens (34 px rows); canvas4's settings list draws 36 px rows (platform request in
`findings/P2.D1b-result.md`).

Kit controls a page must NOT use directly, because they keep the kit's own shape:

| Kit control | Use instead |
|---|---|
| `Button` (38 px, Border hairline, filled danger, faded disabled) | `LookButton` |
| `DataGrid` (a spreadsheet) | `CardTable` + `CardTableRow` |
| `Card` (padded all round, shadow, 16 px title) | `SectionCard`, `SmallCard` |
| `PageHead` / `PageTitle` (no breadcrumb, chips or situation) | `ObjectHeader`, `AreaHeader` |
| `EmptyState` (centred, muted) | `EmptyPanel`, `ErrorPanel` |
| `Facts` / `Fact` (rows, not a grid) | `FactsGrid` + `FactCell` |
| `Badge` for a status (no dot) | `StatusChip` |
| `MoreText` as a disclosure | `Disclosure` |
| `Field` / `SelectField` / `TextAreaField` (the kit's own shape) | `LookInput` / `LookSelect` / `LookTextArea` |
| `Dropdown` · `Checkbox` · `Switch` (Border hairline, kit sizes) | `LookEnumSelect` (over strings `LookSelect`, over rows `LookRefPicker`) · `LookCheckbox`/`LookCheckboxBound` · `LookSwitch` |
| `Select` (the browser's select) · `SelectField` without `detail:` | `LookSelect` — declared in `package.osy` (`replaces`), so `osy lint` says it: below |
| `Toast` (a corner, kit tones) · `ConfirmStep`/`Dialog` for a yes-or-no | `LookToast` · `ConfirmDialog` |

### What it replaces

`package.osy` declares the parts the look draws instead of others' (`replaces "Select" { use "LookSelect"; }`, `osy docs
ui-kit-replacements`): an app on the look whose page draws the browser's select (`Select`) or `SelectField`'s plain face
hears `osy lint`'s `ui-kit-replaces-this-part` (SHOULD), naming `LookSelect`. `SelectField(…, detail: …)` draws its own
list and is not reported. A part the look adds that replaces another gets its `replaces` entry in the same slice.

## A settings page

The parts serve ANY app's settings, not one organisation's policy. Scope is three inputs, never a scope value inside a
part: **where the value comes from** (the kit's `SettingFrom` + a provenance sentence, and a `SourceChip`), **the edit
handlers** (absent = shown only: no ×, no add field, no edit block), and **the tag text** ("Draft v6", or none). The
frame is canvas4's: `AreaTitle`, `ScopeLine(ruled: false)`, then a row 24 apart of the kit's `SectionList(oneLine:
true)` and the section (16 apart): its head, its cards. Replica: `/dev/page-parts/effective` (PolicyFloorEffective,
measured in `pixel-budgets/page-effective.json`); every part with sample content at the foot of `/dev/components`.

### Worked example: an application's outbound hosts (shown, narrowed, and the check)

```osy
Stack(gap: 0, bg: Colors.Surface, borderW: 1, border: Colors.CardLine, rounded: Radius.Panel, overflow: Overflow.Hidden) {
  SettingTableGroup("Outbound hosts");
  SettingTableRow("*.vercel.com", "minmax(0, 1fr) 82px minmax(0, 1.3fr) 130px") {
    slot value { StatusChip("Allowed", StatusTone.Ok); }
    SettingTableText("Deploy preview and production");
    SourceCell("Inherited", SourceTone.Inherited, "Policies ›", "/policies");
  }
  SettingTableRow("nuget.org", "minmax(0, 1fr) 82px minmax(0, 1.3fr) 130px") {
    slot value { StatusChip("Refused", StatusTone.You); }
    SettingTableText("From its .egress file");
    SourceCell("Narrowed here", SourceTone.Narrowed);
  }
}
SettingsCard("Network", description: "Where the app may call out to.") {
  SettingSwitch("Only allowed hosts", value: onlyAllowed, lockedBy: "Northwind Health's policy · version 5",
                lockedWhy: "Patient data stays with named processors.");
}
HostsCard("Outbound hosts", editable: canEdit) {
  HostChipRow("Allowed", allowed, onRemove: canEdit ? Remove : null);   // inherited entries: Removable = false
  HostChipRow("Blocked", blocked, blocked: true, note: "Blocked for every application of Northwind Health.");
  slot add { HostAddField("Add a host", adding, Add, problemLead: refusedLead, problem: refusedRest); }
}
CheckValue("Check a host", typed, outcome: verdict, outcomeTone: verdictTone, lead: verdictLead, rest: verdictRest);
```

A reader with no edit right gets the same rows with no handlers; somebody with no role gets the `PermissionNote`.
An organisation's draft adds the tag (`SettingTableRow(…, tag: "Draft v6", tagText: "adds play.googleapis.com")`),
`EditBanner(…, second: "Saving clears …")`, a `ToneStrip` under the card, `ApprovalRow` and `DiffLines`/`DarkTextCard`.

## Busy actions (L3)

`LookButton` and `ConfirmDangerButton` say when what they started still runs, with nothing from the page: off, and
`busyLabel` ("Working…" without one) in place of the words, said once through a polite live region. The name stays the
words. Pressed plus `Pending.Any`, or `busy:` from outside. The spinner is the platform's own (`.osy-spinner`, in the
words' colour, once a press has waited on the server for 200 ms; still under reduced motion); the look draws none of its
own, so a slow press shows one busy mark.
`ConfirmDialog` adds `busyLabel`, `busySaid` (one quiet line under the body after about a second; none without it)
and `canConfirm` (off for a reason that is not "busy"; do not fold it into `busy`). While busy, Cancel, Escape and a
press outside do nothing. e.g. `ConfirmDialog("Take D-4 to the plan", "…", "Take", Take, Cancel, busy: Take.Pending,
canConfirm: Takeable(), busyLabel: "Taking…", busySaid: "Taking 120 rows to the plan…")`.

A busy button keeps its kind's colour (L3b): `LookButtonFace` has a `busy` variant (Opacity 0.85, default cursor) and the
grey disabled look (`Colors.Line2` fill, `Colors.Border` line, `Colors.Ink4` text) moved to a `plain` variant that applies
only when the button is off and not busy (so Cancel in a busy dialog stays grey). A busy primary keeps `Colors.Primary`
with `Colors.OnPrimary` text; `ConfirmDangerButton` keeps `Colors.BadDot` (its Disabled opacity 0.6 becomes 0.85 when busy,
no hover change). The platform's spinner takes the button's text colour, so it reads white on a primary or the red confirm.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Look;
using OsyrinLookTests;

[Test]
void The_phone_bars_menu_mark_is_in_the_rails_ink_and_shows_on_the_rail_in_both_themes() {
  // The mark's three bars are drawn in the button's ink (Colors.ConsoleRailInk); a non-text icon needs 3:1 (WCAG 1.4.11).
  Assert.Contrast("Colors.ConsoleRailInk", "Colors.ConsoleRail", 3);
  Assert.Contrast("Colors.ConsoleRailInk", "Colors.ConsoleRail", 3, mode: "dark");
  Assert.Contrast("Colors.ConsoleSurface", "Colors.ConsoleRail", 3);
}
```

From [`tests/ConsoleShell.test.osy`](tests/ConsoleShell.test.osy).

```osy
using Osysharp.Look;
using OsyrinLookTests;

[Test]
void Relative_words_hold_at_every_boundary() {
  var now = new DateTime(2026, 11, 12, 15, 0, 0);
  Assert.Equal("just now", RelativeWords(now, now));
  Assert.Equal("just now", RelativeWords(now.AddSeconds(-59), now));
  Assert.Equal("1 min ago", RelativeWords(now.AddSeconds(-60), now));
  Assert.Equal("59 min ago", RelativeWords(now.AddSeconds(-3599), now));
  Assert.Equal("1 h ago", RelativeWords(now.AddSeconds(-3600), now));
  Assert.Equal("15 h ago", RelativeWords(new DateTime(2026, 11, 12, 0, 0, 0), now));
  Assert.Equal("yesterday", RelativeWords(new DateTime(2026, 11, 11, 23, 59, 59), now));
  Assert.Equal("yesterday", RelativeWords(new DateTime(2026, 11, 11, 0, 0, 0), now));
  Assert.Equal("10 Nov", RelativeWords(new DateTime(2026, 11, 10, 23, 59, 59), now));
  Assert.Equal("1 Jan", RelativeWords(new DateTime(2026, 1, 1, 0, 0, 0), now));
  Assert.Equal("31 Dec 2025", RelativeWords(new DateTime(2025, 12, 31, 23, 59, 59), now));
  // Just past midnight, a moment before it is minutes ago, not yesterday; an hour before it is yesterday.
  var night = new DateTime(2026, 11, 12, 0, 10, 0);
  Assert.Equal("20 min ago", RelativeWords(new DateTime(2026, 11, 11, 23, 50, 0), night));
  Assert.Equal("yesterday", RelativeWords(new DateTime(2026, 11, 11, 23, 10, 0), night));
  // Ahead of now, the same wording turned round.
  Assert.Equal("just now", RelativeWords(now.AddSeconds(59), now));
  Assert.Equal("in 5 min", RelativeWords(now.AddMinutes(5), now));
  Assert.Equal("in 2 h", RelativeWords(now.AddHours(2), now));
  Assert.Equal("tomorrow", RelativeWords(new DateTime(2026, 11, 13, 8, 0, 0), now));
  Assert.Equal("20 Nov", RelativeWords(new DateTime(2026, 11, 20, 8, 0, 0), now));
  Assert.Equal("Thu 12 Nov 2026, 09:05 UTC", ExactWhen(new DateTime(2026, 11, 12, 9, 5, 0)));
}
```

From [`tests/Display.test.osy`](tests/Display.test.osy).

## Learn more

- [Osysharp.Look in the package catalogue](https://osysharp.com/packages/#osysharp-look)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
