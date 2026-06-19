# Default light syntax theme for Inkdrop Markdown Editor

The built-in default **light** syntax theme for [Inkdrop](https://www.inkdrop.app/).
It also doubles as a **reference implementation** for anyone who wants to build their
own syntax theme for Inkdrop (which runs on [CodeMirror 6](https://codemirror.net/)),
or migrate a theme written for older versions.

If you just want to read the code, the whole theme is a single stylesheet:
[`styles/index.css`](./styles/index.css).

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

The stylesheet is organised as a stack of layers, each referring only to the one above
it. Re-skinning the theme is mostly a matter of editing the top layers and leaving the
bottom ones alone.

```
primitive tokens  (@inkdropapp/css)   →   --hsl-slate-800, --hsl-blue-500, …
        │
16-slot palette   (--hsl-baseNN)      →   one primitive per slot
        │
resolved colours  (--bd-baseNN)       →   hsl(var(--hsl-baseNN))
        │
semantic vars     (--editor-*, --md-*)→   editor chrome & markdown decorations
        │
syntax classes    (.tok-*, .md-*)     →   the actual highlighting rules
```

### Primitive color tokens

The lowest layer is **not** defined here. The primitive palette — a Tailwind-style ramp
of `--hsl-<family>-<scale>` triplets (`--hsl-slate-50` … `--hsl-slate-950`, plus
`--hsl-white` / `--hsl-black`) — lives in the shared **`@inkdropapp/css`** package and is
loaded by the app before your theme:

> **Primitive color tokens are defined in
> <https://github.com/inkdropapp/css/blob/main/tokens.css>**

Families available: `slate`, `gray`, `zinc`, `neutral`, `stone`, `red`, `orange`,
`amber`, `yellow`, `lime`, `green`, `emerald`, `teal`, `cyan`, `sky`, `blue`, `indigo`,
`violet`, `purple`, `fuchsia`, `pink`, `rose` — each in scales `50, 100, … 900, 950`.

The values are **HSL triplets** (e.g. `220deg 16% 22%`), not finished colours, so you
wrap them in `hsl()` and can add an alpha channel:

```css
color: hsl(var(--hsl-blue-500)); /* opaque   */
background: hsl(var(--hsl-blue-400) / 40%); /* 40% alpha */
```

### Semantic variables

The palette then feeds two families of semantic variables, also in `:root`:

- **`--editor-*`** — editor chrome: foreground/background, caret, selection, gutter,
  active line, tooltips & autocomplete, matching brackets, search matches, whitespace
  rendering, fold placeholders, etc. For example:

  ```css
  --editor-foreground-color: var(--bd-base01);
  --editor-background-color: var(--bd-base00);
  --editor-caret-color: var(--bd-cursor);
  --editor-selection-background: hsl(var(--hsl-slate-500) / 40%);
  ```

- **`--md-*`** — Markdown decorations: code blocks, tables, blockquotes, inline marks:

  ```css
  --md-codeblock-background-color: hsl(var(--hsl-slate-50));
  --md-table-border-color: hsl(var(--hsl-slate-200));
  --md-blockquote-border-color: var(--bd-base06);
  ```

These variable names are the theming contract Inkdrop consumes — keep the names, change
the values.

### Syntax classes

Finally, the actual highlighting. Inkdrop's highlighter emits `.tok-*` classes (one per
[Lezer](https://lezer.codemirror.net/) highlight tag) inside `.cm-editor` and inside
rendered preview code blocks (`.mde-preview .codeblock`):

```css
.cm-editor,
.mde-preview .codeblock {
  .tok-keyword {
    color: var(--bd-base0D);
  }
  .tok-string {
    color: var(--bd-base0C);
  }
  .tok-comment {
    color: var(--bd-base06);
    font-style: italic;
  }
  /* …one rule per token kind… */
}
```

Alongside the `.tok-*` rules are **`.md-*` mark classes** for the Markdown source markers
themselves — `.md-header-mark`, `.md-list-mark`, `.md-emphasis-mark`, `.md-code-mark`,
`.md-task-marker`, etc. — which let you dim or restyle the literal `#`, `-`, `*` and
`` ` `` characters. Leave a rule empty (`{}`) to inherit the default; the file keeps a
few empty placeholders as documentation of what's stylable.

> This file uses **native CSS nesting** (no preprocessor). Inkdrop 6 ships modern CSS, so
> nesting, `@layer`, and `hsl(... / alpha)` all work without a build step.

---

## Migrating an older theme

Porting a theme from Inkdrop 5 (CodeMirror 5, usually authored in LESS) to Inkdrop 6 /
CodeMirror 6? See **[Plugin Migration Guide from v5 to v6 → Syntax themes](https://developer.inkdrop.app/appendix/plugin-migration-from-v5-to-v6#syntax-themes)**.

It covers the `.CodeMirror-*` → `--editor-*` variable mapping and the `.cm-*` → `.tok-*`
token-class mapping. The [anatomy](#anatomy-of-stylesindexcss) above is what you're
migrating _to_.

---

## Reference

- Primitive color tokens — <https://github.com/inkdropapp/css/blob/main/tokens.css>
- This theme's source — [`styles/index.css`](./styles/index.css)
- CodeMirror 6 themes (the underlying npm packages) —
  [inkdropapp/cm6-themes](https://github.com/inkdropapp/cm6-themes)
- Inkdrop documentation — <https://docs.inkdrop.app/>

## License

[MIT](./LICENSE)
