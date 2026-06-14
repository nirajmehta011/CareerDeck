# 📋 task_strategy.md — CareerDeck Build Strategy
> Framework: B.L.A.S.T. | App: CareerDeck | Delivery: Single index.html

---

## 🎯 Project Goal (North Star)
Build **CareerDeck** — a world-class, single-file, AI-powered job application tracker that works by opening `index.html` in any browser. It must feel as polished as Linear, as intelligent as Notion AI, and support Gemini/OpenAI/Anthropic as interchangeable LLM backends.

---

## 📌 Phases Overview

| Phase | Name | Status |
|---|---|---|
| Protocol 0 | Initialization | ✅ Complete |
| Phase 1: B | Blueprint | ✅ Complete |
| Phase 2: L | Link | ✅ Complete |
| Phase 3: A | Architect | ✅ Complete |
| Phase 4: S | Stylize | ⏳ Pending |
| Phase 5: T | Trigger | ⏳ Pending |

---

## 🏗️ Phase 3 Build Order (13 Steps from Objective)

### Step 1 — HTML Shell + Design System
- [x] Full `<head>` with CDN: React 18, ReactDOM, Lucide, Inter/JetBrains Mono fonts
- [x] `<style>` block: all CSS custom properties, keyframes, global resets
- [x] Root mount + AppProvider

### Step 2 — State & Data Layer
- [x] AppContext with `useReducer`
- [x] Initial state shape (applications, activeView, modals, toasts, filters, commandPalette)
- [x] localStorage hydration + debounced persistence (300ms)
- [x] Sample data injection (8 jobs, `careerdeck_initialized` guard)

### Step 3 — Sidebar
- [x] Navigation items (Kanban, List, Analytics, Settings)
- [x] Active state styling
- [x] Mini stats: total jobs, interviews this week, offers

### Step 4 — Kanban View (Core Feature)
- [x] 6 columns (saved → closed) with stage colors
- [x] Job card component with accent bar, logo, match score
- [x] Native HTML5 drag-and-drop
- [x] Drop target highlighting
- [x] Column empty states

### Step 5 — AddJobModal
- [x] Tab toggle: URL Import | Manual Entry
- [x] URL Import: CORS proxy fetch → AI parse → form fill
- [x] Manual Entry: all fields as specced
- [x] AI job parser (callLLM with JSON schema)

### Step 6 — JobDetailPanel (Slide-in, 480px)
- [x] 5 tabs: Overview | AI Tools | Activity | Interview | Documents
- [x] Stage selector pill buttons
- [x] Priority toggle
- [x] Contacts section
- [x] Activity log with icons

### Step 7 — AI Feature Modals (7 total)
- [x] Feature 1: URL Job Parser (used inside AddJobModal)
- [x] Feature 2: Resume Tailor (side-by-side diff)
- [x] Feature 3: ATS Scanner (circular score dial SVG)
- [x] Feature 4: Interview Coach (question cards + STAR scaffold)
- [x] Feature 5: Outreach Writer (3 tabs: cold email, LinkedIn, follow-up)
- [x] Feature 6: Match Score (auto-run on job add, chip on card)
- [x] Feature 7: Weekly Momentum Digest (streamed inline in Analytics)

### Step 8 — List View
- [x] Sortable table (all columns)
- [x] Filter bar (search + stage pills + tags + remote toggle)
- [x] Bulk select + actions
- [x] CSV export

### Step 9 — Analytics View
- [x] 6 stage metric cards
- [x] SVG conversion funnel
- [x] SVG line chart (30-day applications)
- [x] SVG bar chart (day-of-week response rate)
- [x] Weekly Momentum Digest (streamed inline)
- [x] Auto-generated insights list

### Step 10 — Settings View
- [x] LLM Provider selector (Gemini / OpenAI / Anthropic) + model dropdown
- [x] API key input (masked, show/hide)
- [x] Test connection button
- [x] Base resume textarea
- [x] Preferences toggles
- [x] Export/Import/Clear data

### Step 11 — Command Palette (⌘K)
- [x] Full-screen overlay
- [x] Cross-search: companies, roles, tags, notes
- [x] Quick actions
- [x] Recent 5 jobs

### Step 12 — Toast Notification System
- [x] 5 toast types (success, error, warning, info, ai)
- [x] Max 3 visible, queue overflow
- [x] Action button support (e.g., "Write follow-up")

### Step 13 — Polish
- [x] All keyboard shortcuts (⌘K, N, Escape, ⌘1-3, /, Tab)
- [x] Reminder system on mount (follow-up detection)
- [x] Clearbit logo fetching + initials fallback + localStorage cache
- [x] `prefers-reduced-motion` support
- [x] ARIA labels on all icon-only buttons
- [x] Mobile responsive layout

---

## 🔑 Key Decisions Made (Phase 1)
| Decision | Choice | Rationale |
|---|---|---|
| LLM Provider | Multi-provider (Gemini/OpenAI/Anthropic) | User-configurable in Settings |
| Default Provider | Gemini (`gemini-2.0-flash`) | User's preference |
| URL Import Strategy | CORS proxy + paste fallback | Balance reliability and UX |
| App Name | **CareerDeck** | User's branding preference |
| Output Location | `Job Tracker AI Application/index.html` | User-specified |
| Build Approach | Full spec, no compromises | User's explicit request |

---

## ✅ Quality Checklist (Final Gate)
- [ ] App loads with sample data and looks beautiful immediately
- [ ] Can add a job manually in < 30 seconds
- [ ] URL import attempts proxy fetch, falls back to paste
- [ ] Drag-and-drop works and persists on refresh
- [ ] All 7 AI features produce useful, streamed output
- [ ] LLM provider switchable at runtime in Settings
- [ ] Closing and reopening browser restores all data exactly
- [ ] No console errors in normal usage
- [ ] Works on mobile
- [ ] Dark theme consistent — no white flash
- [ ] Every empty state is helpful
