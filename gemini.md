# 📜 gemini.md — CareerDeck Project Constitution
> Last Updated: Phase 1 — Blueprint
> App Name: **CareerDeck**
> Delivery: Single `index.html` file — no build step, no bundler

---

## 🏗️ Architectural Invariants (DO NOT VIOLATE)

1. **Single-file delivery** — Everything lives in one `index.html`. No external JS/CSS files written by us. CDN imports only.
2. **No build step** — Must open directly in any browser without npm, webpack, or any compiler.
3. **No external database** — `localStorage` is the only persistence layer.
4. **LLM-agnostic** — AI calls go through a unified `callLLM()` helper that routes to the user's selected provider (Gemini, OpenAI, or Anthropic). The provider + model + API key are stored in localStorage.
5. **Deterministic UI** — All state transitions must be predictable. AI is used only for content generation, never for routing or business logic decisions.
6. **Client-side only** — No server, no backend, no Node.js. Everything runs in the browser.

---

## 🤖 LLM Provider Configuration

### Supported Providers
| Provider | Default Model | API Base URL |
|---|---|---|
| Gemini (Google) | `gemini-2.0-flash` | `https://generativelanguage.googleapis.com/v1beta/models/` |
| OpenAI | `gpt-4o` | `https://api.openai.com/v1/chat/completions` |
| Anthropic | `claude-sonnet-4-6` | `https://api.anthropic.com/v1/messages` |

### Provider Schema (stored in localStorage `careerdeck_user`)
```json
{
  "llmProvider": "gemini",
  "llmModel": "gemini-2.0-flash",
  "apiKey": "",
  "availableModels": {
    "gemini": ["gemini-2.0-flash", "gemini-1.5-pro", "gemini-1.5-flash"],
    "openai": ["gpt-4o", "gpt-4o-mini", "gpt-4-turbo"],
    "anthropic": ["claude-sonnet-4-6", "claude-3-5-haiku-20241022", "claude-opus-4-5"]
  }
}
```

### Unified `callLLM()` Contract
```javascript
async function callLLM(systemPrompt, userMessage, onStream, config)
// config: { provider, model, apiKey }
// onStream(text): called with accumulated text during streaming
// Returns: full final text string
// Throws: LLMError with .type: 'no_key' | 'rate_limit' | 'network' | 'parse_error'
```

---

## 📦 Data Schemas

### Application Object
```json
{
  "id": "uuid-v4",
  "companyName": "string",
  "roleName": "string",
  "jobUrl": "string",
  "stage": "saved | applied | screening | interview | offer | closed",
  "appliedAt": "ISO-date | null",
  "createdAt": "ISO-date",
  "updatedAt": "ISO-date",
  "location": "string",
  "remote": "boolean",
  "salaryMin": "number | null",
  "salaryMax": "number | null",
  "jdText": "string",
  "jdSummary": "string",
  "tags": ["string"],
  "matchScore": "number(0-100) | null",
  "atsSuggestions": ["string"],
  "resumeText": "string",
  "coverLetter": "string",
  "contacts": [{"name": "string", "role": "string", "linkedIn": "string", "note": "string"}],
  "notes": "string",
  "interviewRounds": [{"round": "string", "date": "ISO-date", "notes": "string", "questions": ["string"]}],
  "followUpDate": "ISO-date | null",
  "followUpSent": "boolean",
  "priority": "low | medium | high",
  "companyLogo": "string (URL)",
  "activityLog": [{"id": "uuid", "event": "string", "timestamp": "ISO-date", "source": "user|ai|system", "detail": {}}]
}
```

### Activity Log Event
```json
{
  "id": "uuid-v4",
  "event": "stage_changed | note_added | ai_used | doc_added | reminder_set",
  "timestamp": "ISO-date",
  "source": "user | ai | system",
  "detail": {}
}
```

### User/Settings Object (`careerdeck_user`)
```json
{
  "name": "string",
  "email": "string",
  "llmProvider": "gemini | openai | anthropic",
  "llmModel": "string",
  "apiKey": "string",
  "baseResume": "string",
  "preferences": {
    "defaultStage": "saved",
    "reminderLeadDays": 5,
    "weeklyDigestEnabled": true
  }
}
```

---

## 🎨 Design System Tokens

### Color Palette
```css
--color-bg:          #0F1117;   /* near-black canvas */
--color-surface:     #161B27;   /* card surface */
--color-surface-2:   #1E2535;   /* hover/elevated */
--color-border:      #2A3347;   /* dividers */
--color-accent:      #4F8EF7;   /* electric blue — primary CTA */
--color-accent-glow: rgba(79,142,247,0.15);
--color-success:     #34D399;   /* offers */
--color-warning:     #FBBF24;   /* follow-ups */
--color-danger:      #F87171;   /* rejections */
--color-text-1:      #F1F5F9;   /* primary */
--color-text-2:      #94A3B8;   /* secondary */
--color-text-3:      #4A5568;   /* placeholder */
```

### Stage Color Map
```
saved:      #64748B (slate)
applied:    #4F8EF7 (blue)
screening:  #A78BFA (violet)
interview:  #FBBF24 (amber)
offer:      #34D399 (green)
closed:     #F87171 (red)
```

---

## ⚙️ localStorage Keys

| Key | Contents |
|---|---|
| `careerdeck_applications` | JSON array of Application objects |
| `careerdeck_user` | User profile + LLM config + API key |
| `careerdeck_base_resume` | Plain text master resume |
| `careerdeck_activity_log` | Global activity event array |
| `careerdeck_initialized` | "true" flag to skip sample data injection |
| `careerdeck_logo_cache` | `{ "domain.com": "https://..." }` |

---

## 🚫 Behavioral Rules (DO NOT Rules)

1. **DO NOT** hardcode any LLM model or provider — always read from user settings.
2. **DO NOT** block UI render on localStorage reads — hydrate state after first paint.
3. **DO NOT** use any external state management library (Redux, Zustand, Jotai, etc.).
4. **DO NOT** make API calls without first checking that `apiKey` is set — show Settings prompt instead.
5. **DO NOT** display raw JSON errors to users — always map to friendly messages.
6. **DO NOT** invent facts in AI prompts — explicitly instruct the LLM to preserve factual accuracy.
7. **DO NOT** write to `tools/` — this is a browser-only single-file app, no file system access.

---

## 🔗 External Integrations

| Service | Purpose | Auth Required | Fallback |
|---|---|---|---|
| Clearbit Logo API | Company logos | None (free) | Initials avatar |
| CORS Proxy (allorigins.win) | URL job import | None | Manual paste fallback |
| Gemini API | AI features (default) | API Key | — |
| OpenAI API | AI features (optional) | API Key | — |
| Anthropic API | AI features (optional) | API Key | — |

---

## 🗺️ Maintenance Log
- `2026-06-13` — Project Constitution initialized (Phase 1 Blueprint approved)
