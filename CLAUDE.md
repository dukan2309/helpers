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
| `nasobilka_helper.html` | Multiplication tables 1–10 |
| `iy_helper.html` | Slovak i/y quiz (hard/soft/both consonants + selected words) |
| `verb_helper.html` | English verb translator + conjugation grid + quiz |

## Shared design system

The newer files (`index.html`, `math_helper_100.html`, `nasobilka_helper.html`, `iy_helper.html`, `verb_helper.html`) share the same CSS custom properties:

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

## Quiz pattern

All helpers share the same quiz structure:

- **Score counters** `quizScore` / `quizTries` (or `score` / `tries`), displayed live. "Pokusy" uses 🎯, not ❌ (which reads as wrong answer).
- **`newQuestion()` / `newQuizQuestion()`** — picks a random item, renders the prompt.
- **`checkQuiz()` / `checkAnswer()`** — validates input, marks ✅/❌, disables inputs.
- **Ďalej** button is disabled until Skontrolovať or Neviem is triggered. Button order: Skontrolovať → Ďalej → Neviem → Koniec.
- **Summary overlay** (`finishQuiz()` / `finishSession()`) shown when word-count limit is reached or user presses Koniec.
- **Keyboard shortcuts** consistent across all helpers: `Enter` = check / next, `End` = finish, `Esc` = cancel quiz. Letter shortcuts (N/H/F) are not used — the user types answers, so letter keys must remain free.

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
