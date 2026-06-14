# AI Job Tracker — Complete Build Prompt
### For Claude (Sonnet 4.6) · React Single-Page Application · Production-Grade

---

## OVERVIEW & MISSION

You are building **TrackFlow** — a world-class, AI-powered job application tracker that is the single best tool a job seeker can use. It must feel as polished as Linear, as intelligent as Notion AI, and as fast as a native app. Every interaction should be instant, every AI feature should feel genuinely useful (not gimmicky), and every empty state should guide the user toward their next action.

This is a **lightweight single-file React application** (HTML + React + Tailwind via CDN) with full AI capabilities powered by the Anthropic Claude API. No build step, no bundler — it must run by opening a single `.html` file in a browser or embedding in an artifact renderer.

---

## VISUAL DESIGN SYSTEM

### Palette
```
--color-bg:          #0F1117   (near-black canvas)
--color-surface:     #161B27   (card surface)
--color-surface-2:   #1E2535   (hover / elevated surface)
--color-border:      #2A3347   (subtle dividers)
--color-accent:      #4F8EF7   (electric blue — primary action)
--color-accent-glow: rgba(79,142,247,0.15)
--color-success:     #34D399   (offers, positive status)
--color-warning:     #FBBF24   (follow-up needed)
--color-danger:      #F87171   (rejection / closed)
--color-text-1:      #F1F5F9   (primary text)
--color-text-2:      #94A3B8   (secondary text)
--color-text-3:      #4A5568   (placeholder / disabled)
```

### Typography
- **Display / headings:** `Inter` (700 weight) — tight letter-spacing: -0.02em
- **Body:** `Inter` (400/500) — line-height: 1.6
- **Mono / code / data:** `JetBrains Mono` — used for match scores, ATS scores, API keys

### Signature design element
Each job card has a **left-side accent bar** whose color reflects the current stage — this is the single most memorable visual element. The bar is 3px wide, uses a gradient from the stage color to transparent, and pulses subtly via CSS animation when a stage changed in the last 24 hours.

### Stage color map
```
saved:      #64748B  (slate)
applied:    #4F8EF7  (blue accent)
screening:  #A78BFA  (violet)
interview:  #FBBF24  (amber)
offer:      #34D399  (green)
closed:     #F87171  (red)
```

### Motion principles
- All modals: fade-in + slide-up (150ms ease-out)
- Cards: no animation on load (instant feel)
- AI streaming text: cursor blink while streaming
- Stage transitions: card accent bar color animates over 300ms
- Toasts: slide in from bottom-right, auto-dismiss 3s

---

## APPLICATION ARCHITECTURE

### File structure (single HTML file)
```
index.html
├── <head>   — CDN imports: React 18, ReactDOM, Tailwind, Lucide icons, Inter font
├── <style>  — CSS custom properties, keyframe animations, global overrides
└── <body>
    └── <div id="root">
        └── React app mounted here
```

### React component tree
```
<App>
├── <Sidebar>          — navigation + user profile + stats summary
├── <MainView>         — switches between views based on activeView state
│   ├── <KanbanView>   — drag-and-drop pipeline (default view)
│   ├── <ListView>     — table view with sort/filter
│   ├── <AnalyticsView>— funnel chart + momentum score + metrics
│   └── <SettingsView> — API key, resume upload, email sync
├── <AddJobModal>      — manual entry + URL import
├── <JobDetailPanel>   — slide-in right panel with full job details
│   ├── <ActivityLog>
│   ├── <DocumentVault>
│   ├── <AIToolsPanel>
│   └── <InterviewNotes>
├── <AIModal>          — full-screen AI feature modals
│   ├── <ResumeTailor>
│   ├── <InterviewCoach>
│   ├── <ATSScanner>
│   └── <OutreachWriter>
├── <ToastStack>       — notification queue
└── <CommandPalette>   — ⌘K global search + quick actions
```

### State management
Use React `useState` + `useReducer` + `useContext`. No external state library. Structure:

```javascript
// Global app state shape
{
  applications: [],        // array of application objects
  activeView: 'kanban',    // 'kanban' | 'list' | 'analytics' | 'settings'
  selectedJobId: null,     // opens detail panel
  modals: {
    addJob: false,
    ai: { open: false, mode: null, jobId: null }
  },
  user: {
    name: '',
    apiKey: '',            // stored in localStorage
    baseResume: '',        // plain text, stored in localStorage
  },
  toasts: [],
  filters: { stage: 'all', search: '', tags: [] },
  isCommandPaletteOpen: false
}
```

