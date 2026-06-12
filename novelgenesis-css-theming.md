# NovelGenesis — CSS & Theming Reference

All styling uses CSS custom properties (variables) on the `:root` / `[data-theme]` selector. No Tailwind. No external CSS framework.

---

## Theme System

Themes are applied by setting `data-theme="themename"` on `<html>`. Managed by the `Theme` global:

```js
Theme.apply('midnight')    // sets data-theme attribute + saves to settings
Theme.current()            // → current theme name string
Theme.showPicker()         // opens theme picker modal
```

### 6 Available Themes

| Theme | Style | Key Colors |
|---|---|---|
| `midnight` | Default dark | Deep blue-black bg, indigo accent |
| `cyber` | Cyberpunk dark | Near-black bg, neon cyan/teal accent |
| `space` | Space dark | Deep navy bg, purple/violet accent |
| `classic` | Dark professional | Slate-dark bg, blue accent |
| `paper` | Light sepia | Cream/off-white bg, brown accent |
| `beige` | Light warm | Warm beige bg, amber accent |

---

## CSS Custom Properties

All color references in HTML/CSS must use these variables — never hardcode hex values.

### Core Variables

| Variable | Purpose | Example usage |
|---|---|---|
| `--bg` | Page/app background | `background: var(--bg)` |
| `--surface` | Card/panel background | `background: var(--surface)` |
| `--surface2` | Secondary surface (inputs, code blocks) | `background: var(--surface2)` |
| `--border` | Border color | `border: 1px solid var(--border)` |
| `--accent` | Primary accent (indigo/cyan/purple) | `color: var(--accent)` |
| `--accent2` | Secondary accent | `color: var(--accent2)` |
| `--text` | Primary text | `color: var(--text)` |
| `--text2` | Secondary/muted text | `color: var(--text2)` |
| `--danger` | Error/destructive | `color: var(--danger)` |
| `--success` | Success | `color: var(--success)` |
| `--warn` | Warning | `color: var(--warn)` |
| `--shadow` | Box shadow value | `box-shadow: var(--shadow)` |
| `--radius` | Default border radius | `border-radius: var(--radius)` |
| `--sidebar-w` | Sidebar width | `width: var(--sidebar-w)` |

### Editor Variable

| Variable | Purpose |
|---|---|
| `--editor-font-size` | Rich text editor font size (set from settings, e.g. `16px`) |

---

## Utility Classes

### Layout

| Class | CSS |
|---|---|
| `.flex` | `display: flex` |
| `.flex-col` | `flex-direction: column` |
| `.items-center` | `align-items: center` |
| `.justify-between` | `justify-content: space-between` |
| `.gap-2` | `gap: 8px` |
| `.gap-3` | `gap: 12px` |
| `.mt-3` | `margin-top: 12px` |
| `.mb-3` | `margin-bottom: 12px` |
| `.fade-in` | Opacity fade-in animation on mount |

### Card Pattern

```html
<div class="card">
  <div class="card-title">Title</div>
  <!-- content -->
</div>
```

`.card` → `background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px; box-shadow: var(--shadow)`

`.card-grid` → CSS grid, `repeat(auto-fill, minmax(280px, 1fr))`, gap 16px

### Buttons

| Class | Style | Use |
|---|---|---|
| `.btn` | Base button | Required on all buttons |
| `.btn-primary` | `--accent` bg, white text | Primary action |
| `.btn-secondary` | Surface bg, border | Secondary action |
| `.btn-danger` | `--danger` bg | Destructive action |
| `.btn-ghost` | Transparent, text only | Subtle action |
| `.btn-sm` | Smaller padding/font | Compact buttons |

```html
<button class="btn btn-primary">Save</button>
<button class="btn btn-secondary">Cancel</button>
<button class="btn btn-danger btn-sm">Delete</button>
```

### Form Elements

