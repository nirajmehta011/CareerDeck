# 💼 CareerDeck — AI-Powered Job Application Tracker

**CareerDeck** is a world-class, single-file, privacy-first, AI-powered job application tracker designed to help job seekers organize their funnel, optimize their resumes, and accelerate their career journey.

Everything is packed into a single `index.html` file that runs entirely in your browser. There is **no installation, no server, and no backend database setup** required.

👉 **Try it Live**: [https://career-deck.vercel.app/](https://career-deck.vercel.app/)

---

## 📺 Video Walkthrough

Watch the walkthrough video to see **CareerDeck**'s key features in action, including the AI Job Card Parser, Resume Tailor, AI Document Tools, and STAR Interview Coach:

[Walkthrough Video](walkthrough.mp4)

---
## 🌐 Live Demo:

You can access the live application deployed on Vercel here: 👉 [Live Carrer Deck Website](https://career-deck.vercel.app/)

## 🚀 Key Features

### 1. Visual Application Pipeline
* **Kanban Board**: Drag-and-drop job application cards across recruitment stages (Saved, Applied, Screening, Interview, Offer, Closed) with visual column metrics.
* **List View**: A high-density table view with search, multi-tag filters, remote toggles, and bulk action exports (CSV).

### 2. Multi-LLM AI Suite (Settings & Config)
* Supports interchangeable API backends: **Google Gemini**, **OpenAI**, **Anthropic**, **OpenRouter**, **Groq**, and **DeepSeek**.
* Easily fetch lists of models directly from the provider endpoints inside the app.
* Configure custom feature preferences (default landing view, stages, follow-up notifications, weekly digest intervals).

### 3. Integrated AI Document Tools
* **ATS Scanner**: Analyzes your master resume against any job description, identifying keyword gaps and formatting issues using a custom circular SVG score gauge.
* **Resume Tailor**: Restructures and rewrites resume bullet points using the recruiter-preferred **XYZ formula** (*"Accomplished [X] as measured by [Y], by doing [Z]"*). View side-by-side diffs and accept/reject suggestions.
* **Cover Letter Generator**: Autogenerates natural, human-sounding cover letters. Excludes physical address information for privacy and auto-injects JavaScript-calculated current dates (preventing placeholder errors).
* **Document Export**: Visually previews tailored resumes and compiles exports directly to standard Word (`.docx`) and formatted PDF (`jsPDF`) formats using premium typography.
* **Resume Document Parser**: Upload any PDF, Word document, or text file to extract plain text directly in the browser (uses Mammoth.js & PDF.js loaded on-demand via CDN).

### 4. Interactive Interview & Communication Tools
* **Interview Coach**: Generates 15 role-specific practice questions (behavioral, technical, situational) with structured workspace inputs tailored to the **STAR framework** (Situation, Task, Action, Result).
* **Outreach Writer**: Drafts customized outreach templates for cold emails, LinkedIn connection messages, and recruiter follow-ups.

### 5. Advanced Analytics & insights
* Custom SVG-rendered Funnel charts, 30-day activity line graphs, and day-of-week response bars.
* **Weekly Momentum Digest**: Streams real-time dashboard analytics and checklists to keep candidates engaged and structured.

### 6. Command Palette (⌘K)
* Access global searches, quick navigations, command triggers, and recent files instantly with a spotlight-style overlay.

---

## 🔒 Privacy & Architecture Invariants

1. **Client-Side Only**: All state transitions and logic run in the user's browser.
2. **Local Storage Persistence**: All application data, resumes, documents, settings, and logs are persisted locally using `localStorage`. No external database is used.
3. **API Key Security**: Your API keys are stored only in your browser's local state. Requests are made directly from your machine to the AI providers.
4. **Zero-Build Delivery**: Ready to open out of the box. No compilers, webpack, or npm builds needed.

---

## 🛠️ Built With

* **Core**: React 18 (CDN UMD build) + Babel Standalone (in-browser transpiler)
* **Styling**: Tailwind CSS (CDN) + Custom Vanilla CSS tokens
* **Icons**: Lucide Icons (React wrapper)
* **Libraries (Loaded On-Demand)**:
  * [docx.js](https://docx.js.org/) (Word document generation)
  * [jsPDF](https://github.com/parallax/jsPDF) (PDF page formatting and rendering)
  * [PDF.js](https://mozilla.github.io/pdf.js/) (PDF file parsing)
  * [Mammoth.js](https://github.com/mweiss/mammoth.js) (Word document parsing)

---

## 🏃 How to Run

Because the application is serverless, you have two simple options to launch it:

### Option 1: Direct File Open
Simply double-click the [index.html](file:///Users/nirajmehta/Documents/AI%20Projects/Job%20Tracker%20AI%20Application/index.html) file, or drag it into any web browser.

### Option 2: Local Server (Recommended for API testing)
To prevent potential CORS blockages from some API endpoints, run a local web server in the directory:

```bash
# Using Python
python3 -m http.server 8000

# Using Node.js (npx)
npx serve
```
Open `http://localhost:8000` (or `http://localhost:3000`) in your browser.
