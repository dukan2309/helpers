# Verb Helper (Slovesník)

English verb learning app for Zuzka — translation, conjugation display, and quiz.

## Features

### Translator
- Slovak ↔ English lookup (local dictionary ~150 verbs + online fallback via MyMemory)
- Swap button (⇄) exchanges both fields and re-triggers lookup
- Shows irregular/regular badge

### Conjugation grid
- 4 tenses: Present simple, Present continuous, Past simple, Future simple
- Singular / plural columns with ending highlights
- Toggle to show Slovak equivalents alongside English forms
- Irregular forms card (V1 / V2 / V3) shown for every verb

### Quiz (Skúška)
- Shows Slovak infinitive + English pronoun (e.g. 🇸🇰 hrať → 🇬🇧 She)
- Pronoun hint shown (e.g. "3rd person singular (f.)")
- Tense selector: choose any combination of Prítomný / Prítomný priebehový / Minulý / Budúci
- One input row per selected tense; Enter moves between rows, then submits
- Question count selector: 5 / 10 / 20 / ∞ (default 10); ends quiz with summary when reached
- Changing count mid-quiz resets score and starts fresh
- Buttons: Skontrolovať `Enter` · Neviem `H` · Ďalej `N` · Koniec `F`
- Ďalej: checks first if not yet answered, then advances
- Summary overlay with score, grade, and play-again option

## Slovak conjugation
- `SK_PRES_IRREG` table covers all verbs from the dictionary
- `SK_PAST_IRREG` table for irregular past stems
- Rhythmic-law verbs (ľúbiť, cítiť, …) have explicit entries (short endings)
- Impersonal verbs (pršať, snežiť) use "It / ono" only
- `mať rád` agreement: rád/rada/rado/radi by gender/number

## English conjugation
- `isCVC()` + `NO_DOUBLE` set for correct -ing/-ed doubling
- UK spelling (travelling, travelled)
