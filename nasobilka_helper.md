# Násobilka Helper

Multiplication-tables trainer (malá násobilka 1–10) for Zuzka, in the same
style as `math_helper_100.html`. Pure HTML/CSS/JS, no build step.

## Quiz flow
- **Table picker**: pill buttons `1×`–`10×` plus `🎲 Mix`. Picking a table
  immediately generates a new problem; Mix randomises the table each problem.
- **Problem**: `num1 × num2 = ?` where `num1` = chosen table (or random in Mix),
  `num2` = random 1–10.
- **Buttons / hotkeys**: Kontrola `Enter` · Pomoc `H` · Ďalší `N` · Koniec `F`.
- **Score bar + summary overlay** with wrong-answer retry — same pattern as
  `math_helper_100.html` (`wrongProblems`, `retryMistakes`, `finishSession`).

## Visualizations (fill in on reveal)
1. **Násobilková mriežka** – 11×11 grid; highlights row `num1` and column `num2`,
   result cell pops green on reveal.
2. **Opakované sčítanie** – `num2` colored chips of `num1`. On reveal each chip
   shows a small green **skip-counting** number above it (3, 6, 9, 12 …). The
   final result circle uses the **same rainbow colour as the theory table** and
   is **clickable** → smooth-scrolls to the theory table and highlights the cell.
3. **Matica bodiek** – dot rectangle. Shows **both** arrangements to demonstrate
   commutativity: `num1 × num2` next to `num2 × num1` with an `=` between them
   (single block when the factors are equal). Each rectangle is labelled with the
   row count on the left (blue) and the column count on top (pink).

## Theory: Tabuľka násobilky
- Full 10×10 grid.
- **Rainbow colouring** by `max(row, col)` — pink (top-left) → blue
  (bottom-right), forming nested L-shapes. Row header, column header, and the
  L-band of body cells all share one hue. Mirrors the cover-alls.com chart.
- Header row/column are **separated** from the body by a spacer column and
  spacer row (empty gap tracks).
- **Hover** any number → **L-shape highlight**: the row from the left header up
  to the cell and the column from the top header down to the cell stay lit; all
  other cells dim to 20%. Traces a product back to its two factors.

## Tips: Triky a pravidlá
Per-table rules (×1, ×2, ×5, ×9, ×10) as tinted cards with a coloured accent
border; each shows the rule and a worked example on separate lines.

## Helper `rainbowColor(t)`
```js
function rainbowColor(t) {
  const hue = (330 + t * 230) % 360;   // pink → blue
  return `hsl(${hue}, 70%, 74%)`;
}
```
Used by the theory grid (`t = (max(r,c)-1)/9`), the headers, and the repeated-
addition result circle so colours stay consistent across the page.
