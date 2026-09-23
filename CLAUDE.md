# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

No build step. Open any `.html` file directly in a browser. All CSS and JS are inline in each file.

```
# open home page
start index.html

# or serve locally to avoid CORS on font fetches
npx serve .
```

## Architecture

Each helper is a **single self-contained HTML file** — no framework, no bundler, no shared JS modules. CSS and JS live inline inside `<style>` and `<script>` tags.

| File | Purpose |
|---|---|
| `index.html` | Home page — tile grid linking to all helpers |
| `math_helper.html` | Addition/subtraction up to 20 (older style) |
| `math_helper_100.html` | Addition/subtraction up to 100 |
| `nasobilka_helper.html` | Multiplication tables 1–10 (multi-select + Mix) |
| `delenie_helper.html` | Division tied to the times tables 1–10 (inverse of `nasobilka_helper.html`, exact/no remainders) |
| `iy_helper.html` | Slovak i/y quiz (hard/soft/both consonants + selected words) |
| `verb_helper.html` | English verb translator + conjugation grid + quiz |

## Shared design system

The newer files (`index.html`, `math_helper_100.html`, `nasobilka_helper.html`, `delenie_helper.html`, `iy_helper.html`, `verb_helper.html`) share the same CSS custom properties:

```css
--primary: #7c6cf2;   /* lavender */
--accent:  #ff7aa8;   /* pink */
--green:   #4cc38a;
--orange:  #ffb86b;
--ink:     #2b2b3f;
--muted:   #6c6e89;
--line:    #e7e9f4;
```

Font: **Quicksand** (Google Fonts). Background: three-ellipse radial gradient on `#fafbff`. `math_helper.html` predates this system and uses Comic Sans + an orange gradient.

### h1 gradient + emoji pattern

Applying a CSS gradient to `h1` text via `background-clip:text; color:transparent` also clips child emoji, turning them solid purple. Always split the heading into two spans:

```html
<h1><span class="h1-icon">🔢</span> <span class="h1-grad">Title text</span></h1>
```

```css
h1 { color: var(--ink); }          /* emoji inherit this — visible */
h1 .h1-grad {
  background: linear-gradient(135deg, var(--primary), var(--accent));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
/* h1-icon gets no gradient — just inherits color: var(--ink) */
```

## Home button

All helpers have a fixed lavender pill button `← Domov` linking to `index.html`:

```html
<a class="home-btn" href="index.html">← Domov</a>
```

It sits directly after `<body>`, outside any wrapper div, and uses `position: fixed; top: 14px; left: 16px`. Every file also has a `@media (max-width: 600px)` block that shrinks the button and adds `padding-top: 60px` to `body` so the button doesn't overlap the page title on phones.

## SEO / social meta

The site is served from the custom domain **pomocnici.com** (see `CNAME`). Every page's `<head>` carries, right after `<title>`:

- `<meta name="description">` — unique Slovak description per page (the Google snippet).
- `<link rel="canonical">` — absolute `https://pomocnici.com/<file>.html` (home page uses `https://pomocnici.com/`).
- Favicon: `<link rel="icon" type="image/png" href="favicon.png">` (96×96) + `<link rel="apple-touch-icon" href="apple-touch-icon.png">` (180×180). Both are square center-crops of `pomocnici_emblem.png` (which is 1408×768 — using it directly makes the browser squash the favicon into a distorted oval, so a square crop is required). Built with the PowerShell System.Drawing technique (see Images section).
- **PWA / Android home-screen icon**: `<link rel="manifest" href="manifest.webmanifest">` (relative, on **every** page). `manifest.webmanifest` (repo root) declares `name`/`short_name` "Pomocníci", `display: standalone`, `background_color: #171d3a` (deep indigo), `theme_color: #7c6cf2`, and three icons: `icon-192.png` + `icon-512.png` (`purpose: "any"`) and `icon-maskable-512.png` (`purpose: "maskable"`). **Why maskable + opaque background:** Android masks home-screen icons to a circle/squircle and fills any transparency white, then shrinks non-maskable icons into a small safe circle — that's what made the icon "a small picture in a white circle." The maskable icon keeps its content inside the inner 80% safe zone on a full-bleed opaque indigo background, so Android's mask crops the background, not the medallion. All four icon PNGs are center-crops of `pomocnici_emblem.png` on a `#171d3a` background (crop x=321, y=0, size=768): `icon-192.png`/`icon-512.png` at 90%, `icon-maskable-512.png` at 80% (safe zone), `apple-touch-icon.png` at 92%. Built with the PowerShell System.Drawing technique (`g.Clear($bg)` then `DrawImage` the crop into a centered square; see Images section).
- Open Graph + Twitter card tags. **OG image is always the absolute URL `https://pomocnici.com/og_image.jpg`** — a 1200×630 (1.905:1) social-sized center-crop of the classroom hero `pomocnici_ucebna.jpg`, sized to fit Facebook/Twitter cards without letterboxing. `og:title`/`og:description` mirror the page title/description; `og:locale` is `sk_SK`.
- `index.html` also has a `WebSite` **JSON-LD** block (`application/ld+json`) — validate it parses as JSON after editing.

