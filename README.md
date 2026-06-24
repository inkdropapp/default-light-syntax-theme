# Default light syntax theme for Inkdrop Markdown Editor

The built-in default **light** syntax theme for [Inkdrop](https://www.inkdrop.app/).
Its colours are derived from the **Tailwind Ice** palette (its dark counterpart,
`default-dark-syntax`, uses **Tailwind Moon**). It also doubles as a **reference
implementation** for anyone who wants to build their own syntax theme for Inkdrop (which
runs on [CodeMirror 6](https://codemirror.net/)), or migrate a theme written for older
versions.

If you just want to read the code, the whole theme is a single stylesheet —
[`styles/index.css`](./styles/index.css) — and it contains **only CSS variables**.

---

## How a syntax theme is packaged

A syntax theme is an ordinary Inkdrop package whose `package.json` declares a `theme`
of type `"syntax"` and points at one or more stylesheets:

```json
{
  "name": "my-syntax",
  "version": "1.0.0",
  "description": "My syntax theme",
  "theme": "syntax",
  "styleSheets": ["index.css"],
  "engines": { "inkdrop": "^6.x" }
}
```

- `theme: "syntax"` — tells Inkdrop this package paints the editor/preview, as opposed
  to a UI theme.
- `styleSheets` — paths (relative to `styles/`) loaded into the app, in order.
- `engines.inkdrop: "^6.x"` — target Inkdrop 6. (Older themes targeted `^5.x`; see
  [Migrating](#migrating-an-older-theme) below.)

Two slots exist in Inkdrop: a **UI theme** and a **syntax theme**, chosen independently
in _Preferences → Themes_. This package fills the syntax slot.

---

## Quick start

Read the [theme development guide](https://developers.inkdrop.app/guides/create-a-theme).

## Anatomy of `styles/index.css`

A syntax theme is **just a set of CSS variables**. The entire file is one `:root { … }`
block — it has no selectors. The rules that actually paint the highlighting (`.tok-*`,
`.md-*`) ship in the shared **`@inkdropapp/css`** package and consume your variables, so
re-skinning the theme means changing variable _values_, not writing CSS rules.

```
primitive tokens   (@inkdropapp/css)                   →  --color-blue-600, --hsl-blue-500, …
        │
semantic variables (--editor-*, --syntax-*, --md-*)    →  this theme (:root only)
        │
highlighting rules (.tok-*, .md-*, in @inkdropapp/css) →  map token classes onto your vars
```

### Primitive color tokens

The lowest layer is **not** defined here. The primitive palette — a Tailwind-style ramp
in the shared **`@inkdropapp/css`** package, loaded by the app before your theme — comes
in two forms:

- **`--color-<family>-<scale>`** — finished, opaque colours (e.g. `--color-blue-600`).
- **`--hsl-<family>-<scale>`** — the underlying **HSL triplets** (e.g. `217deg 91% 60%`),
  for when you need an alpha channel.

> **Primitive color tokens are defined in
> <https://github.com/inkdropapp/css/blob/main/tokens.css>**

Families available: `slate`, `gray`, `zinc`, `neutral`, `stone`, `red`, `orange`,
`amber`, `yellow`, `lime`, `green`, `emerald`, `teal`, `cyan`, `sky`, `blue`, `indigo`,
`violet`, `purple`, `fuchsia`, `pink`, `rose` — each in scales `50, 100, … 900, 950`,
plus `--color-white` / `--color-black`.

```css
color: var(--color-blue-600); /* opaque finished colour     */
background: hsl(var(--hsl-blue-400) / 40%); /* HSL triplet with 40% alpha */
```

### Semantic variables

Everything in this theme is one of three variable families, all in `:root`:

- **`--editor-*`** — editor chrome: foreground/background, caret, selection, gutter,
  active line, tooltips & autocomplete, matching brackets, search matches, whitespace
  rendering, fold placeholders, etc.

  ```css
  --editor-foreground-color: var(--color-gray-800);
  --editor-background-color: var(--color-white);
  --editor-caret-color: var(--color-violet-600);
  --editor-selection-background: hsl(var(--hsl-violet-600) / 13%);
  ```

- **`--syntax-*`** — one variable per highlight token; this is the token-colour contract
  (see [below](#syntax-tokens---syntax-)).

  ```css
  --syntax-keyword-color: var(--color-violet-600);
  --syntax-string-color: var(--color-green-600);
  --syntax-comment-color: var(--color-slate-400);
  --syntax-comment-font-style: italic;
  ```

- **`--md-*`** — Markdown decorations: code blocks, inline code, tables, blockquotes,
  inline marks, list markers.

  ```css
  --md-codeblock-background-color: hsl(var(--hsl-slate-50));
  --md-table-border-color: hsl(var(--hsl-slate-200));
  --md-blockquote-border-color: var(--color-slate-500);
  ```

These variable names are the theming contract Inkdrop consumes — **keep the names, change
the values.**

### Syntax tokens (`--syntax-*`)

Inkdrop's highlighter emits `.tok-*` classes (one per
[Lezer](https://lezer.codemirror.net/) highlight tag) inside `.cm-editor` and inside
rendered preview code blocks (`.mde-preview .codeblock`). The rules that paint them ship
in **`@inkdropapp/css`** and simply read your variables — you don't write them:

```css
/* lives in @inkdropapp/css, not in your theme */
.cm-editor,
.mde-preview .codeblock {
  .tok-keyword {
    color: var(--syntax-keyword-color);
  }
  .tok-string {
    color: var(--syntax-string-color);
  }
  .tok-comment {
    color: var(--syntax-comment-color);
    font-style: var(--syntax-comment-font-style, italic);
  }
}
```

So your job is only to set the `--syntax-*` values. A few conventions:

- There is **one `--syntax-<token>-color` per token**, plus `-font-style` /
  `-font-weight` / `-text-decoration` where relevant (`--syntax-comment-font-style`,
  `--syntax-heading-font-weight`, `--syntax-link-text-decoration`, …) and
  `--syntax-invalid-border-bottom` for the error squiggle.
- **Modifier variants default to their base token**, so you only override what differs —
  e.g. `--syntax-name-function-color: var(--syntax-name-color)`.
- **Tokens you don't colour fall back to the editor foreground** — e.g.
  `--syntax-punctuation-color: var(--editor-foreground-color)`.

The full set of tokens is the
[`tokenHighlighter`](https://github.com/inkdropapp/cm6-themes) catalog; this theme defines
a value for every one.

### Markdown source markers (`.md-*`)

Alongside the token rules, `@inkdropapp/css` also styles the Markdown **source markers** —
`.md-header-mark`, `.md-list-mark`, `.md-emphasis-mark`, `.md-code-mark`,
`.md-task-marker`, the inline-code box, etc. — from the `--md-*` variables. Set those
variables to restyle the literal `#`, `-`, `*` and `` ` `` characters and the code-block
chrome.

> This theme uses no preprocessor and contains no selectors of its own. Inkdrop 6 ships
> modern CSS, so `hsl(... / alpha)` and custom properties work without a build step.

---

## Migrating an older theme

Porting a theme from Inkdrop 5 (CodeMirror 5, usually authored in LESS) to Inkdrop 6 /
CodeMirror 6? See **[Plugin Migration Guide from v5 to v6 → Syntax themes](https://developer.inkdrop.app/appendix/plugin-migration-from-v5-to-v6#syntax-themes)**.

It covers the `.CodeMirror-*` → `--editor-*` variable mapping and the `.cm-*` → `.tok-*`
token-class mapping — which you now drive by setting `--syntax-*` variables rather than
writing `.tok-*` rules. The [anatomy](#anatomy-of-stylesindexcss) above is what you're
migrating _to_.

---

## Reference

- Primitive color tokens — <https://github.com/inkdropapp/css/blob/main/tokens.css>
- This theme's source — [`styles/index.css`](./styles/index.css)
- The shared highlighting rules (`.tok-*` / `.md-*`) and `--syntax-*` contract —
  [`@inkdropapp/css`](https://github.com/inkdropapp/css)
- CodeMirror 6 themes (the underlying npm packages) —
  [inkdropapp/cm6-themes](https://github.com/inkdropapp/cm6-themes)
- Inkdrop documentation — <https://docs.inkdrop.app/>

## License

[MIT](./LICENSE)
