# NovelGenesis — Data Schema & localStorage Reference

All data is stored in `localStorage`. No server. No IndexedDB.

---

## KEYS Object

```js
const KEYS = {
  // Global (not per-project)
  projects:   'ng_projects',   // Project[] — master list
  active:     'ng_active',     // string — active project ID
  settings:   'ng_settings',   // Settings object
  apikeys:    'ng_apikeys',    // obfuscated API keys object
  templates:  'ng_templates',  // Template[] — user-created templates
  covers:     'ng_covers',     // CoverEntry[] — book cover images (global)

  // Per-project (functions that take pid)
  world:      pid => `ng_world_${pid}`,      // WorldEntry[]
  chars:      pid => `ng_chars_${pid}`,      // Character[]
  narrative:  pid => `ng_narrative_${pid}`,  // NarrativeData object
  scenes:     pid => `ng_scenes_${pid}`,     // Scene[]
  chapters:   pid => `ng_chapters_${pid}`,   // Chapter[]
  stats:      pid => `ng_stats_${pid}`,      // StatsData object
  outline:    pid => `ng_outline_${pid}`,    // OutlineData object
  voice:      pid => `ng_voice_${pid}`,      // VoiceProfile object (from StyleAnalyzer)
  genLibrary: pid => `ng_genlib_${pid}`,     // GenerationEntry[] (AI generation history)
};
```

**Always use the function form for per-project keys.** `KEYS.world(pid)` not `KEYS.world`.

---

## Project Object

```js
{
  id: string,           // uid() — e.g. 'abc123'
  name: string,
  genre: string,        // e.g. 'Dark Fantasy'
  description: string,
  targetWords: number,  // word count goal
  archived: boolean,    // soft delete
  createdAt: string,    // ISO date
  updatedAt: string,    // ISO date
}
```

Stored as `Project[]` at `KEYS.projects`. `KEYS.active` holds the active project's `id`.

### Projects API

```js
Projects.all()              // → Project[] (all, including archived)
Projects.create(data)       // creates + saves, returns new Project
Projects.update(id, patch)  // merges patch, updates updatedAt
Projects.delete(id)         // removes from array entirely
Projects.wordCount(pid)     // → total word count across all chapters
```

---

## Per-Project Data Shapes

### WorldEntry (`KEYS.world(pid)`)

```js
[{
  id: string,
  name: string,
  type: string,       // 'location' | 'faction' | 'lore' | 'history' | etc.
  description: string,
  notes: string,
  createdAt: string,
}]
```

### Character (`KEYS.chars(pid)`)

```js
[{
  id: string,
  name: string,
  role: string,         // e.g. 'Protagonist', 'Antagonist', 'Supporting'
  age: string,
  personality: string,
  backstory: string,
  goals: string,
  appearance: string,
  notes: string,
  createdAt: string,
}]
```

### NarrativeData (`KEYS.narrative(pid)`)

```js
{
  premise: string,      // core story premise / synopsis
  theme: string,
  tone: string,
  setting: string,
  logline: string,
  notes: string,
}
```

### Scene (`KEYS.scenes(pid)`)

```js
[{
  id: string,
  title: string,
  status: string,       // 'idea' | 'draft' | 'written' | 'revised'
  pov: string,          // point-of-view character
  location: string,
  notes: string,
  content: string,      // scene prose (may be long)
  order: number,
  createdAt: string,
}]
```

### Chapter (`KEYS.chapters(pid)`)

```js
[{
  id: string,
  title: string,
  content: string,      // HTML string from contenteditable editor
  order: number,
  wordCount: number,
  createdAt: string,
  updatedAt: string,
}]
```

`content` is HTML (from the `contenteditable` rich-text editor). When exporting, use `stripHtml()` to get plain text:
```js
function stripHtml(html) {
  const d = document.createElement('div');
  d.innerHTML = html;
  return d.innerText || '';
}
```

### OutlineData (`KEYS.outline(pid)`)

