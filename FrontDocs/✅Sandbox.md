# The Heify Sandbox

## What is the Sandbox?

The Sandbox is a personal, isolated workspace within the Heify platform. Each authenticated user gets their own Sandbox instance — a self-contained environment where they can build transcription pipelines, manage quality evaluation criteria, upload audio, and explore performance analytics without affecting any other user or any production system.

The Sandbox is fully browser-based. It connects to the Heify API using an API key that is stored in the browser and sent automatically with every request. Until that key is configured and validated, most features remain locked.

---

## Getting started

### 1. Authentication

Access to the Sandbox requires an account. Authentication is **OTP-based**: the user enters their email address, receives a one-time code, and enters it to log in. No password is required.

After successful login the user is redirected to `/sandbox/dashboard`.

### 2. Automatic setup

When the Sandbox layout loads for the first time, it automatically attempts to **provision a reserved API key** (`API_KEY_SANDBOX_RESERVED`). This key is created on the user's behalf, validated by making a test call to the Heify API, and stored in the browser's localStorage.

If provisioning succeeds, the Sandbox is immediately ready to use. A brief *"Setting up the sandbox..."* notice may appear during this process. On success, a *"Sandbox ready"* notification is shown.

If automatic provisioning fails, the user is directed to the **API Keys & Sandbox** page to configure the connection manually.

### 3. Sandbox ready

The Sandbox is considered **ready** when two conditions are both true:
- An API key is stored in the browser.
- That key has been validated against the Heify API.

When the Sandbox is not ready, a **SandboxSetupWarning** banner is displayed at the top of affected pages. This banner includes a *Configure* button that navigates directly to the API Keys & Sandbox page. Features that depend on the API — transcription, configurations, evaluators, analytics — are disabled until the Sandbox is ready.

---

## Navigation

The Sandbox is organized into six main areas, accessible from the collapsible sidebar on desktop and a bottom navigation bar on mobile.

| Section | Route | Purpose |
|---|---|---|
| **Dashboard** | `/sandbox/dashboard` | Overview of the pipeline, summary counts, and setup status |
| **Configurations** | `/sandbox/configurations` | Create and manage transcription configurations (the foundation of every pipeline) |
| **Evaluators** | `/sandbox/evaluators` | Define AI quality criteria to score transcriptions |
| **Participants** | `/sandbox/participants` | Register the agents or speakers who appear in the audio |
| **Transcribe** | `/sandbox/create-transcription` | Upload audio files and submit them for transcription |
| **Transcriptions** | `/sandbox/transcriptions` | View, filter, and inspect all processed transcriptions |
| **Analysis** | `/sandbox/analytics/*` | AI-powered analytics dashboards (by Configuration, Evaluator, or Participant) |

### Sidebar extras

The desktop sidebar footer contains:
- **AI Assistant** — opens the Heify NotebookLM assistant in a new tab (compact icon card)
- **Docs** — external documentation (compact icon card)
- **Support** — help channel (compact icon card)
- **Dark / Light mode toggle** — pill-style toggle with sun/moon icons flanking the track. Persists preference to `localStorage` under the key `heify-theme`. On first load, falls back to `prefers-color-scheme`. Also available as an icon button on the login page.

The user menu at the bottom of the sidebar shows the user's role (**Admin** or **Member**), remaining transcription minutes, and analytics quota counts.

---

## Core workflow

A typical Heify Sandbox workflow follows this sequence:

```
1. Create a Configuration
       ↓
2. (Optional) Create Evaluators and Participants
       ↓
3. Upload audio via Transcribe
       ↓
4. Review results in Transcriptions
       ↓
5. Explore trends in Analytics
```

**Configurations** are the foundation. Every transcription must be associated with a configuration, which defines what fields to extract from the audio and how to process it.

**Evaluators** add a quality scoring layer. An evaluator defines a set of criteria — each with a name, description, weight, and strictness level — that are applied to transcriptions to produce a score and flag any critical failures.

**Participants** identify the agents or speakers in the recordings, enabling participant-level performance tracking across the analytics dashboards.

**Transcribe** is where audio enters the system. Files are uploaded (or submitted via URL), associated with a configuration, and optionally linked to evaluators and a participant. Once processed, the result appears in Transcriptions.

**Analytics** provides three reporting dashboards: one per Configuration (aggregate field extraction metrics), one per Evaluator (quality and pass-rate performance), and one per Participant (individual agent performance across campaigns). Each dashboard generation consumes one unit of the corresponding analytics quota.

---

## Quota

The Sandbox operates under two types of quota:

| Quota type | What it controls |
|---|---|
| **Transcription minutes** | Total audio duration that can be processed |
| **Analytics quota** | Number of analysis reports that can be generated per module (configurations, evaluators, participants) |

Remaining quota is shown in the sidebar user menu. When an analytics quota is exhausted, the Generate Analysis button is disabled on the corresponding dashboard.

---

## API keys and team access

All API keys are managed from the **API Keys & Sandbox** page (`/sandbox/api-keys`). The auto-provisioned reserved key appears pinned at the top of the list. Users can also create additional keys with custom names (up to 10 keys total).

Team members can be invited from the same page. Each member gets their own access to the Sandbox under the same account. The Admin role can manage keys and team membership; Members have access to all pipeline features but cannot administer keys or team settings.

---

## Roles

| Role | Capabilities |
|---|---|
| **Admin** | Full access: pipeline, analytics, API key management, team management |
| **Member** | Pipeline and analytics access; cannot manage API keys or team |

---

## Page-by-page documentation

For detailed UI and behavior reference, see the individual documentation files:

- `SandboxAuthentication.md` — Login and OTP flow
- `APIKeysSandbox&Team.md` — API key management and team access
- `Configurations.md` — Configuration CRUD and detail view
- `Evaluators.md` — Evaluator CRUD and criteria management
- `Participants.md` — Participant management
- `Transcribe.md` — Audio upload and transcription submission
- `Transcriptions.md` — Transcription list and detail view
- `ConfigurationAnalysis.md` — Configuration-level analytics dashboard
- `EvaluatorAnalysis.md` — Evaluator performance dashboard
- `ParticipantAnalysis.md` — Participant performance dashboard
