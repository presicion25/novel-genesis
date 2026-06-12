# NovelGenesis — AI & API Reference

All AI calls go through the global `AI` object. API keys are stored in localStorage (XOR-obfuscated) and called directly from the browser — no proxy server.

---

## AI.call()

```js
const r = await AI.call(provider, userPrompt, opts);
// r.text   → string (AI response text)
// r.error  → string | null (error message if call failed)
```

### Parameters

| Param | Type | Description |
|---|---|---|
| `provider` | `'claude'` \| `'openai'` \| `'gemini'` | Provider to use |
| `userPrompt` | `string` | User message |
| `opts.system` | `string?` | System prompt |
| `opts.model` | `string?` | Model ID (falls back to provider default) |

### Usage Pattern

```js
// 1. Get provider (check key availability first)
const keys = Store.getKeys();
const defaultProv = keys.claude ? 'claude' : keys.openai ? 'openai' : keys.gemini ? 'gemini' : 'claude';

// 2. Show loading state
outputEl.innerHTML = '<span class="ai-spinner"></span> Generating…';

// 3. Call
const r = await AI.call(provider, userPrompt, { system: systemPrompt, model: modelId });

// 4. Handle error
if (r.error) {
  outputEl.innerHTML = `<span style="color:var(--danger)">Error: ${r.error}</span>`;
  return;
}

// 5. Use r.text
```

### JSON Response Pattern

When the AI is expected to return JSON, always strip potential markdown fences:

```js
const r = await AI.call(provider, prompt, { system, model });
if (r.error) { /* handle */ return; }

let parsed;
try {
  const m = r.text.match(/\{[\s\S]*\}/);
  parsed = JSON.parse(m ? m[0] : r.text);
} catch (e) {
  // Fall back to showing raw text
  outputEl.textContent = r.text;
  return;
}
// use parsed
```

---

## AI.generateImage()

```js
const r = await AI.generateImage(provider, prompt, opts);
// r.image  → base64 data URL string (e.g. 'data:image/png;base64,...')
// r.error  → string | null
```

### Parameters

| Param | Type | Description |
|---|---|---|
| `provider` | `'openai'` \| `'google'` | Only these two support image gen |
| `prompt` | `string` | Image description prompt |
| `opts.n` | `number?` | Number of images (default 1) |
| `opts.size` | `string?` | e.g. `'1024x1024'` (OpenAI) |
| `opts.aspectRatio` | `string?` | e.g. `'9:16'` (Google) |

**Note:** Anthropic/Claude does NOT support image generation in this app.

---

## Text Models

### WRITING_MODELS object

```js
const WRITING_MODELS = {
  claude: [
    { id: 'claude-opus-4-8',          label: 'Opus 4.8' },
    { id: 'claude-sonnet-4-6',        label: 'Sonnet 4.6' },
    { id: 'claude-haiku-4-5-20251001',label: 'Haiku 4.5' },   // ★ cheapest
  ],
  openai: [
    { id: 'gpt-4o',       label: 'GPT-5.5' },       // displayed label differs from actual ID
    { id: 'gpt-4-turbo',  label: 'GPT-5.4' },
    { id: 'gpt-4o-mini',  label: 'GPT-5.4 Mini' },  // ★ cheapest
  ],
  gemini: [
    { id: 'gemini-3.5-flash',       label: 'Gemini 3.5 Flash' },  // ★ balanced
    { id: 'gemini-3.1-pro',         label: 'Gemini 3.1 Pro' },
    { id: 'gemini-3.1-flash-lite',  label: 'Gemini 3.1 Flash Lite' },
  ],
};
```

**Warning:** The OpenAI labels ("GPT-5.5", "GPT-5.4") are marketing names used in the UI — the actual API model IDs are `gpt-4o`, `gpt-4-turbo`, `gpt-4o-mini`. Do not change the IDs.

---

## Image Generation Models

| Provider | Model ID | Notes |
|---|---|---|
| OpenAI | `gpt-image-2` | 1024×1024, returns base64 |
| Google | `gemini-3.1-flash-image` | Returns base64 via Gemini multimodal API |

---

## API Endpoints (Direct Browser Calls)

| Provider | Endpoint |
|---|---|
| Anthropic | `https://api.anthropic.com/v1/messages` |
| OpenAI | `https://api.openai.com/v1/chat/completions` (text) · `https://api.openai.com/v1/images/generations` (images) |
| Google | `https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent` |

