# 📓 findings.md — Research, Discoveries & Constraints
> App: CareerDeck | Updated: Phase 1 Blueprint

---

## 🔍 Key Findings from Objective Analysis

### Architecture
- Single `index.html` — no build step required. CDN-only dependencies.
- React 18 via CDN with Babel standalone for JSX transpilation in-browser.
- State: `useState` + `useReducer` + `useContext` — no external state library.
- All persistence: `localStorage` only.

### LLM Integration — Multi-Provider Approach
The objective specified Claude Sonnet only, but user wants a multi-provider system. Here's what we discovered:

**Gemini API (Default)**
- Endpoint: `https://generativelanguage.googleapis.com/v1beta/models/{model}:streamGenerateContent?key={apiKey}`
- Streaming: Server-Sent Events (SSE), similar to Anthropic
- JSON mode: supported via `responseMimeType: "application/json"`
- CORS: Gemini API allows direct browser calls ✅

**OpenAI API**
- Endpoint: `https://api.openai.com/v1/chat/completions`
- Streaming: SSE with `stream: true`
- CORS: Does NOT allow direct browser calls without proxy ⚠️
  - Workaround: Users must handle their own CORS proxy or use OpenAI's client-side SDK
  - Note: Will work if user has CORS disabled or uses a proxy

**Anthropic API**
- Endpoint: `https://api.anthropic.com/v1/messages`
- Streaming: SSE with `stream: true`
- CORS: Does NOT allow direct browser calls natively ⚠️
  - Requires `anthropic-dangerous-allow-browser: true` header
  - Will add this header; user accepts the risk by setting an API key

### URL Import / CORS Constraints ✅ DECIDED
- **Decision:** Paste-only approach. No CORS proxy.
- AddJobModal has two tabs: **Paste JD** (AI-parses pasted text) and **Manual Entry** (no AI)
- In Paste JD mode: user pastes full JD text → AI parses → form auto-fills → user confirms → save
- This is more reliable, simpler, and avoids all CORS issues entirely

### Clearbit Logo API
- `https://logo.clearbit.com/{domain}` — free, no auth, no CORS issues
- Returns 404 for unknown domains — must handle with `onError` → initials fallback
- Cache in `careerdeck_logo_cache` localStorage key

### Drag-and-Drop
- Native HTML5 DnD API used (no library)
- `dataTransfer.setData()` / `getData()` for job ID and from-stage
- Visual highlight on `onDragEnter` / `onDragLeave`
- `will-change: transform` on draggable cards for GPU acceleration

### SVG Charts (No Library)
- Conversion funnel: horizontal bars with % labels
- Line chart (30-day): path with viewBox, computed from application dates
- Bar chart (day-of-week): simple rect elements
- All charts: responsive with `viewBox` and `preserveAspectRatio`

---

## ⚠️ Known Constraints / Risks

| Constraint | Severity | Mitigation |
|---|---|---|
| OpenAI/Anthropic CORS | Medium | Anthropic: add `anthropic-dangerous-allow-browser` header. OpenAI: warn user in Settings. |
| URL scraping blocked by major job boards | Medium | CORS proxy attempt + graceful paste fallback UX |
| Single file size could get large (~3,000+ lines) | Low | Acceptable for single-file delivery; minification not required |
| Babel CDN in-browser transpilation is slow on first load | Low | Add loading indicator; subsequent renders are fast |
| `localStorage` 5MB limit | Very Low | Text-heavy data (resumes, JDs) could approach limit for heavy users — add warning |

---

## 📚 CDN Dependencies (Final List)

```html
<!-- React 18 + Babel (JSX transpilation) -->
<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

<!-- Tailwind CSS (via CDN) -->
<script src="https://cdn.tailwindcss.com"></script>

<!-- Lucide Icons (React) -->
<script src="https://unpkg.com/lucide-react@latest/dist/umd/lucide-react.js"></script>

<!-- Google Fonts: Inter + JetBrains Mono -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
```

---

## 🎨 Design Inspiration
- **Linear.app** — Kanban feel, instant interactions, monochromatic dark theme
- **Notion AI** — AI output streaming inline, useful not gimmicky
- **Vercel Dashboard** — Clean data density, consistent typography

---

## 📝 Sample Data (8 Jobs Pre-Loaded)
Pre-filled across all 6 stages so app looks alive on first open:
- Stripe → Senior Frontend Engineer (interview, 87%)
- Linear → Product Designer (screening, 72%)
- Vercel → Developer Advocate (applied, 91%)
- Anthropic → Software Engineer (saved, 95%)
- Figma → Engineering Manager (offer, 78%)
- Notion → Full Stack Engineer (applied, 83%)
- GitHub → Senior SWE (closed, 65%)
- Arc → iOS Engineer (screening, 55%)
