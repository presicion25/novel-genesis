# NovelGenesis — Claude Reference Guide

Reference context for working on `novelgenesis (with API).html`.
Load this when editing that file: Claude Code users can run `claude --context novelgenesis-CLAUDE.md`.

---

## What This App Is

**NovelGenesis** is a single-file, local-first HTML app (~5000 lines). It is a novel-writing companion with multi-provider AI assistance. There is no backend, no build step, no npm. Everything lives in one `.html` file opened directly in a browser via `file://`.

**Stack:** Vanilla JS (no frameworks) · CSS custom properties · localStorage (all persistence) · Direct browser calls to Anthropic / OpenAI / Google AI · ElevenLabs TTS

---

## Critical Rules

- **One file only.** All HTML, CSS, and JS lives in `novelgenesis (with API).html`. Never split into separate files.
- **No frameworks.** No React, Vue, Svelte, etc. Vanilla JS only.
- **No npm / no build.** The file must open in a browser directly without any toolchain.
- **localStorage is the only database.** No server, no IndexedDB.
- **Direct API calls.** AI providers are called directly from the browser (`fetch`). No proxy.
- **API keys stored obfuscated.** Keys are XOR-obfuscated (key=37). Never store them plain. Use `Store.obfuscate()` / `Store.deobfuscate()`.
- **No external libraries.** No CDN imports. Everything inlined.

---

## Global Architecture

The app boots from a single IIFE at the bottom: `(function init(){...})()`.

### Core Global Objects

| Object | Purpose |
|---|---|
| `Store` | localStorage read/write/remove + key obfuscation |
| `State` | Runtime state: `activeProjectId`, `currentView`, `project`, `dirty`, `autosaveTimer` |
| `Router` | SPA navigation: `Router.register(name, fn)` · `Router.navigate(name)` |
| `Modal` | Dialog: `Modal.open(title, bodyEl, footerEl)` · `Modal.close()` · `Modal.confirm(title, msg)→Promise<bool>` |
| `Toast` | Toasts: `Toast.show(msg, type, duration)` — types: `'success'`, `'error'`, `'info'` |
| `Theme` | CSS theme: `Theme.apply(name)` · `Theme.current()` · `Theme.showPicker()` |
| `Projects` | CRUD: `Projects.all()` · `Projects.create(data)` · `Projects.update(id, patch)` · `Projects.delete(id)` · `Projects.wordCount(pid)` |
| `AI` | AI calls: `AI.call(provider, prompt, opts)` · `AI.generateImage(provider, prompt, opts)` |
| `Search` | `Search.open()` · `Search.query(q)→results[]` · `Search.close()` |
| `AutoSave` | `AutoSave.start()` — polls every 3s, saves editor if `State.dirty` |
| `ExportModule` | `ExportModule.showDialog()` — PDF/DOCX/EPUB/JSON export |

### Window-Attached Modules (set inside Router.register callbacks)

| Global | View |
|---|---|
| `window.WorldModule` | worldbuilding |
| `window.CharModule` | characters |
| `window.NarrativeModule` | narrative |
| `window.SceneModule` | scenes |
| `window.OutlineModule` | outline |
| `window.EditorModule` | editor (chapters/writing) |
| `window.AIModule` | ai assistant |
| `window.CoverGen` | covergen |
| `window.TemplatesModule` | templates |
| `window.ApiSettings` | apisettings |
| `window.SettingsModule` | settings |
| `window.SavedModule` | saved/portfolio |
| `window.BlurbGen` | blurbgen |
| `window.StyleAnalyzer` | styleanalyzer |
| `window.RoyaltyCalc` | royaltycalc |

---

## Router Pattern

```js
// Register a view
Router.register('viewname', () => {
  const wrap = el('div', 'fade-in');
  // build DOM
  wrap.innerHTML = `...`;
  // wire events
  window.MyModule = { _method() { ... } };
  return wrap;  // MUST return the element
});

// Navigate
Router.navigate('viewname');
```

**Registered views:** `dashboard`, `worldbuilding`, `characters`, `narrative`, `outline`, `scenes`, `editor`, `ai`, `templates`, `covergen`, `saved`, `royaltycalc`, `blurbgen`, `styleanalyzer`, `apisettings`, `settings`, `help`

Sidebar nav items use `data-view="viewname"` — must match exactly.

---

## Helper Functions