Rules:
- Canonical, `og:url`, `og:image`, and sitemap entries use **absolute** `pomocnici.com` URLs; everything else (internal links, favicon, images) stays **relative** so the pages also work at `dukan2309.github.io/helpers/` and when opened locally.
- Page language: all content is Slovak → `<html lang="sk">` on every file (the two math helpers were `lang="en"` by mistake — fixed).
- `robots.txt` and `sitemap.xml` live at the repo root. **When adding a new page, add a matching `<url>` entry to `sitemap.xml`.**
- Google Search Console: domain verified via a DNS TXT record at websupport.sk; sitemap submitted at `https://pomocnici.com/sitemap.xml`.

## Quiz pattern

All helpers share the same quiz structure:

- **Score counters** `quizScore` / `quizTries` (or `score` / `tries`), displayed live. "Pokusy" uses 🎯, not ❌ (which reads as wrong answer).
- **`newQuestion()` / `newQuizQuestion()`** — picks a random item, renders the prompt.
- **`checkQuiz()` / `checkAnswer()`** — validates input, marks ✅/❌, disables inputs.
- **Ďalej** button is disabled until Skontrolovať or Neviem is triggered. Button order: Skontrolovať → Ďalej → Neviem → Koniec.
- **Summary overlay** (`finishQuiz()` / `finishSession()`) shown when word-count limit is reached or user presses Koniec.
- **Keyboard shortcuts** consistent across all helpers: `Enter` = check / next, `End` = finish, `Esc` = cancel quiz. Letter shortcuts (N/H/F) are not used — the user types answers, so letter keys must remain free.

## math_helper_100.html specifics

Two **display modes**, chosen by a **toggle switch** (`.method-toggle` at the top of the quiz card) with two clickable labels: **riadkové počítanie** (left) ↔ **písomné počítanie** (right). The switch is a checkbox `#col-mode` (`onColModeChanged()`); clicking either label calls `setColMode(bool)` which flips the checkbox. `applyDisplayMode()` toggles the `.active` class on `#mt-classic`/`#mt-column`.

- **Horizontal mode** (default): the main quiz card with the horizontal `num1 op num2 = [#answer-input]` line (`#h-problem`), plus all five visualization cards (`sbs-card`, `bridge-card`, `pv-card`, `grid-card`, `nl-card`).
- **Column mode** (written method, Slovak "písomný postup / stĺpcová metóda"): the five viz cards are hidden and `#h-problem` is swapped for the stacked column layout rendered into `#col-method-area` (inside the **same** quiz card — there is no separate card). The student types the answer directly into digit input boxes. Column mode is constrained to **two-digit values (≤ 99)** — `generate()`'s while-loop rejects any problem where `num1`, `num2`, or `correctAnswer` exceeds 99, so the layout only ever needs tens + ones. `onColModeChanged()` regenerates if the current problem is out of range when switching into column mode.

`applyDisplayMode()` owns all show/hide: the four viz cards by id, `bridge-card` force-hidden (re-shown by `renderBridge()` in horizontal mode only), and `#h-problem` ↔ `#col-method-area`. It runs once at init (before `nextProblem()`), on every toggle, **and at the top of `render()`** (so the post-check classic-viz cards re-hide on each new problem). Toggling mid-problem preserves reveal state via `renderAll(answered || hintUsed)`.