### Persistence
Use `localStorage` for all data. Keys:
- `trackflow_applications` — JSON array
- `trackflow_user` — user profile + API key
- `trackflow_base_resume` — plain text resume
- `trackflow_activity_log` — array of activity events

On mount, hydrate from localStorage. On every state change, persist to localStorage (debounced 300ms).

---

## DATA MODELS

### Application object
```javascript
{
  id: crypto.randomUUID(),
  companyName: '',
  roleName: '',
  jobUrl: '',
  stage: 'saved',          // saved | applied | screening | interview | offer | closed
  appliedAt: null,         // ISO date string
  createdAt: new Date().toISOString(),
  updatedAt: new Date().toISOString(),
  location: '',
  remote: false,
  salaryMin: null,
  salaryMax: null,
  jdText: '',              // full job description text
  jdSummary: '',           // AI-generated 2-sentence summary
  tags: [],                // string array
  matchScore: null,        // 0–100
  atsSuggestions: [],      // array of missing keywords
  resumeText: '',          // tailored resume text for this job
  coverLetter: '',
  contacts: [],            // [{name, role, linkedIn, note}]
  notes: '',               // freeform notes
  interviewRounds: [],     // [{round, date, notes, questions[]}]
  followUpDate: null,
  priority: 'medium',      // low | medium | high
  companyLogo: '',         // URL (fetched from clearbit)
  activityLog: []          // [{event, timestamp, source, detail}]
}
```

### Activity log event
```javascript
{
  id: crypto.randomUUID(),
  event: 'stage_changed',   // stage_changed | note_added | ai_used | doc_added | reminder_set
  timestamp: new Date().toISOString(),
  source: 'user',           // user | ai | system
  detail: { from: 'applied', to: 'screening' }
}
```

---

## VIEWS & FEATURES — DETAILED SPECS

### 1. KANBAN VIEW (default)
```
Layout: horizontal scroll container with 6 columns (one per stage)
Each column:
  - Header: stage name + colored dot + application count badge
  - "Add job" quick-add button at top
  - Cards stacked vertically, drag-and-drop reorderable
  - Column background: subtle stage-color tint (5% opacity)

Job card anatomy:
  ┌─[accent bar]──────────────────────────────┐
  │ [Company logo 24px] Company Name    [⋮]   │
  │ Role Title                                │
  │ 📍 Location · 💰 Salary range            │
  │ [tag] [tag] [tag]                         │
  │ ─────────────────────────────────────────│
  │ Match: [███░░] 67%   Applied: 3 days ago  │
  └───────────────────────────────────────────┘

Interactions:
  - Click card body → open JobDetailPanel (slide in from right)
  - Drag card to new column → stage changes, activity logged, toast shown
  - ⋮ menu → Quick actions: Mark applied, Set reminder, Delete, Duplicate
  - "Add job" button → opens AddJobModal pre-filled with column's stage
  - Empty column → illustration + "Drop jobs here or add one" message
```

### 2. LIST VIEW
```
Table with columns: [Company] [Role] [Stage] [Location] [Match%] [Applied] [Follow-up] [Actions]
- Sortable by clicking column headers (asc/desc toggle)
- Filter bar: search input + stage pills + tag filter + remote toggle
- Row click → opens JobDetailPanel
- Bulk select checkboxes → bulk delete or bulk stage change
- Export button → downloads CSV of all applications
```

### 3. JOB DETAIL PANEL (slide-in, right side, 480px wide)
```
Header section:
  - Company logo (large, 40px) + company name + role title
  - Stage selector (pill buttons, click to change stage)
  - Priority toggle (Low / Medium / High)
  - Quick action bar: [🤖 AI Tools] [📝 Notes] [📄 Docs] [🔗 Share]

Tabs:
  1. Overview
     - Job description (expandable, max 200px collapsed)
     - Key details: location, salary, remote, applied date, URL
     - Tags (editable, add/remove inline)
     - Contacts section (add recruiter / hiring manager contacts)

  2. AI Tools
     - 4 large buttons: Resume Tailor | Interview Coach | ATS Scan | Outreach Writer
     - Below each button: last-used timestamp if used before
     - Match score display with breakdown (skills, seniority, keywords)

  3. Activity
     - Reverse chronological list of all events
     - Events have icons: 🔄 stage change, 🤖 AI used, 📝 note, 📧 email detected
     - "Add manual note" text area at top

  4. Interview rounds
     - List of all interview rounds (add/edit/delete)
     - Each round: date, round type (phone/tech/behavioral/final), interviewer names, notes textarea, questions practiced (from coach)
     - Upcoming interviews show countdown timer

  5. Documents
     - Resume version linked to this job (view / upload new)
     - Cover letter text (editable inline)
     - Additional files list
```

