---
name: bidi-direction
description: >-
  Apply this whenever the response contains Arabic text, or mixes Arabic with
  English/Latin words, numbers, URLs, code, or punctuation. Ensures correct
  bidirectional (RTL/LTR) ordering so embedded Latin runs, digits, and
  punctuation render in the right visual position instead of jumping around.
  Triggers: any Arabic in the reply, Arabic-English mixed sentences, Arabic with
  version numbers / prices / dates, Arabic sentences ending in "?" or ":".
---

# Arabic / English bidirectional text

## The problem

When Right-to-Left (Arabic) and Left-to-Right (Latin, digits) text share a line,
the renderer applies the Unicode Bidirectional Algorithm (UBA). Left alone, the
UBA often places embedded runs in the wrong visual spot:

- A Latin word or number inside an Arabic sentence drifts to the wrong side.
- Trailing punctuation (`?` `!` `:` `.`) attaches to the wrong end of the line.
- Parentheses / brackets around a mixed run mirror or swap.
- A `-` between two numbers, or a `/` in a path, reorders the operands.

The character order is already logical and correct; the renderer decides the
*visual* order. So the model cannot fix this by "ordering characters better."
The only robust fix is to emit **invisible Unicode bidi-control characters** that
mark where each run begins and ends. They survive Markdown and copy-paste and
add no visible glyph.

## The control characters

Prefer **isolates** (modern, self-contained) over the older embeddings. Use
**marks** for a single stray boundary. These are referred to by name/code point
below — do not try to recognise them by eye, they are invisible:

- **LRI** — U+2066 — Left-to-Right Isolate — open an LTR run (Latin, code, URL) inside Arabic
- **PDI** — U+2069 — Pop Directional Isolate — close the most recent isolate
- **RLI** — U+2067 — Right-to-Left Isolate — open an Arabic run inside LTR text
- **FSI** — U+2068 — First-Strong Isolate — open a run whose direction is auto-detected from its first strong character
- **LRM** — U+200E — Left-to-Right Mark — invisible LTR anchor for a single boundary
- **RLM** — U+200F — Right-to-Left Mark — invisible RTL anchor; fixes trailing punctuation on an Arabic line

Every isolate you open (LRI / RLI / FSI) must be closed with exactly one PDI.

## Rules

1. **Wrap every Latin / number / code run embedded in Arabic** with LRI … PDI.
   Keep the wrap tight — just the foreign token, not the Arabic spaces around it.
   Applies to: version numbers, English product and tech names, file paths, URLs,
   code identifiers, emails, inline `code spans`, prices, dates in Latin digits.

   Example (version number inside Arabic):
   `النسخة ⁦v2.1.0⁩ جاهزة`

2. **Numeric ranges / ratios** (`3-5`, `16/9`, `1,200`): wrap the whole token in
   LRI … PDI so digits and separator keep their order.

   Example:
   `المدة ⁦3-5⁩ أيام`

3. **Arabic run inside an English line**: wrap with RLI … PDI (or FSI … PDI when
   unsure of the run direction).

   Example:
   `The label reads ⁧تشغيل⁩ on the button.`

4. **Trailing punctuation on an Arabic sentence** (`?` `!` `:`): if it jumps to
   the wrong side, append RLM (U+200F) immediately after it.

   Example:
   `هل انتهيت؟‏`

5. **Never** wrap a whole Arabic paragraph in isolates — that fights the
   renderer. Isolate only the embedded foreign run.

6. **Pure-English replies**: do nothing. These controls are for mixed or Arabic
   content only.

## Block direction (alignment) — the honest limit

These characters fix *ordering within a line*. They do **not** right-align a
whole Arabic block in a plain-Markdown chat renderer. Block alignment is a
UI/CSS concern (`dir="rtl"`, `text-align`) that an inline text response cannot
set. For full RTL layout in your own app, that belongs in the frontend
(`dir` attribute / CSS), not in the emitted text.

## Self-check before sending an Arabic reply

- Every English word / number / path inside Arabic wrapped in LRI … PDI?
- Each opened isolate closed with one PDI?
- Arabic sentence ending in `? ! :` followed by RLM if it drifts?
- No whole paragraph needlessly isolated?
