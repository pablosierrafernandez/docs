# Evaluators

**Routes:** `/evaluators` · `/evaluators/new` · `/evaluators/:id`
**Components:** `EvaluatorsView.vue` · `CreateEvaluatorView.vue` · `EvaluatorDetailView.vue`
**Access:** Authenticated. Sandbox must be ready.

An **Evaluator** is a reusable quality-control template that defines the criteria the AI uses to audit a call transcript. You specify a set of scored criteria, assign weights, and the AI produces a structured pass/fail report for every transcription that uses this evaluator.

---

## 1 — List View (`/evaluators`)

### Header controls

| Element | Behavior |
|---|---|
| **Reload** button | Forces a fresh fetch from the API. Shows a spinning icon while loading. Disabled when Sandbox is not ready. |
| **New Evaluator** button | Navigates to the creation form. Disabled when Sandbox is not ready. |

### Filter & sort bar

Visible only when there is at least one evaluator or an active search query.

| Element | Behavior |
|---|---|
| Search input | Filters by name (tag), description, or evaluator ID. Results update in real time. |
| Sort by **Name** | Toggles ascending/descending alphabetical sort on tag. Active sort shows an up/down arrow icon. |
| Sort by **Date** | Toggles ascending/descending sort on creation date. |
| **Grid** / **List** toggle | Switches between card grid and table layout. |
| Result counter | Shows `X of Y` evaluators. |
| Last updated | Shows relative time since the last API fetch (e.g. "Updated 2 minutes ago"). |

### Badges

Each evaluator card or row displays:

| Badge | Color | Meaning |
|---|---|---|
| Language name | Grey | The feedback language configured for the evaluator. |
| **N Criteria** | Indigo | Total number of evaluation criteria defined. |
| **Critical** | Red | Shown only if at least one Strict criterion exists. |

### View modes

**Grid view** — Cards arranged in a responsive grid (1→2→3 columns). Each card shows: tag, evaluator ID, description, language + criteria badges, and creation date.

**List view** — Full-width table with columns: Evaluator (tag + description), Language, Criteria (count + Critical badge if applicable), Created date. Clicking any row opens the detail view.

### Bulk selection & delete

- Hovering a row/card reveals a **checkbox** that replaces the evaluator icon.
- Selecting at least one item shows a blue **bulk action bar** with a count and a red **Delete selection** button.
- The table header has a **select all** checkbox for the current filtered results.
- Clicking **Delete selection** opens the delete modal in bulk mode.

> Single-item deletion is done from the detail view, not from the list.

### States

| State | What is shown |
|---|---|
| Loading | Animated skeleton cards/rows (no spinner) |
| Empty (no evaluators) | Empty state with icon, message, and "Create Evaluator" button |
| Error | Red error banner with message and Retry button |
| No search results | "No results found" with a clear-search button |
| Sandbox not ready | Yellow `SandboxSetupWarning` banner |

### Delete modal (bulk)

Triggered by the **Delete selection** button in the bulk action bar.

| Element | Behavior |
|---|---|
| Title | "Delete Evaluators" |
| Description | Names the count of selected evaluators. |
| Warning | "This action is irreversible." in a red alert box. |
| **Delete** button | Executes deletion sequentially. Shows a success or error notification per result. |
| **Cancel** / click outside | Closes without deleting. |

After deletion, a success notification is shown and removed items disappear from the list.

---

## 2 — Create View (`/evaluators/new`)

Two-column layout on large screens: **form** (left) and **Suggested Criteria + Quality Tips** panel (right, sticky).

### General Parameters section

#### Evaluator Name *(required)*
Free-text identifier (e.g. *"Outbound Sales Audit 2025"*). Max 100 characters. Validated on submit — cannot be empty. Character counter shown.

#### Feedback Language
Searchable dropdown. Selects the language in which the AI will write the evaluation feedback. Default: *"Default"* (inferred from the audio). Supports 60+ languages.

