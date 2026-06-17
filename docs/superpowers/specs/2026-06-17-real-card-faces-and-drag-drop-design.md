# Real card faces + drag-and-drop (desktop/tablet only)

Date: 2026-06-17
Scope: `index.html` (single-file React 18 + in-browser Babel app)

## Goal

On the **wide** layout only (`useIsWide()` → `min-width:900px` + landscape:
desktop + landscape tablets), replace the minimalist typographic card faces with
**real playing-card faces**, and let the player **drag** the drawn card onto a
column in addition to tapping. The phone (vertical) layout is unchanged.

## 1. Card art — cardmeister

- **Library:** [cardmeister](https://cardmeister.github.io) `<playing-card>` custom
  element. One script, **Unlicense / public domain**. It renders an `<img>` whose
  `src` is an SVG data-URI that scales to the element's CSS box.
- **Delivery:** CDN `<script>` tag alongside React/Babel (matches the existing
  pattern). Can be inlined later for full offline single-file if desired.
- **Mapping** `{r,s}` → `cid` (`[Rank][Suit]`): ranks `2–9` digit, `10→T`,
  `11→J`, `12→Q`, `13→K`, `14→A`; suits `♠→S ♥→H ♦→D ♣→C`. e.g. `{14,'♠'}→"AS"`.
- **Render points** (only when `wide`):
  - `CardV` (column cards, both sides) → `<Face>` instead of `.cr/.cs` text.
  - `DrawnCard` front face → `<Face>`; the **back** stays the existing themed CSS
    back (`.face.back.en/.you`). `CARD_BACK` webp is untouched.
  - A shared `Face` component renders `<playing-card cid=… class="pcface">`,
    sized 100% of the slot; `.cardv`'s `overflow:hidden` + radius clips it.
- **Attack rip VFX:** the `.riphalf top/bot` halves duplicate `.cr/.cs` text today.
  On wide they instead each contain a `<Face>` clipped by the existing
  `clip-path` triangles, so the tear still reads. `.dying` smash, glow, pulse,
  and `pop` act on the container and need no change. Mobile rip untouched.

## 2. Drag-and-drop (pointer events, wide only)

- **Source:** the drawn card in the YOU-rail `.drawhold`. Drag starts on
  `pointerdown` over the `.drawfly` card when `wide && turn==="p" && drawn &&
  !busy`.
- **Mechanic:** a fixed-position `.dragghost` `<Face>` follows the pointer
  (`touch-action:none` so tablets don't scroll). `pointermove` hit-tests the three
  player column halves via `colRefs`; a column with `<3` cards highlights
  (`.half.mine.dragover`). `pointerup` over a valid column → `place("p", col,
  ghostRect)`; the placed card flies in **from the drop point**. Release elsewhere
  → ghost clears, draw stays so the player can retry.
- **Coexist:** tap-to-place (`onColTap`) is unchanged; both paths call `place()`.
- **`place()` change:** add an optional `originRect` param. When supplied (drag),
  the placed card's fly animation originates there; otherwise (tap) it flies from
  the `.drawfly` draw slot as today.
- **Scope:** all drag code/handlers attach only in the `wide` JSX branch; mobile
  remains tap-only.

## 3. Verification

Preview at desktop + tablet (landscape) widths and a phone width:
- All ranks/suits render real faces on wide; mobile is visually unchanged.
- Drag places into each column; snap-back on off-target release; tap still works.
- Attack destruction (rip), hand glow, win flow still play on wide.
- No console errors; cardmeister script loads.

## Non-goals

- No change to game logic, scoring, AI, audio, or the mobile layout.
- Not full-bleed/unique-illustration art — standard real poker faces.