### 4. ADD JOB MODAL
```
Two modes:

Mode A — URL Import:
  - Large URL input with paste button
  - "Import job" button → triggers AI parsing
  - Loading state: skeleton + "Analyzing job listing…" text
  - On success: form auto-fills, user reviews, clicks "Save"

Mode B — Manual Entry:
  - Company name (text)
  - Role title (text)
  - Stage (select, default: saved)
  - Location (text) + Remote toggle
  - Salary range (two number inputs: min / max)
  - Job URL (text, optional)
  - Job description (large textarea)
  - Tags (multi-input, comma-separated)
  - Notes (textarea)

Tab toggle between modes at top of modal.
```

### 5. ANALYTICS VIEW
```
Top row — 6 metric cards (matching stage colors):
  [14 Saved] [8 Applied] [5 Screening] [3 Interview] [1 Offer] [4 Closed]

Conversion funnel (visual):
  Applied → Screening: 62%
  Screening → Interview: 60%
  Interview → Offer: 33%
  (Use SVG bars, not a chart library)

Charts row:
  - Applications over time (line, last 30 days) — built with SVG
  - Response rate by day-of-week (bar) — SVG
  - Top companies by stage (horizontal bar) — SVG

Momentum digest section:
  - White card with AI icon
  - "Generate this week's digest" button
  - Streams AI summary inline (no modal)
  - Shows: pace assessment, stale applications to follow up, recommended focus

Insights list:
  - Auto-generated insights: "You haven't heard back from 3 companies in 10+ days"
  - "Your Tuesday applications have the highest response rate"
  - "Companies with 50–200 employees respond to you faster"
```

### 6. SETTINGS VIEW
```
Sections:
  1. Profile — name, email (display only)
  2. API Configuration — Anthropic API key input (masked, show/hide toggle) + "Test connection" button
  3. Base Resume — large textarea to paste your master resume text
  4. Preferences — default stage on add, reminder lead time, weekly digest on/off
  5. Data — Export all data (JSON), Import from JSON, Clear all data
  6. Keyboard shortcuts reference table
```

### 7. COMMAND PALETTE (⌘K / Ctrl+K)
```
Full-screen overlay with centered search box
Search across: job titles, companies, tags, notes
Quick actions: "Add new job", "Go to analytics", "Open settings", "Export data"
Recent: last 5 viewed jobs
```

---

## AI FEATURES — DETAILED IMPLEMENTATION

### API call helper
```javascript
async function callClaude(systemPrompt, userMessage, onStream, apiKey) {
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': apiKey,
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-6',
      max_tokens: 2000,
      stream: true,
      system: systemPrompt,
      messages: [{ role: 'user', content: userMessage }]
    })
  });

  // Handle SSE streaming
  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let fullText = '';
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    const chunk = decoder.decode(value);
    const lines = chunk.split('\n').filter(l => l.startsWith('data: '));
    for (const line of lines) {
      try {
        const data = JSON.parse(line.slice(6));
        if (data.type === 'content_block_delta' && data.delta?.text) {
          fullText += data.delta.text;
          onStream(fullText);
        }
      } catch {}
    }
  }
  return fullText;
}
```

### Feature 1: URL Job Parser
```
SYSTEM PROMPT:
"You are a job listing parser. Extract structured data from the provided job listing text.
Return ONLY valid JSON, no markdown, no explanation.
Schema: {
  companyName: string,
  roleName: string,
  location: string,
  remote: boolean,
  salaryMin: number|null,
  salaryMax: number|null,
  jdSummary: string (2 sentences max),
  tags: string[] (tech stack, role type, seniority — max 6),
  jdText: string (full cleaned JD text)
}"

USER MESSAGE: "Parse this job listing:\n\n[pasted or scraped text]"

Implementation:
1. User pastes URL → fetch via cors-anywhere proxy OR prompt user to paste JD text directly (more reliable)
2. Show loading spinner with "Reading job listing…" text
3. Parse JSON response → populate form fields
4. Show preview → user confirms → save
```