All visualization rendering fans out through **`renderAll(reveal)`**, which branches on `columnMode` — `render()` calls `renderAll(false)`; `checkAnswer()` calls `renderAll(true)`. Do **not** call individual `renderX()` functions from those sites.

**`renderColumnMethod(reveal)`** — stacked place-value layout on a 3-column grid (`op | T | O`, classes `.cm-op/.cm-t/.cm-o`; the redundant hundreds column was removed). num1 row, operator+num2 row, `.cm-line` rule, then the result row:
- **before reveal:** two editable answer boxes (`inputBox()` → `.cm-input`, tens + ones) — always exactly 2 regardless of operation, since column mode never exceeds 99. The ones box gets focus; typing a digit auto-advances **right→left** (ones→tens, the column-addition order), and Backspace on an empty box moves back left→right (`colSibling(inp, ±1)`).
- **after reveal:** the correct answer digits, colored, with `.cm-ans` (pop animation).

**Carry / borrow school notation (reveal only).** On reveal `renderColumnMethod` annotates the **num1 row** with the marks a child writes on paper, via a small `mark(cls, txt)` → `.cm-regroup` span (orange `#f07d3a`) positioned absolutely inside the tens/ones cells (which are `position: relative`):
- **Carry** (addition, `o1 + o2 ≥ 10`): a little `1` above the tens (`.cm-carry`).
- **Borrow** (subtraction, `o1 < o2`): the tens digit gets a diagonal strike (`.cm-struck` with a rotated `::after` bar), the reduced tens value is written above it (`.cm-newtens`), and a little `1` sits on the ones (`.cm-borrowone`) so they read as `1o`.
A one-line caption (`.cm-note`, below the grid) ties the marks to words — e.g. `2 − 8 sa nedá → požičiame si 1 desiatku…` / `7 + 5 = 12 → 1 desiatku prenesieme vyššie.`. `.col-method-area` is `flex-direction: column` so the note stacks under the grid; `.col-method` has `padding-top` for the marks above the top row.

**`readColumnValue()`** assembles the entered number from the `.cm-input` boxes by place class (blank = 0); returns `NaN` if all blank. `checkAnswer()` reads it in column mode (vs `#answer-input` in horizontal). The global keydown handler skips letter shortcuts when focus is a `.cm-input` (Enter still checks).

**Post-check classic-viz reveal:** in column mode, after a correct/wrong check `checkAnswer()` calls **`showClassicVizReveal()`**, which renders **only the place-value card** (`renderPlaceValue(true)` + un-hides `pv-card`) solved for the same problem — place value is the single visualization that maps onto the written method (and its own text already narrates the carry/borrow: *"prenesieme 10!"* / *"požičali sme 10 z desiatok"*). It re-hides on the next `render()` (via `applyDisplayMode()` at its top).

**Pomoc in column mode** fills **only the ones box** with the correct ones digit (still editable) and focuses the tens box so the student works out the tens themselves; sets `hintUsed`; the student confirms with Kontrola (scores "with help"). In horizontal mode Pomoc still reveals the side visualizations.

## verb_helper.html specifics

This is the most complex file. Key data structures and functions:

- **`VERBS[]`** — master dictionary (~150 entries). Each entry: `{ en, sk[], forms, note?, impersonal? }`. `sk` is an array of Slovak infinitives.
- **`SK_PRES_IRREG`** / **`SK_PAST_IRREG`** — lookup tables for Slovak conjugation by infinitive. Every verb in the dictionary has an explicit entry; do not rely on a conjugation engine for quiz answers.
- **`skConjugate(entry, tense, pronKey)`** — returns the conjugated Slovak form. For the quiz the prompt uses `entry.sk[0]` (the infinitive) directly to avoid conjugation bugs.
- **`expectedAnswer(entry, tense, pronKey)`** — returns the expected English answer for a given tense + pronoun.
- **`pickPron()`** — returns `[label, pronKey, skKey, hint]` (4 elements). `pronKey` is one of `i / you / heshe / we / they`.
- **`getSelectedTenses()`** — reads `.quiz-tc:checked` checkboxes → array of `'pres' | 'presc' | 'past' | 'fut'`.
- **`refreshTenseRows()`** — re-renders tense input rows for the *current* verb/pronoun when checkboxes change (does **not** pick a new verb).
- **`quizWordLimit`** (0 = unlimited) / **`quizWordsAnswered`** — enforce the count selector.
- **Scoring** — one point per tense form answered correctly (`quizScore / quizTries`). Partial credit: if 1 of 4 tenses correct, score += 1, tries += 4. Closing the summary resets the score.
- **Two-stage Pomoc** — first press shows the English translation as a hint; second press reveals all correct forms and marks the question wrong. "Ďalej" button stays disabled until Skontrolovať or second Pomoc press; it shows a tooltip when disabled.
- **`quizStart` button** — always labeled "Reštartovať kvíz" after first start; always does a full reset (no "Pokračovať" state).