```js
el(tag, className, textContent)  // → HTMLElement
h(tag, attrs, children)          // → HTMLElement with attrs + children array
uid()                            // → random ID string
wc(text)                         // → word count integer
readingLevel(text)               // → Flesch-Kincaid grade string
today()                          // → 'YYYY-MM-DD'
escXml(str)                      // → XML-escaped string (for DOCX/EPUB export)
noProjectEl()                    // → empty-state element with "no project" message
projectSelectEl(curPid, onChange)// → project <select> element
```

---

## Coding Conventions

1. **DOM construction:** `el()` for simple nodes; `innerHTML` template literals for complex markup; `h()` for attrs-heavy nodes.
2. **Inline onclick are acceptable** for module method calls: `onclick="MyModule._action()"`.
3. **All module public APIs** go on `window.ModuleName = { ... }` inside the register callback.
4. **No `document.write`, no `eval()`, no dynamic script injection.**
5. **Error handling:** Failed async → `Toast.show('msg', 'error')`.
6. **Confirm before destructive ops:** `Modal.confirm(title, msg).then(ok => { if(!ok) return; ... })`.
7. **Loading states on AI calls:** Show `<span class="ai-spinner"></span>` before `await AI.call(...)`.
8. **Dirty flag:** Call `State.setDirty(true)` after data mutations. AutoSave picks up every 3s.

---

## Store API

```js
Store.get(key)           // → parsed value | null
Store.set(key, value)    // stringify to localStorage
Store.remove(key)        // removeItem
Store.getKeys()          // → {claude, openai, gemini, elevenlabs} (deobfuscated)
Store.obfuscate(str)     // XOR key=37 → stored form
Store.deobfuscate(str)   // reverse
```

---

## State Object

```js
State.activeProjectId    // string | null — current project
State.project            // project object | null (derived from activeProjectId)
State.currentView        // string — registered route name
State.dirty              // boolean
State.setDirty(bool)
State.autosaveTimer      // setInterval handle
State.pendingAITemplate  // Template object pre-loaded for AI view (set by TemplatesModule._use)
```

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Escape` | Close modal / close search / exit focus mode |
| `Ctrl+/` | Open search overlay (outside inputs) |
| `Ctrl+S` | Save current editor chapter |
| `F11` | Toggle focus/zen mode in editor |

---

## Export Formats

| Format | Implementation | Notes |
|---|---|---|
| PDF | `window.print()` | Browser print → save as PDF |
| DOCX | Raw Word XML Blob | Opens in LibreOffice/Word. Not a true zip. |
| EPUB | XHTML file download | Rename `.epub.html` → `.xhtml` for ebook readers |
| JSON | Project data object | Full backup: project + world + chars + narrative + scenes + chapters |

---

## Common Gotchas

- **No active project guard:** Most views check `State.activeProjectId` and return `noProjectEl()` if null.
- **KEYS are functions for per-project data:** Use `KEYS.world(pid)`, not `KEYS.world`. See `novelgenesis-data-schema.md`.
- **Template system:** `DEFAULT_TEMPLATES` array is hardcoded read-only. User templates are in localStorage. `getTemplates()` merges both. Never write `DEFAULT_TEMPLATES` items to storage.
- **CoverGen `KEYS.covers` is global (not per-project):** Covers stored as base64 strings — can be large.
- **Voice profile (`KEYS.voice(pid)`):** JSON object from StyleAnalyzer (pov, tone, sentenceStyle, etc.) injected into AI prompts — not an audio file.
- **Royalty Calculator:** Purely computational, no storage, no AI.
- **localStorage ~5-10 MB limit:** Long novels can approach this. Export JSON regularly.
- **AI JSON parse pattern:** After every `AI.call()` that expects JSON: `const m = r.text.match(/\{[\s\S]*\}/); const parsed = JSON.parse(m ? m[0] : r.text);` — strips markdown fences.
- **Provider availability:** Check `Store.getKeys()` before showing AI features. Gates: `keys.claude`, `keys.openai`, `keys.gemini`, `keys.elevenlabs`.

---

## Supplementary Reference Files

| File | Contents |
|---|---|
| `novelgenesis-data-schema.md` | localStorage KEYS object, per-project data shapes, project CRUD details |
| `novelgenesis-ai-api.md` | AI.call / AI.generateImage signatures, model IDs, API endpoints, ElevenLabs TTS |
| `novelgenesis-css-theming.md` | CSS custom properties, 6 themes, utility classes, styling patterns |