### Feature 2: Resume Tailor
```
SYSTEM PROMPT:
"You are an expert resume writer with 15 years of experience helping candidates get interviews at top companies.
Your task: rewrite the user's experience bullet points to maximize alignment with the target job description.

Rules:
- Keep the same number of bullet points, rewrite content only
- Use strong action verbs and metrics where the original has them
- Mirror keywords from the JD naturally (do not keyword-stuff)
- Preserve factual accuracy — never invent numbers or companies
- Return ONLY valid JSON: {
    bullets: [{ original: string, rewritten: string, keywordsAdded: string[] }],
    atsScoreBefore: number (0-100),
    atsScoreAfter: number (0-100),
    topMissingKeywords: string[],
    summaryNote: string (one sentence on the biggest improvement made)
  }"

USER MESSAGE:
"Job Description:\n[jdText]\n\nMy Resume:\n[baseResume]\n\nPlease tailor my resume for this role."

UI: Side-by-side diff view. Each bullet pair shows:
- Left: original (with strikethrough on changed words in red)
- Right: rewritten (with added keywords highlighted in blue)
- "Accept" / "Reject" toggle per bullet
- ATS score meter: before vs after (animated bar)
- "Copy tailored resume" button at bottom
```

### Feature 3: ATS Scanner
```
SYSTEM PROMPT:
"You are an ATS (Applicant Tracking System) expert. Analyze the resume against the job description.
Return ONLY valid JSON: {
  overallScore: number (0-100),
  categories: {
    keywords: { score: number, found: string[], missing: string[] },
    skills: { score: number, found: string[], missing: string[] },
    experience: { score: number, notes: string },
    formatting: { score: number, issues: string[] }
  },
  topActions: string[] (3 specific things to fix, most impactful first),
  verdict: string (one sentence plain English assessment)
}"

UI:
- Circular score dial (SVG, animates from 0 to score on load)
- 4 category bars with scores
- "Missing keywords" chips (click to copy keyword)
- "Top 3 actions" list with checkboxes (user can mark done)
- Re-scan button after changes
```

### Feature 4: Interview Coach
```
SYSTEM PROMPT:
"You are a senior career coach and ex-FAANG interviewer. Generate a comprehensive interview prep guide
for the candidate. Return ONLY valid JSON: {
  questions: [
    {
      type: 'behavioral'|'technical'|'culture'|'situational',
      question: string,
      whyAsked: string (one sentence),
      starScaffold: {
        situation: string (prompt/hint for this candidate based on their resume),
        task: string,
        action: string,
        result: string
      },
      redFlags: string (common mistakes to avoid)
    }
  ] (15 questions total: 6 behavioral, 5 technical/role-specific, 2 culture, 2 situational),
  companyInsights: string (3 bullet points about company culture/values based on JD),
  suggestedQuestions: string[] (5 smart questions to ask the interviewer)
}"

UI:
- Question list with type badges (color-coded)
- Click question → expands with full STAR scaffold
- "Practice mode": hides answer, shows only question, user types draft answer → submits for AI feedback
- Progress tracker: checkboxes to mark questions practiced
- "Generate feedback on my answer" button (follow-up Claude call)
```

### Feature 5: Outreach Writer
```
SYSTEM PROMPT:
"You are a networking and outreach expert. Write a personalized, genuine outreach message.
Tone: professional but human, not salesy. Length: 3–4 short paragraphs max.
The message must feel handcrafted, not templated.
Return ONLY valid JSON: {
  coldEmail: { subject: string, body: string },
  linkedInNote: string (max 300 chars),
  followUpEmail: { subject: string, body: string }
}"

USER MESSAGE:
"Company: [companyName]
Role I'm targeting: [roleName]
My background (relevant parts): [excerpt from baseResume]
Notes about this company/why I'm interested: [user's notes on the job]"

UI:
- Three tabs: Cold Email | LinkedIn Note | Follow-up Email
- Copy button per template
- "Regenerate" button with optional tone adjustment (More formal / More casual / More concise)
- Character counter on LinkedIn note
```

### Feature 6: Match Score
```
SYSTEM PROMPT:
"Compare this resume against this job description. Score fit from 0–100.
Return ONLY valid JSON: {
  overallScore: number,
  breakdown: {
    skillsMatch: { score: number, matched: string[], missing: string[] },
    experienceMatch: { score: number, note: string },
    seniorityMatch: { score: number, note: string },
    keywordDensity: { score: number, note: string }
  },
  verdict: 'strong'|'moderate'|'stretch'|'reach',
  oneLineVerdict: string
}"

Trigger: automatically when a job is added with a JD + user has a base resume
Display: compact score chip on job card + full breakdown in AI Tools tab
```