### Quiz card layout

The quiz card has two rows followed by tense input rows:

1. **SK row** (`.quiz-prompt`): `SK_FLAG_SVG` (inline SVG, `min-width: 130px`) + Slovak infinitive (`.sk-word`).
2. **EN row** (`.pron-row`): `GB_FLAG_SVG` (inline SVG, `min-width: 130px`) + person label (`.tense-pron`) + ghost-button hint (`.pron-info-btn`) showing e.g. `3rd person – singular`.
3. **Tense rows** (`.tense-row`): tense label (`.tense-lbl`, desktop `min-width: 130px`, mobile `flex: none; width: 120px`) + person label (`.tense-pron`) + input + feedback span.

The fixed widths on flag spans, tense labels, and pron-lbl-col keep the verb, person, and input boxes aligned in the same column across all rows. Mobile widths were measured via headless Chrome to fit the longest label.

**Flag icons** use inline SVG constants `SK_FLAG_SVG` / `GB_FLAG_SVG` (defined just above `newQuizQuestion()`). Do not use emoji flags — they render as "SK"/"GB" text on Windows. Each flag span also includes a `<span class="flag-title">SK</span>` / `<span class="flag-title">EN</span>` text label.

## JS syntax check

After any edit to a helper's `<script>` block, validate with:

```bash
node -e '
const fs=require("fs");
const html=fs.readFileSync("FILE.html","utf8");
const m=html.match(/<script>([\s\S]*)<\/script>/);
new Function(m[1]);
console.log("Syntax OK");
'
```

## Multi-select table/divisor picker pattern

`nasobilka_helper.html` and `delenie_helper.html` share the same picker pattern:

```js
let selectedTables = new Set([1]);  // or selectedDivisors
let mixMode = false;

function updatePickerUI() {
  document.querySelectorAll('.tbl-btn[data-table]').forEach(b => {
    b.disabled = mixMode;
    b.classList.toggle('active', !mixMode && selectedTables.has(+b.dataset.table));
  });
  document.getElementById('mix-btn').classList.toggle('active', mixMode);
}
function toggleTable(i) {
  if (mixMode) return;
  if (selectedTables.has(i)) { if (selectedTables.size === 1) return; selectedTables.delete(i); }
  else { selectedTables.add(i); }
  updatePickerUI(); generate(); render();
}
function toggleMix() { mixMode = !mixMode; updatePickerUI(); generate(); render(); }
```

- **Mix** disables all individual buttons via native `disabled` + global `button:disabled` CSS.
- At least one table/divisor always stays selected (last one cannot be deselected).

## delenie_helper.html specifics

Key variables: `dividend`, `divisor`, `quotient`. `correctAnswer = quotient`. `generate()` sets `divisor` from `selectedDivisors` (or random 1–10 in Mix), then `quotient = rand(1,10)`, `dividend = divisor * quotient` — always exact, no remainder.

### Two deliberate division models — do not unify

The two main visualizations intentionally show *different* meanings of ÷:

| Section | Model | Example 20 ÷ 4 |
|---|---|---|
| 📏 Číselná os | **Quotitive** — "how many jumps of 4 fit?" → count the arcs | 5 arcs of −4 → answer = 5 |
| ⬛ Rozdeľovanie do skupín | **Partitive** — "share into 4 groups, how many each?" → answer is per-group size | 4 rows × 5 dots → answer = 5 |