```html
<div class="form-group">
  <label class="form-label">Field Name</label>
  <input type="text" placeholder="...">
  <!-- or -->
  <textarea rows="4"></textarea>
  <!-- or -->
  <select>...</select>
</div>
```

`input`, `textarea`, `select` inside `.form-group` get automatic styling via CSS: `background: var(--surface2); border: 1px solid var(--border); color: var(--text); border-radius: calc(var(--radius) * 0.6)`

### Page Header

```html
<div class="page-header">
  <div>
    <div class="page-title">Page Title</div>
    <div class="page-sub">Subtitle / description</div>
  </div>
  <div class="flex gap-2">
    <!-- actions -->
  </div>
</div>
```

### Tabs

```html
<div class="tabs mb-3">
  <div class="tab active" data-tab="tab1">Tab 1</div>
  <div class="tab" data-tab="tab2">Tab 2</div>
</div>
```

Active tab gets `color: var(--accent); border-bottom: 2px solid var(--accent)`.

### Empty State

```html
<div class="empty-state">
  <div class="empty-icon">📝</div>
  <div class="empty-title">Nothing here yet</div>
  <p style="color:var(--text2)">Descriptive text</p>
  <button class="btn btn-primary mt-3">Create First Item</button>
</div>
```

### AI Spinner

```html
<span class="ai-spinner"></span> Generating…
```

Animated spinner shown during AI calls. Always pair with descriptive text.

---

## Inline Styling Conventions

When using inline `style=` (common in this codebase due to single-file architecture):

1. **Always use CSS variables** for colors, never hex:
   ```html
   style="color:var(--text2);background:var(--surface2)"  ✅
   style="color:#6b7280;background:#f3f4f6"               ❌
   ```

2. **Use `color-mix()` for tinted backgrounds:**
   ```html
   style="background:color-mix(in srgb,var(--accent) 10%,transparent)"
   style="background:color-mix(in srgb,var(--success) 10%,transparent)"
   ```

3. **Sidebar width:** Always use `var(--sidebar-w)` when referencing sidebar dimensions.

4. **Font sizes:** Use `px` or `em` literals — no Tailwind-style sizing classes.

---

## Theme Color Reference (Approximate)

These are approximate — the actual values are in the `<style>` block's `[data-theme]` rules.

### midnight (default dark)

```css
--bg: #0f1117;
--surface: #1a1d27;
--surface2: #242736;
--border: #2e3347;
--accent: #6366f1;   /* indigo */
--text: #e8eaf0;
--text2: #8b90a7;
--danger: #ef4444;
--success: #22c55e;
--warn: #f59e0b;
```

### paper (light)

```css
--bg: #f5f0e8;
--surface: #faf7f2;
--surface2: #ede8df;
--border: #d4c9b8;
--accent: #7c5c3a;   /* brown */
--text: #2c2418;
--text2: #6b5a47;
```

---

## Status / Badge Colors

Used across cards and indicators — always inline:

```html
<!-- Scene status -->
<span style="background:var(--surface2);color:var(--text2);padding:2px 8px;border-radius:12px;font-size:11px">idea</span>
<span style="background:color-mix(in srgb,var(--warn) 15%,transparent);color:var(--warn)">draft</span>
<span style="background:color-mix(in srgb,var(--accent) 15%,transparent);color:var(--accent)">written</span>
<span style="background:color-mix(in srgb,var(--success) 15%,transparent);color:var(--success)">revised</span>
```

---

## Focus Mode (Editor)

When `document.body.classList.contains('focus-mode')` is true:

- Sidebar hides
- Editor fills the viewport
- Minimal toolbar only

Toggle via `F11` keyboard shortcut or `EditorModule.toggleFocus()`.

---

## Print Styles

The `@media print` block in the style section:

- Hides sidebar, toolbar, buttons
- Shows only chapter content in readable form
- Used by `ExportModule._pdf()` which calls `window.print()`

Do not add non-print-safe styles inside the main content area without `@media not print` guards.