#### Description *(optional)*
Short textarea (max 250 characters) to describe the evaluator's purpose. Character counter shown.

---

### Evaluation Context section

Provides situational context to the AI so it understands who is being evaluated and under what circumstances.

**6 preset templates** — pill buttons that prefill the context textarea:

| Preset | Context description |
|---|---|
| Call center | Customer support agent on an inbound call following internal protocols |
| Job interview | Candidate being assessed for role suitability and communication clarity |
| Training or coaching | Student or participant in a training session, assessing comprehension and participation |
| Sales meeting | Salesperson on a sales call, assessing needs identification and objection handling |
| Customer follow-up | Account manager in a follow-up meeting, assessing relationship quality and solution proposals |
| Custom | Blank — write your own context |

Selecting a preset fills the textarea. Selecting **Custom** clears it. The textarea only appears after a preset is selected. Max 1000 characters.

---

### Evaluation Criteria section

Defines the behaviors the AI will score on each transcript. Supports up to **10 criteria**.

A counter at the top shows `N / 10 criteria used`.

Each criterion requires:
- **Criteria Name** *(required, max 100 chars)* — human-readable label (e.g. `Corporate Greeting`).
- **Type** — determines how the AI scores this criterion (see [Criteria Types](#criteria-types)).
- **Weight** *(0–100 %)* — contribution to the final score. Strict criteria always have weight 0.
- **AI Instructions** *(required, max 1000 chars)* — natural-language description telling the AI exactly what to look for in the transcript.

A trash icon removes the criterion. The **Add Criteria** button in the section header and a dashed **Add New Criteria** button below the list both add a blank criterion. At the 10-criterion limit, the add buttons are disabled (a tooltip explains the limit).

When a criterion is added from the Suggested Criteria panel, the new card **scrolls into view and pulses** with a violet highlight for ~1.5 s. A success notification is also shown.

If no criteria are defined, a dashed empty state prompts the user to add the first one.

---

### Weight bar & submit *(sticky at bottom of form)*

A sticky panel fixed at the bottom shows the weight status and the submit button.

| Element | Behavior |
|---|---|
| Total Weight display | Shows the current sum of all non-strict criteria weights (e.g. `Total Weight: 85%`). |
| Status chip | **Correct** (green, checkmark) when total = 100% · **Incomplete** (amber, triangle) otherwise. |
| **Balance Weights** button | Appears when total ≠ 100%. Distributes 100% evenly across all non-strict criteria, with any remainder added to the first one. Fires a success notification. |
| **Create Evaluator** button | Blue when weights are valid. **Red + pulsing** when weights are wrong — text changes to "Review Weights". |
| Submit with wrong weights | Bar and button flash red for ~2 seconds. A warning notification is shown. |
| Loading state | Button shows spinner + "Saving..." while the API call is in progress. |

---

### Suggested Criteria panel *(right column)*

8 pre-built criteria — click to instantly add to the form:

| Criteria | Type | Default weight | What it evaluates |
|---|---|---|---|
| Corporate Greeting | Boolean | 10% | Agent presents name, company and welcome |
| Identity Verification | **Strict** | 0% | Two personal data points verified before giving sensitive info |
| Empathy | Scale | 20% | Use of acknowledgement phrases; no condescending tone |
| Active Listening | Scale | 15% | No interruptions; paraphrases; asks clarifying questions |
| Resolution | Boolean | 25% | Concrete solution offered or escalation with defined next step |
| Objection Handling | Scale | 15% | Responds to objections with facts, adapts proposal |
| Sales Close | Boolean | 15% | Explicitly asks for the sale or commitment |
| Formal Farewell | Boolean | 0% | Summarizes actions, offers further help, waits for client to hang up |

Once a criterion has been added, its button shows a checkmark, the name is struck through, and it displays "Already added". It cannot be added twice.

---

### Quality Tips panel *(right column)*

4 collapsible advisory cards (the panel can be collapsed with the chevron button):

| Tip | Guidance |
|---|---|
| **Be specific and observable** | Describe concrete actions the AI can verify. Avoid subjective criteria. Bad: *"Be friendly"* → Good: *"Greet the client by name and thank them for the call"*. |
| **Define what should NOT happen** | Include negative examples in AI instructions to detect violations and reduce false positives. Example: *"Do not interrupt the client"*. |
| **Add context and examples** | Include expected phrases or specific business situations. Example: *"Agent must mention the Premium Plus plan ($29.99/month) and its benefits"*. |
| **One criteria, one action** | Split complex criteria into simple ones. Each should evaluate a single action. Bad: *"Greeted AND verified AND offered"* → Good: three separate criteria. |

---

### Unsaved changes modal

If the user navigates away with a non-empty form, a modal appears:

- **"Keep editing"** — stays on the form.
- **"Leave without saving"** — discards all data and navigates away.

A form is considered dirty if any of the following are non-default: name, description, language, context, or criteria list.

### Sandbox not ready

A `SandboxSetupWarning` banner appears at the top of the form. Submitting shows a modal warning with a button to go to API Keys.

---

## 3 — Detail & Edit View (`/evaluators/:id`)

### Header buttons

| Button | Mode | Behavior |
|---|---|---|
| ← Back to Evaluators | Read | Navigates to the list. |
| ← Cancel | Edit | Discards changes and returns to read mode. |
| View Analytics | Read | Navigates to the Analytics view for this evaluator. |
| Refresh | Read | Re-fetches the evaluator from the API. |
| **Edit** | Read | Enters edit mode (turns header icon and card borders amber). |
| **Clone** (copy icon) | Read | Duplicates the evaluator — navigates to create form with all fields pre-filled (see [Clone](#clone)). |
| Delete (trash icon) | Read | Opens the delete confirmation modal. |
| **Save changes** | Edit | Saves the form. Shows spinner while saving. |

In edit mode, an amber **"Editing"** badge appears next to the evaluator name and the page icon changes from a clipboard to a pencil.

### Sandbox not ready banner

An amber warning banner appears with a button to configure the API Key. Edit and Delete actions are blocked.

---

### Read mode — cards

#### General Information
- **Name (Tag)** — display name.
- **Evaluation Language** — selected feedback language.
- **Description** — shown in italic. Shows *"No description available."* if empty.
- **Evaluation Context** — shown only if defined, in a green-tinted card.
- **Created on** — formatted date and time.

#### Critical Criteria *(shown only if Strict criteria exist)*
Red-bordered card with a Siren icon and a count badge. Lists each Strict criterion with a sequential `#N` number, name, and description.

#### Evaluation Criteria *(weighted — Boolean and Scale)*
Each criterion is shown as a row with:
- Sequential number `#N`
- Name (bold)
- Description
- Type badge (`Yes/No` or `Scale`)
- Weight percentage (right-aligned with a % circle icon)

Empty state shown if no weighted criteria are defined.

---

### Edit mode — cards (amber border)

All cards gain an amber border and header to signal edit mode.

#### General Information *(editable)*
- **Tag** — read-only in edit mode (labeled "Not editable").
- **Evaluation Language** — searchable dropdown.
- **Description** — textarea (max 250 chars).
- **Evaluation Context** — preset pill buttons + textarea (max 1000 chars). Same presets as the create form.

#### Evaluation Criteria *(editable)*
- Same fields as the create form: name, type select, weight input, AI instructions textarea.
- Trash button per criterion (disabled when only 1 criterion remains — minimum of 1 required).
- **Add Criteria** button centered at the bottom of the list.
- Maximum **10 criteria** (button disabled at limit).

### Sticky save bar *(edit mode)*

Fixed at the bottom of the screen during editing. Contains:
- Weight status display (Total Weight %) + status chip + **Balance Weights** button.
- **Cancel** (full width, secondary) — discards and exits edit mode.
- **Save changes** (2× wider, blue primary) — saves the evaluator. Shows spinner while saving. Turns red when weights are not valid.

### Delete modal *(read mode)*

| Element | Behavior |
|---|---|
| Title | "Delete Evaluator" |
| Description | Names the evaluator tag. |
| Warning | "This action is irreversible." in a red box. |
| Error block | Shown below the warning if the delete API call fails. |
| **Delete** button | Spinner while deleting. After success, navigates back to the list. |
| **Cancel** / click outside | Closes modal (disabled while deletion is in progress). |

### API Key warning modal

Shown when the Sandbox is not configured and the user attempts to delete:

- Title: "Missing API Key"
- Message: "You must configure the Sandbox API Key to perform actions."
- **Cancel** | **Configure** (navigates to API Keys)

---

## Clone

Triggered by the **Clone** (copy icon) button in the detail view header. Navigates to `CreateEvaluatorView.vue` and pre-fills all fields using the current evaluator data passed via `history.state.cloneData`.

| Field | Pre-filled value |
|---|---|
| **Name (Tag)** | `"{original_tag} (Copy)"` — truncated to 100 chars if needed |
| **Feedback Language** | Copied from original |
| **Description** | Copied from original |
| **Evaluation Context** | Copied from original |
| **Criteria** | All criteria cloned: name, type (`boolean`, `scale`, `strict`), weight, AI instructions. New temporary IDs assigned to avoid reactive collisions. |

The data is transferred entirely in memory via `history.state` (no API call, no draft stored). If the user refreshes the browser, the pre-fill is lost and a blank form is shown. The clone is not created until the user clicks **Create Evaluator**.

---

## Criteria Types

| Type | How the AI scores | Weight behavior |
|---|---|---|
| **Boolean** | Answers Yes (pass) or No (fail) | Contributes to score via weight % |
| **Scale** | Scores 1–5 based on degree of compliance | Contributes to score via weight % |
| **Strict** | If not met, the **entire evaluation is automatically failed** regardless of other scores | Always 0% — excluded from weight sum |

---

## Suggested Criteria Reference

| Criteria name | Type | Default weight | Evaluates |
|---|---|---|---|
| Corporate Greeting | Boolean | 10% | Full name + company name + welcome at call start |
| Identity Verification | Strict | 0% | Two personal data points verified before sensitive info is shared |
| Empathy | Scale | 20% | Acknowledgement phrases used; no minimizing or condescending tone |
| Active Listening | Scale | 15% | No interruptions; paraphrasing and clarifying questions used |
| Resolution | Boolean | 25% | Concrete solution offered or escalation with timeline defined |
| Objection Handling | Scale | 15% | Objections answered with data; proposal adapted to client needs |
| Sales Close | Boolean | 15% | Explicit close attempt made (e.g. "Shall we proceed?") |
| Formal Farewell | Boolean | 0% | Actions summarized; further help offered; waits for client to hang up |

> The default weights for the pre-built criteria (excluding Strict) sum to **100%**, making them immediately usable as a complete evaluator without manual adjustment.

---

## Notes

- The **Tag** (name) cannot be changed after creation — it is read-only in edit mode. Plan naming conventions before creating evaluators at scale.
- **Strict criteria** act as automatic disqualifiers. A single failed Strict criterion marks the entire evaluation as failed, regardless of how well other criteria were met. Use them only for non-negotiable compliance requirements (e.g. identity verification, legal disclosures).
- **AI Instructions** are the primary signal for evaluation quality. The more specific and bounded the instruction — including what should and should not happen, with example phrases — the more consistent the AI results.
- **Weight balancing**: weights must sum to exactly 100% across all non-strict criteria before the evaluator can be saved. Use the "Balance Weights" button to distribute them evenly when needed.
- An evaluator can be used by multiple transcription jobs simultaneously. Deleting an evaluator does not affect already-processed transcriptions.