The grid draws `divisor` rows of `quotient` dots. The caption reads `"N skupín po M"` where N = divisor, M = quotient.

### Number line CSS classes

| Class | Purpose |
|---|---|
| `.nl-line` | The horizontal axis bar |
| `.nl-tick` | Faint scale tick (no label) |
| `.nl-pt` | Coloured dot at each landing point (start blue, end green, intermediate lavender) |
| `.nl-ptlabel` | Running value shown below each landing dot |
| `.nl-arc` | Dashed semicircle for each backward jump |
| `.nl-jump-num` | Circular lavender badge with jump counter (1, 2, 3…) above each arc |
| `.nl-sub` | "−divisor" pill tag inside each arc |

Before reveal: only start and 0 dots shown, with a hint prompt. On reveal: all landing points + numbered arcs.

### `skPlural` helper

```js
function skPlural(n, one, few, many) {
  return n === 1 ? one : (n >= 2 && n <= 4) ? few : many;
}
// e.g. skPlural(quotient, 'skok', 'skoky', 'skokov')
//      skPlural(divisor,  'skupina', 'skupiny', 'skupín')
```

### Fact family (`#fact-family`)

Shows four linked facts: `divisor × quotient = dividend`, `quotient × divisor = dividend`, `dividend ÷ divisor = quotient`, `dividend ÷ quotient = divisor`. Quotient cells show `?` until reveal. CSS classes: `.ff-dividend` (blue), `.ff-divisor` (pink), `.ff-quotient` (green).

## iy_helper.html specifics

- Word list comes from `zoznam_slov.txt` (fetched at runtime) plus a hardcoded fallback array.
- Quiz feedback messages must **not** contain `i/í` or `y/ý` letter pairs. The "nie X" suffix is appended dynamically from the clicked letter, not hardcoded.
- Consonant categories: `TVRDE` (orange `#f07d3a`), `MAKKE` (blue `#4a90d9`), `OBOIAKE` (lavender `#7c6cf2`).

### QW word array

Each entry in the `QW` array:

```js
{ b:'b', a:'cykel', ans:'i',
  err_fam:'bicykel nie je vybrané slovo → po "b" píšeme i',
  err_dlz:'bicykel nemá dĺžeň → píšeme i (nie í)',
  ctx:'dobrý b_cykel',   // optional — sentence shown in place of "b_cykel"
  hl:'"b"'               // optional — substring to highlight in the rule
}
```

- `b` + gap + `a` assembles the word display (e.g. `b` + `_` + `cykel`).
- `ans` is `'i' | 'í' | 'y' | 'ý'`, or an array of accepted answers.

**Difficulty** is assigned by array index via two Sets at the bottom of the script:

```js
const _D1 = new Set([0,1,…]);  // easy   🐣
const _D3 = new Set([81,86,…]); // hard   🐲
// everything else → medium 🦊
QW.forEach((w,i) => { w.d = _D3.has(i) ? 3 : _D1.has(i) ? 1 : 2; });
```

**Critical:** indices are positional. **Appending** to the end of `QW` is safe — just add the new indices to `_D1` or `_D3` as needed. **Removing or inserting** anywhere else requires decrementing/incrementing every higher index in both Sets (use a script; manual edits miss entries).

## Images

`pomocnici_ucebna.jpg` is the hero image on `index.html`. It has a colorful edge-to-edge illustration so it is used as a plain JPEG (no `mix-blend-mode`). It is clickable — a CSS lightbox overlay opens the image full-size (click or Esc to close). Images with a white/transparent background use `mix-blend-mode: multiply` to blend into the page gradient.

When converting a JPEG with a plain background to a transparent PNG, use the flood-fill C# script embedded in PowerShell (`Add-Type -TypeDefinition`) with `LockBits` for performance. Seed the flood-fill from all four edges; tune brightness/saturation thresholds to match the actual background color (sample corners first).

Other image assets at repo root: `pomocnici_stoja.jpg`, `pomocnici_stoja.png`, `pomocnici_stol.jpg`, `pomocnici_emblem.jpg`, `pomocnici_emblem.png`, `pomocnici_ucebna.jpg`.