```js
{
  // Structure varies — typically beat sheet or act breakdown
  acts: [{
    id: string,
    title: string,
    beats: [{
      id: string,
      text: string,
      done: boolean,
    }],
  }],
  notes: string,
}
```

### StatsData (`KEYS.stats(pid)`)

```js
{
  dailyWords: {
    'YYYY-MM-DD': totalWordCount,   // snapshot per day
  },
  streak: number,   // writing streak days
}
```

### VoiceProfile (`KEYS.voice(pid)`)

Set by StyleAnalyzer. Injected into AI prompts automatically by AIModule.

```js
{
  summary: string,          // one-sentence voice description for AI injection
  pov: string,              // 'First Person' | 'Third Limited' | etc.
  sentenceStyle: string,    // 'Short and punchy' | 'Long and complex' | etc.
  tense: string,            // 'Present' | 'Past' | 'Mixed'
  tone: string,             // emotional register description
  vocabularyLevel: string,  // 'Simple' | 'Accessible' | 'Sophisticated' | 'Dense'
  strengths: string[],
  distinctive: string[],
  recommendations: string[],
}
```

### GenerationEntry (`KEYS.genLibrary(pid)`)

AI generation history stored per project:

```js
[{
  id: string,
  type: string,       // e.g. 'character', 'scene', 'worldbuilding'
  prompt: string,
  result: string,     // raw AI text output
  model: string,
  ts: string,         // ISO date
}]
```

---

## Global Data Shapes

### Settings (`KEYS.settings`)

```js
{
  autoSave: boolean,        // default: true
  fontSize: number,         // editor font size in px (default: 16)
  defaultProvider: string,  // 'claude' | 'openai' | 'gemini'
  defaultModel: string,     // model ID string
  theme: string,            // theme name
}
```

### API Keys (`KEYS.apikeys`)

Stored XOR-obfuscated. Access via `Store.getKeys()` which deobfuscates automatically:

```js
Store.getKeys() // → { claude: string|null, openai: string|null, gemini: string|null, elevenlabs: string|null }
```

To save keys, obfuscate first:
```js
const keys = Store.get(KEYS.apikeys) || {};
keys.claude = Store.obfuscate(rawApiKey);
Store.set(KEYS.apikeys, keys);
```

### Template (`KEYS.templates`)

User-created templates only. Built-in templates live in `DEFAULT_TEMPLATES` array (hardcoded, never persisted):

```js
[{
  id: string,
  name: string,
  category: string,   // 'worldbuilding' | 'characters' | 'scene' | 'story'
  system: string,     // AI system prompt (role framing)
  prompt: string,     // user prompt body
  createdAt: string,
  updatedAt?: string,
}]
```

`getTemplates()` returns `[...userTemplates, ...DEFAULT_TEMPLATES]`.

### CoverEntry (`KEYS.covers`)

Global (not per-project). Covers are saved from the Book Cover Generator:

```js
[{
  id: string,
  image: string,     // base64 data URL (can be large — ~100-300KB per image)
  prompt: string,    // the prompt used to generate it
  title: string,     // book title at time of save
  ts: string,        // ISO date
}]
```

---

## JSON Export / Import Format

The `SavedModule._export(pid)` function produces:

```js
{
  project: Project,
  world: WorldEntry[] | null,
  chars: Character[] | null,
  narrative: NarrativeData | null,
  scenes: Scene[] | null,
  chapters: Chapter[] | null,
}
```

Exported as `{projectName}.novelgenesis.json`.

Import (in SavedModule / Portfolio view) reads this structure back and restores all keys.

**Note:** Stats, outline, voice profile, genLibrary, and covers are NOT included in the JSON export. Only the core writing data.

---

## Storage Size Considerations

- `KEYS.covers` is the biggest risk — base64 images are ~100-300KB each.
- `KEYS.chapters(pid)` content grows with long manuscripts.
- `KEYS.genLibrary(pid)` can accumulate large AI outputs.
- Total localStorage budget: ~5-10 MB (browser-dependent).
- Warn users via Toast if storage write fails (catch `localStorage.setItem` exceptions).