### Feature 7: Weekly Momentum Digest
```
SYSTEM PROMPT:
"You are a career coach reviewing a job seeker's weekly activity. Be direct, specific, and encouraging.
No generic advice. Only insights derived from this specific data. Max 5 sentences total.
End with exactly ONE recommended action for this week."

USER MESSAGE:
"Here is my job search activity for the past 7 days:
- Applications added: [N]
- Stages advanced: [list]
- Applications with no response for 7+ days: [list of company names]
- Interviews scheduled: [N]
- Offers: [N]
- Most active day: [day]

Generate my weekly momentum digest."

UI: Streams inline in the analytics view, not a modal. Looks like a note card with AI avatar.
```

---

## DRAG AND DROP IMPLEMENTATION

Use native HTML5 drag-and-drop (no library needed):

```javascript
// On each card
draggable={true}
onDragStart={(e) => {
  e.dataTransfer.setData('jobId', job.id);
  e.dataTransfer.setData('fromStage', job.stage);
}}

// On each column
onDragOver={(e) => e.preventDefault()}
onDrop={(e) => {
  const jobId = e.dataTransfer.getData('jobId');
  const fromStage = e.dataTransfer.getData('fromStage');
  if (fromStage !== targetStage) {
    updateJobStage(jobId, targetStage);
    logActivity(jobId, 'stage_changed', { from: fromStage, to: targetStage });
    showToast(`Moved to ${targetStage}`, 'success');
  }
}}
```

Add visual drop-target highlighting: `onDragEnter` adds a highlighted border class, `onDragLeave` removes it.

---

## REMINDER SYSTEM

Client-side only (no server). On app load:
```javascript
// Check all applications on mount
applications.forEach(job => {
  if (job.stage === 'applied' && job.appliedAt) {
    const daysSinceApplied = differenceInDays(new Date(), new Date(job.appliedAt));
    if (daysSinceApplied >= 5 && !job.followUpSent) {
      showToast(`Follow up with ${job.companyName}? Applied ${daysSinceApplied} days ago`, 'warning', {
        action: { label: 'Write follow-up', onClick: () => openOutreachWriter(job.id) }
      });
    }
  }
});
```

Store `lastReminderShown` timestamp per job in localStorage to avoid repeat toasts.

---

## SAMPLE DATA (pre-loaded on first launch)

Pre-populate with 8 sample applications across all stages so the app looks alive on first open:
```javascript
const SAMPLE_JOBS = [
  { companyName: 'Stripe', roleName: 'Senior Frontend Engineer', stage: 'interview', matchScore: 87, location: 'Remote', tags: ['React', 'TypeScript', 'Senior'] },
  { companyName: 'Linear', roleName: 'Product Designer', stage: 'screening', matchScore: 72, location: 'San Francisco, CA', tags: ['Figma', 'Design Systems'] },
  { companyName: 'Vercel', roleName: 'Developer Advocate', stage: 'applied', matchScore: 91, location: 'Remote', tags: ['DevRel', 'Next.js', 'Content'] },
  { companyName: 'Anthropic', roleName: 'Software Engineer', stage: 'saved', matchScore: 95, location: 'San Francisco, CA', tags: ['AI', 'Python', 'Senior'] },
  { companyName: 'Figma', roleName: 'Engineering Manager', stage: 'offer', matchScore: 78, location: 'New York, NY', tags: ['Management', 'Frontend'] },
  { companyName: 'Notion', roleName: 'Full Stack Engineer', stage: 'applied', matchScore: 83, location: 'Remote', tags: ['React', 'Node.js', 'PostgreSQL'] },
  { companyName: 'GitHub', roleName: 'Senior SWE', stage: 'closed', matchScore: 65, location: 'Remote', tags: ['Go', 'Distributed Systems'] },
  { companyName: 'Arc', roleName: 'iOS Engineer', stage: 'screening', matchScore: 55, location: 'San Francisco, CA', tags: ['Swift', 'iOS', 'Cocoa'] }
];
```

---

## KEYBOARD SHORTCUTS