These are CORS-open endpoints — browsers can call them directly with no preflight issues using the proper headers.

---

## API Key Storage & Obfuscation

Keys are stored at `KEYS.apikeys` as an obfuscated object:

```js
// Store a key
const keys = Store.get(KEYS.apikeys) || {};
keys.claude = Store.obfuscate(rawKey);   // XOR with key=37
Store.set(KEYS.apikeys, keys);

// Read a key
const keys = Store.getKeys();   // auto-deobfuscates all keys
// keys.claude, keys.openai, keys.gemini, keys.elevenlabs
```

**Obfuscation algorithm (XOR, key=37):**
```js
Store.obfuscate = (str) => btoa(str.split('').map(c => String.fromCharCode(c.charCodeAt(0) ^ 37)).join(''));
Store.deobfuscate = (str) => atob(str).split('').map(c => String.fromCharCode(c.charCodeAt(0) ^ 37)).join('');
```

This is NOT encryption — it only prevents casual inspection. Anyone with DevTools can decode it.

---

## ElevenLabs TTS

Used in the Editor view for chapter narration / text-to-speech.

```js
const keys = Store.getKeys();
// keys.elevenlabs → API key string

// Direct API call pattern:
const resp = await fetch('https://api.elevenlabs.io/v1/text-to-speech/{voiceId}', {
  method: 'POST',
  headers: {
    'xi-api-key': keys.elevenlabs,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    text: selectedText,
    model_id: 'eleven_monolingual_v1',
    voice_settings: { stability: 0.5, similarity_boost: 0.75 },
  }),
});
const audioBlob = await resp.blob();
const audioUrl = URL.createObjectURL(audioBlob);
// play via <audio> element
```

---

## Voice Profile Injection

If a voice profile exists for the active project (`Store.get(KEYS.voice(pid))`), AIModule injects it into the system prompt automatically:

```js
const voice = Store.get(KEYS.voice(pid));
if (voice?.summary) {
  systemPrompt += `\n\nAuthor's voice profile: ${voice.summary}. POV: ${voice.pov}. Tense: ${voice.tense}. Match this style.`;
}
```

This happens in AIModule before calling `AI.call()`. When editing AIModule, preserve this injection.

---

## AI Generation Response Structure

All modules that call the AI expect one of two response forms:

### 1. Structured JSON (most modules)

The AI is prompted to return a specific JSON schema. After receiving `r.text`:
```js
const m = r.text.match(/\{[\s\S]*\}/);
const obj = JSON.parse(m ? m[0] : r.text);
```

Common JSON schemas used:
- **CharModule:** `{name, role, age, personality, backstory, goals, appearance}`
- **WorldModule:** `{name, type, description, history, significance}`
- **SceneModule:** `{title, setting, mood, conflict, beats: string[]}`
- **OutlineModule:** `{acts: [{title, beats: string[]}]}`
- **StyleAnalyzer:** `{summary, pov, sentenceStyle, tense, tone, vocabularyLevel, strengths[], distinctive[], recommendations[]}`

### 2. Plain text (BlurbGen, some AI Assistant modes)

Raw text is rendered directly — no JSON parse. Split by regex or display as-is.

---

## Provider Availability Gates

Before showing any AI feature, check key availability:

```js
const keys = Store.getKeys();
const hasKey = !!(keys.claude || keys.openai || keys.gemini);

if (!hasKey) {
  // Show warning banner with link to apisettings
  wrap.innerHTML += `<div class="card mb-3" style="border-color:var(--warn)">
    <div style="color:var(--warn)">⚠️ No AI API key set. <a href="#" onclick="Router.navigate('apisettings')">Add keys →</a></div>
  </div>`;
}

// Per-feature image gen gate (only openai + google support it)
const hasImageGen = !!(keys.openai || keys.gemini);
```

Default provider selection:
```js
const defaultProv = keys.claude ? 'claude' : keys.openai ? 'openai' : keys.gemini ? 'gemini' : 'claude';
```

---

## Template System for AI Calls

Templates (`KEYS.templates` + `DEFAULT_TEMPLATES`) have an optional `system` field used as the AI system prompt:

```js
const t = TemplatesModule.getById(id);
const r = await AI.call(provider, t.prompt, {
  system: t.system || undefined,
  model: modelId,
});
```

`TemplatesModule._use(id)` sets `State.pendingAITemplate = t` and navigates to the `ai` view, where AIModule picks it up on mount.
