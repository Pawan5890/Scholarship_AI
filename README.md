# Scholarship_AI

# Scholarship AI — आपका व्यक्तिगत स्कॉलरशिप सहायक

A bilingual (English / हिंदी) web app that helps Indian students discover scholarships they're eligible for, understand the application process, and prepare their documents — powered by a **deterministic eligibility engine** and an **AI assistant that only explains verified records** (it never decides eligibility on its own).

Published at: **https://scholarshipai.base44.app**

---

## 🎯 Mission & Design Principles

1. **Deterministic eligibility, AI explanation.** A code-based engine classifies every scholarship against the student's profile into one of six strict categories. The LLM only *explains* those results — it never re-decides eligibility, invents amounts, or promises awards.
2. **Trust comes first.** Only human-verified records from official sources (NSP, state portals) are shown as "confirmed." Unverified data is clearly badged.
3. **Bilingual & accessible.** Full English + Hindi UI, voice input (Web Speech API), and spoken responses (TTS, user-triggered only).
4. **Gen-Z oriented design.** Vibrant indigo/violet/pink palette, gradient hero, motivational banners, clean cards.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND (React + Vite)                │
│  src/pages/*  →  src/components/*  →  src/lib/*              │
│   Home, Search, Profile, ScholarshipDetail, Chat, Reviewer    │
│                                                               │
│   Tailwind CSS (token-based design system in index.css)       │
│   shadcn/ui components, lucide-react icons                     │
│   react-router-dom (routing), @tanstack/react-query (cache)   │
│                                                               │
│   LanguageContext (EN/HI) · translations.js (all UI strings)  │
└──────────────────────────┬────────────────────────────────────┘
                           │  base44 SDK (@/api/base44Client)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                     BASE44 PLATFORM BACKEND                    │
│                                                               │
│  ENTITIES (base44/entities/*.jsonc)                           │
│   ├─ Scholarship        — the scholarship record               │
│   ├─ EligibilityRule   — per-scholarship matching rules       │
│   ├─ ScholarshipSource — provenance / source tracking         │
│   ├─ StudentProfile    — the student's saved profile          │
│   └─ UploadedDocument  — private file uploads (metadata)     │
│                                                               │
│  BACKEND FUNCTION (base44/functions/chatWithAssistant)        │
│   └─ entry.ts — LLM orchestration (DeepSeek + platform LLM)   │
│                                                               │
│  CORE INTEGRATIONS                                            │
│   ├─ InvokeLLM (DeepSeek primary, platform fallback)          │
│   ├─ UploadPrivateFile / CreateFileSignedUrl (docs)          │
│   └─ Web Speech API (browser-native, client-side)             │
│                                                               │
│  AUTH — platform-managed (email/password, Google, OTP, reset) │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
.
├── index.html                      # Title, meta tags, fonts
├── src/
│   ├── App.jsx                     # Router + auth gate + providers
│   ├── main.jsx                    # Entry point
│   ├── index.css                   # Design tokens (indigo/pink theme)
│   ├── tailwind.config.js          # Token → Tailwind class mapping
│   │
│   ├── pages/
│   │   ├── Home.jsx                # Hero + embedded ChatPanel + features + trust
│   │   ├── Search.jsx              # Filter + search scholarships, eligibility badges
│   │   ├── Profile.jsx            # Student profile form (drives matching)
│   │   ├── ScholarshipDetail.jsx   # Full scholarship view + eligibility explanation
│   │   ├── Chat.jsx                # Standalone chat page (alt entry)
│   │   └── Reviewer.jsx            # Admin: add/edit/delete scholarships + rules
│   │
│   ├── components/
│   │   ├── Layout.jsx              # Top nav + mobile bottom nav + <Outlet/>
│   │   ├── ChatPanel.jsx           # Bilingual chat UI + voice + markdown
│   │   ├── MarkdownMessage.jsx     # Rich markdown + link preview cards
│   │   ├── VoiceInput.jsx          # Speech-to-text (EN/HI)
│   │   ├── SpeakButton.jsx         # Text-to-speech (user-triggered)
│   │   ├── ScholarshipCard.jsx     # Search result card
│   │   ├── ScholarshipForm.jsx     # Reviewer: scholarship + rules editor
│   │   ├── ProfileForm.jsx         # Student profile form
│   │   ├── EligibilityBadge.jsx    # Classification badge
│   │   ├── LanguageSelector.jsx    # EN/HI toggle
│   │   ├── ProtectedRoute.jsx      # Auth gate
│   │   └── ui/*                    # shadcn/ui primitives
│   │
│   ├── lib/
│   │   ├── eligibility.js         # ⭐ Deterministic eligibility engine
│   │   ├── constants.js           # Education levels, states, categories, docs
│   │   ├── translations.js        # All EN/HI UI strings
│   │   ├── LanguageContext.jsx     # Language state (persisted)
│   │   ├── AuthContext.jsx         # Auth state + redirects
│   │   ├── query-client.js         # React Query client
│   │   └── utils.js                # cn() helper
│   │
│   └── api/
│       └── base44Client.js         # Pre-initialized Base44 SDK
│
└── base44/
    ├── entities/
    │   ├── Scholarship.jsonc
    │   ├── EligibilityRule.jsonc
    │   ├── ScholarshipSource.jsonc
    │   ├── StudentProfile.jsonc
    │   └── UploadedDocument.jsonc
    │
    └── functions/
        └── chatWithAssistant/
            └── entry.ts            # LLM chat backend (DeepSeek + fallback)
```

---

## 🗄️ Data Model (Entities)

### `Scholarship`
The core record — name, provider, award amount + frequency, education levels, courses, states, academic year, application open/deadline dates, deadline confidence, required documents, official URL, source URLs, verification status, caveats, and bilingual summaries (EN/HI).

### `EligibilityRule`
A per-scholarship rule linked by `scholarship_id`. Each rule has a `field` (education_level, course, state, marks_percentage, marks_cgpa, family_income, category, gender, disability, etc.), an `operator` (eq, ne, gte, lte, gt, lt, in, not_in, exists), and a value (string/number/array). Rules carry bilingual descriptions used in explanations.

### `StudentProfile`
The student's saved profile — education level, course, academic year, marks (value + type + CGPA scale), institution, state, domicile state, family income, category, gender, disability, previous scholarships. This is what the eligibility engine matches against.

### `ScholarshipSource`
Provenance tracking — source URL, source name, whether it's official, last checked date, which field was verified, and notes. Supports the "trust comes first" principle.

### `UploadedDocument`
Private file upload metadata — file URI, name, type, size, extracted text, upload purpose, and verified flag.

---

## ⚙️ The Eligibility Engine (`src/lib/eligibility.js`)

This is the heart of the app — **pure deterministic code, no LLM reasoning**.

### Classification (6 strict categories)

| Category | Meaning | Color |
|---|---|---|
| `confirmed` | All rules pass, source verified, deadline verified & open | emerald |
| `potential` | All rules pass but deadline unverified/unknown | amber |
| `info_required` | One or more required profile fields are missing | sky |
| `ineligible` | At least one rule failed | rose |
| `closed` | Application deadline has passed | slate |
| `unverified` | All rules pass but the scholarship data isn't verified | violet |

### Evaluation flow
```
evaluateEligibility(scholarship, rules, profile)
  1. Filter rules where rule.scholarship_id === scholarship.id
  2. For each rule → checkRule(rule, profile)
       - getProfileField() maps rule.field → profile value
       - apply operator (eq/ne/gte/lte/gt/lt/in/not_in/exists)
       - returns { status: "pass" | "fail" | "missing" }
  3. Bucket results into passed / failed / missing
  4. Compute deadline status (open/closed/unknown)
  5. Apply classification priority:
       closed > ineligible > info_required > unverified > potential > confirmed
  6. Return { classification, deadlineStatus, verification, missing, failed, passed }
```

**Key safety detail:** CGPA and percentage are *not* interchangeable — `marks_percentage` only reads the value when `marks_type === "percentage"`, and vice versa. This prevents silent mismatches.

`explainResult(result, lang)` then builds a plain-English or Hindi sentence explaining *why* the result was classified that way — shown on the detail page and fed to the AI assistant.

---

## 🤖 The AI Assistant (`base44/functions/chatWithAssistant/entry.ts`)

A backend function that orchestrates the LLM, grounded strictly in stored records.

### How it works
1. Receives `{ message, profile, language, scholarships, history }` from `ChatPanel`.
2. The `scholarships` array is **pre-computed by the deterministic engine** on the client (classification, award, deadline status, missing/failed fields, official URL, documents, summary) — the LLM never re-evaluates eligibility.
3. Builds a structured context string listing up to 12 scholarships with their classifications and key facts.
4. **Two paths:**
   - **URL detected in message** → uses `InvokeLLM` with `add_context_from_internet: true` (Gemini Flash) to look up the pasted scholarship/form link and guide the student step-by-step.
   - **Standard path** → calls **DeepSeek** (`deepseek-chat`, temp 0.3, 700 max tokens) with the system prompt + context + last 6 messages. Falls back to platform `InvokeLLM` if DeepSeek is unavailable.
5. Returns `{ reply }`.

### System prompt rules (enforced)
- Lead with the answer — **don't interrogate** the student with a list of profile questions (the info is already in context).
- Only explain records provided in context — **never invent** names, amounts, deadlines, documents, or links.
- For every recommended scholarship, include: name + provider, award amount + frequency, deadline + status, official URL, "why apply" benefit, and "what to do next."
- Never promise an award or claim an application was submitted.
- Respond in the student's language (EN/HI).

### Secrets
- `DEEPSEEK_API_KEY` — stored as an app secret, accessed via `secrets.get()` server-side only.

---

## 🎨 Design System

Token-based, defined in `src/index.css` and mapped in `tailwind.config.js`:

- **Primary:** indigo `hsl(243 75% 59%)`
- **Accent:** pink/magenta `hsl(322 81% 55%)`
- **Gradients:** indigo → violet → purple for hero/chat headers; amber, emerald, rose, sky, slate for classification colors.
- Fonts: system sans-serif stack (heading/body/display tokens).
- Responsive: mobile-first with a bottom nav (4 items) on small screens, top nav on desktop.

---

## 🔐 Authentication

Platform-managed — the app does **not** implement its own auth backend. Boilerplate pages exist and are wired in `App.jsx`:
- `/login`, `/register`, `/forgot-password`, `/reset-password`
- Email/password, Google OAuth, OTP verification, password reset
- `ProtectedRoute` gates all app pages; unauthenticated users redirect to `/login`.

---

## 🌐 Internationalization

- `LanguageContext` persists the choice (EN/HI) in localStorage, defaults to English.
- `translations.js` holds every UI string in both languages via `makeT(lang)`.
- Scholarship records store **both** `summary_en` and `summary_hi`; the UI picks based on `lang`.
- Eligibility explanations are bilingual via `explainResult(result, lang)`.

---

## 📱 Pages & User Flows

| Page | Route | Purpose |
|---|---|---|
| **Home** | `/` | Hero, embedded AI chat, motivational banner, feature highlights, trust section |
| **Profile** | `/profile` | Student fills their profile → drives all eligibility matching |
| **Search** | `/search` | Search + filter scholarships by education level / state / year; shows confirmed vs. potential counts; each card links to detail |
| **Scholarship Detail** | `/scholarship/:id` | Full record: eligibility badge + explanation, award, deadline, documents, missing/unmet conditions, caveats, official link, sources, speak button |
| **Chat** | `/chat` | Standalone full-height chat (alt to home-embedded chat) |
| **Reviewer** | `/reviewer` | Admin CRUD: add/edit/delete scholarships + their eligibility rules |

---

## 🛠️ Local Development

This is a Base44 project — run it with the Base44 CLI, not raw `npm run dev`.

```bash
# Prerequisites
npm install
npm install -g base44@latest
# Install Deno (local backend runtime)

# One-time per machine / clone
base44 login
base44 link

# Run frontend + local backend together
base44 dev
# → opens http://localhost:5173
```

**Notes:**
- `base44 dev` runs the frontend for you — never run `npm run dev` separately.
- The app must be published at least once for the UI to load under `base44 dev`.
- Entity data is **in-memory only** locally (wiped on restart); Core integrations and OAuth forward to your deployed app.
- Frontend-only against live backend: `base44 dev --remote` (writes go to production data).

---

## 🚀 Publishing

Publish from the Base44 dashboard (top-right **Publish** button), not `base44 deploy`:

```bash
base44 dashboard open
```

---

## 🔌 Integrations & Secrets

| Integration | Use | Where |
|---|---|---|
| **DeepSeek API** | Primary LLM for chat | Server-side via `DEEPSEEK_API_KEY` secret |
| **Platform InvokeLLM** | Fallback LLM + web-search path (Gemini Flash) | Server-side via `base44.asServiceRole` |
| **UploadPrivateFile** | Private document uploads | Client-side → `file_uri` stored in `UploadedDocument` |
| **CreateFileSignedUrl** | Time-limited download URLs for private files | Client-side |
| **Web Speech API** | Voice input (STT) + spoken responses (TTS) | Browser-native, client-side only |

---

## 📦 Key Dependencies

React 18 · Vite · Tailwind CSS · shadcn/ui · lucide-react · react-router-dom · @tanstack/react-query · react-markdown · framer-motion · @base44/sdk

---

## ⚠️ Known Limitations

- Web Speech API Hindi recognition reliability varies across mobile browsers.
- Seed dataset is limited (~12 verified scholarships).
- LLM latency can occasionally cause chat timeouts (mitigated by 20s AbortSignal timeout + fallback).
- GitHub 2-way sync requires the Builder plan or higher.

---

## 📚 Docs & Support

- Base44 docs: https://docs.base44.com
- Local development: https://docs.base44.com/developers/backend/overview/local-dev/local-development-overview
- GitHub integration: https://docs.base44.com/developers/app-code/local-development/github
- Support: https://app.base44.com/support