| Shortcut | Action |
|---|---|
| ⌘K / Ctrl+K | Open command palette |
| N | New job (when no input focused) |
| Escape | Close any open modal/panel |
| ⌘1 | Switch to Kanban view |
| ⌘2 | Switch to List view |
| ⌘3 | Switch to Analytics view |
| / | Focus search bar |
| Tab | Navigate between job cards |

---

## TOAST NOTIFICATION SYSTEM

```javascript
// Toast types: 'success' | 'error' | 'warning' | 'info' | 'ai'
// Each toast: { id, message, type, duration (ms), action? }
// Max 3 visible at once (older ones push up and fade)
// Position: bottom-right, 16px from edges
// AI toasts have a sparkle icon and blue accent
```

---

## COMPANY LOGO FETCHING

Use Clearbit Logo API (free, no auth):
```javascript
const logoUrl = `https://logo.clearbit.com/${domain}`;
// Fallback: two-letter initials avatar using company name
// Cache in localStorage: { 'stripe.com': 'https://...' }
```

---

## ERROR STATES

Every AI feature must handle:
1. **No API key set** → friendly prompt to add key in Settings, link directly to Settings
2. **Network error** → retry button + "Check your connection" message
3. **API rate limit** → "Claude is a bit busy, try in 30 seconds" + auto-retry countdown
4. **Malformed JSON response** → fallback to plain text display + "Something went wrong parsing the result"
5. **Empty job description** → "Add a job description first to use this feature" inline hint

---

## EMPTY STATES

Each empty state must be helpful, not just decorative:
- **No jobs at all** → "Start your job search. Add your first job listing above, or paste a job URL to import it automatically."
- **Kanban column empty** → Small dashed border + "Drag jobs here"
- **No AI results yet** → "Run the AI tools from the AI Tools tab in any job's detail panel"
- **No analytics data** → "Track at least 5 applications to see your conversion funnel"

---

## PERFORMANCE REQUIREMENTS

- Initial render: < 100ms (no heavy computations on mount)
- localStorage reads: debounce writes at 300ms, never block render
- AI streaming: update React state every ~50ms during streaming (not every chunk — batch with RAF)
- Drag-and-drop: zero layout shifts during drag, use `will-change: transform` on draggable cards
- Search/filter: computed with `useMemo`, never recomputes unless deps change

---

## ACCESSIBILITY

- All modals trap focus and return focus on close
- Kanban stage changes also keyboard-accessible via context menu
- Color is never the only indicator (icons + labels always accompany color)
- ARIA labels on all icon-only buttons
- `prefers-reduced-motion`: disable all transitions/animations

---

## IMPLEMENTATION INSTRUCTIONS FOR CLAUDE

1. **Start with the full HTML shell** — include all CDN imports in `<head>`, define all CSS custom properties in `<style>`, mount React app in `<div id="root">`

2. **Build state and data layer first** — AppContext, initial state, localStorage persistence, sample data injection

3. **Build the Sidebar** — navigation items, active state highlighting, mini stats (total jobs, interviews this week, offers)

4. **Build the Kanban view** — all 6 columns with cards, drag-and-drop, job card component, column headers

5. **Build the AddJobModal** — URL import mode and manual mode, both fully functional

6. **Build the JobDetailPanel** — slide-in panel with all 5 tabs

7. **Build each AI feature modal** — streaming output, loading states, copy buttons

8. **Build List view and Analytics view** — table, filters, SVG charts

9. **Build Settings view** — API key input, base resume textarea, preferences

10. **Build Command Palette** — search + quick actions

11. **Add all toast notifications** — trigger from every meaningful user action

12. **Add sample data** — inject on first load (check localStorage for `trackflow_initialized` flag)

13. **Final polish** — keyboard shortcuts, empty states, error handling, responsive layout

---

## QUALITY BAR

Before considering this complete, verify:
- [ ] App loads with sample data and looks beautiful immediately
- [ ] Can add a job manually in < 30 seconds
- [ ] Can paste job text and AI fills all fields correctly
- [ ] Dragging a card between columns works smoothly and persists on refresh
- [ ] All 5 AI features produce useful, streamed output
- [ ] Closing and reopening the browser restores all data exactly
- [ ] No console errors in normal usage
- [ ] Works on mobile (cards stack vertically, modals are full-screen)
- [ ] Dark theme is consistent — no white flash, no mismatched backgrounds
- [ ] Every empty state is helpful

---

*Build this as a single `index.html` file. Every feature listed must be implemented — not stubbed or mocked. The user should be able to open this file in any browser and immediately use it as a production-quality job tracker.*