# Configurations

**Routes:** `/configurations` · `/configurations/new` · `/configurations/:id`
**Components:** `ConfigurationsListView.vue` · `CreateConfigurationView.vue` · `ConfigurationDetailView.vue`
**Access:** Authenticated. Sandbox must be ready.

A **Configuration** defines how the system processes an audio file: whether to generate an AI summary, which language to use, what custom vocabulary to apply, which data fields to extract from the transcript, and where to send webhook notifications on completion. Configurations are reusable templates — you apply one when submitting a transcription.

---

## 1 — List View (`/configurations`)

### Header controls

| Element | Behavior |
|---|---|
| **Reload** button | Forces a fresh fetch from the API. Shows a spinning icon while loading. Disabled when Sandbox is not ready. |
| **New Configuration** button | Navigates to the creation form. Disabled when Sandbox is not ready. |

### Filter & sort bar

Visible only when there is at least one configuration or an active search query.

| Element | Behavior |
|---|---|
| Search input | Filters by tag, configuration ID, or vocabulary words. Results update in real time. |
| Sort by **Date** | Toggles ascending/descending sort on creation date. Active sort shows an up/down arrow icon. |
| **Grid** / **List** toggle | Switches between card grid and table layout. Preference is local to the session. |
| Result counter | Shows `X of Y configurations`. When a filter is active, a blue `filtered` chip appears. |
| Last updated | Shows relative time since the last API fetch (e.g. "Updated 2 minutes ago"). |

### Capability badges

Each configuration card or row displays badges indicating which features are active:

| Badge | Color | Meaning |
|---|---|---|
| **AI Summary** | Blue | Summary generation is enabled. |
| **Webhooks** | Emerald green | At least one webhook URL is configured. |
| **N field(s)** | Violet | Number of extraction fields defined. |
| **N word(s)** | Orange | Number of custom vocabulary words. |

If none are active, the card shows *"No special capabilities"*.

### View modes

**Grid view** — Cards arranged in a responsive grid (1→2→3 columns). Each card shows: tag, configuration ID, capability badges, vocabulary preview (first 2 words + "+N more"), and creation date.

**List view** — Full-width table with columns: Configuration (tag + ID), Capabilities (badges), Creation date. Clicking any row opens the detail view.

### Bulk selection & delete

- Hovering a row/card reveals a **checkbox** that replaces the tag icon.
- Selecting at least one item shows a blue **bulk action bar** at the top of the list with a count and a red **Delete selection** button.
- The table header has a **select all** checkbox for the current filtered results.
- Clicking **Delete selection** opens the delete modal in bulk mode.

### States

| State | What is shown |
|---|---|
| Loading | Animated skeleton cards/rows (no spinner) |
| Empty (no configurations) | Empty state with icon, message, and "Create First Configuration" button |
| Error | Red error banner with message and Retry button |
| No search results | "No configurations found" with a clear-search button |
| Sandbox not ready | Yellow `SandboxSetupWarning` banner with a link to API Keys |

### Delete modal

Triggered by the trash icon on a card/row (single) or the bulk action bar (multiple).

| Element | Behavior |
|---|---|
| Title | "Delete configuration" |
| Description | Names the configuration (single) or count (bulk). |
| Warning | "This action cannot be undone." in a red alert box. |
| **Delete** button | Shows spinner while processing. Disabled during deletion. |
| **Cancel** / click outside | Closes without deleting. |

After successful deletion, a success notification is shown and the item is removed from the list.

---

## 2 — Create View (`/configurations/new`)

Two-column layout on large screens: **form** (left) and **Quick Actions + Tips** panel (right, sticky).

### Main Parameters section

#### Tag *(required)*
Free-text identifier for the configuration (e.g. *"Customer support"*, *"Sales calls"*). This is the display name used across the platform. Validated on submit — cannot be empty.

#### AI Summary toggle
Toggle switch that enables automatic summarization of transcripts.

- **OFF** (default): no summary generated.
- **ON**: summary is generated; the **Summary Language** dropdown becomes enabled.

#### Summary Language *(active only when summary is ON)*
Searchable dropdown. Selects the output language for the summary. Default: *"Default"* (matches the detected audio language). Supports 60+ languages.

#### Analytics Language
Searchable dropdown. Selects the language for Executive and Qualitative Analysis reports. Default: *"Default"* (inferred from the majority audio language). Always enabled regardless of the summary toggle.

#### Custom Summary Instructions *(visible only when summary is ON)*
Optional textarea (max 300 characters) with specific directives for the AI summarizer.

**Preset templates** — 7 quick-fill buttons appear above the textarea:

| Preset | What it generates |
|---|---|
| Selection interviews | Key candidate info: experience, strengths, weaknesses, hire/no-hire recommendation |
| Meetings & minutes | Structured minutes: attendees, topics, agreements, assigned tasks, next steps |
| Customer follow-up | Client status, issues, satisfaction level, commitments, pending follow-up actions |
| Sales | Prospect needs, objections, value proposition, next commercial step, close probability |
| Team meetings | Topics, decisions, blockers, task assignments, next meeting date |
| Training sessions | Topics covered, key concepts, attendee questions, areas needing reinforcement |
| Custom | Blank — write your own instructions |

Selecting a preset fills the textarea with a ready-to-edit prompt. The character counter turns red above 300 chars.

#### Custom Vocabulary
Tag-style input. Type a word and press **Enter** to add it. Each word appears as a pill tag with a delete button. Use this for technical jargon, brand names, or acronyms that the transcription engine should recognize.

#### Webhook Notifications *(optional)*
Two URL inputs (both optional):
- **Success URL** (green icon): called when transcription completes successfully.
- **Error URL** (red icon): called when transcription fails.

### Extraction Fields section

Defines structured data the AI will extract from each transcript. Supports up to **20 fields**.

Each field requires:
- **Field name** *(required)* — machine-readable key (e.g. `customer_name`).
- **Data type** — `String`, `Number`, `Boolean`, `Array`.
- **Description for the AI** *(required, max 1000 chars)* — natural-language instruction telling the AI what to extract. A "Follow our Tips" badge reminds users to write specific, bounded descriptions.

When a field is added (via Quick Actions or manually), it **scrolls into view and pulses** with a blue highlight for 3 seconds to confirm it was added.

State: if no fields are defined, a dashed empty state prompts the user to add the first field.

### Quick Actions panel *(right column)*

8 pre-built extraction field templates — click to instantly add to the form:

| Quick Action | Field name | Type | What it extracts |
|---|---|---|---|
| Sentiment Analysis | `sentiment_analysis` | String | `"POSITIVE"`, `"NEGATIVE"`, or `"NEUTRAL"` |
| Classification | `classification_tag` | String | Assigns a category from a list you define |
| Quality Score | `quality_score` | Number | 0–10 quality rating based on your criteria |
| Next Action | `next_action` | String | Recommended next step from a defined list |
| Feature Extraction | `feature_extraction` | Array | Which features from a defined list are present |
| Extract Specific Data | `specific_data_extraction` | String | Any custom data point you specify |
| Resolution | `first_call_resolution` | Boolean | Whether the customer's issue was resolved |
| Customer Satisfaction | `customer_satisfaction` | Boolean | Whether the customer was satisfied |

> The descriptions pre-filled by Quick Actions contain **placeholders in brackets** (e.g. `[CATEGORY 1]`, `[YOUR CRITERIA]`). Replace them with your actual values before saving.

### Tips panel *(right column)*

Two advisory cards:

- **Use bounded responses** — Define a fixed set of valid values (e.g. `"POSITIVE"`, `"NEGATIVE"`) for consistency and easier downstream analysis.
- **Detailed context** — Provide examples in the field description to guide the AI toward more accurate results.

### Submit button

Sticky at the bottom of the form column. Shows spinner + animated dots while creating. Disabled if Sandbox is not ready.

### Unsaved changes modal

If the user navigates away with a non-empty form (any field touched), a modal appears:

- **"Keep editing"** — stays on the form.
- **"Leave without saving"** — discards all data and navigates away.

### Sandbox not ready

If the Sandbox API key is not configured, a `SandboxSetupWarning` banner appears at the top of the form. Submitting shows a modal warning with a button to go to API Keys.

---

## 3 — Detail & Edit View (`/configurations/:id`)

### Header buttons

| Button | Mode | Behavior |
|---|---|---|
| ← Back to Configurations | Read | Navigates to the list. |
| Cancel | Edit | Discards changes and returns to read mode. |
| View Analytics | Read | Navigates to the Analytics view for this configuration. |
| Reload | Read | Re-fetches the configuration from the API. |
| **Edit** | Read | Enters edit mode (turns header icon and card borders amber). |
| **Clone** (copy icon) | Read | Duplicates the configuration — navigates to create form with all fields pre-filled (see [Clone](#clone)). |
| Delete (trash icon) | Read | Opens the delete confirmation modal. |
| **Save changes** | Edit | Saves the form. Shows spinner while saving. |

In edit mode, a amber badge **"Editing"** appears next to the configuration name and the page icon changes from a gear to a pencil.

### Sandbox not ready banner

If the Sandbox is not configured, an amber warning banner appears with a button to go to API Keys. Edit and Delete actions are blocked.

---

### Read mode — 5 cards

#### General Information
- **Configuration ID** — monospace display with a copy-to-clipboard button (shows checkmark for 2 s after copying).
- **Tag** — display name.
- **Creation date** — formatted with full date and time.

#### AI Summary & Analysis
- **Transcript Summary** — Enabled / Disabled pill (green or grey).
- **Summary Language** — selected output language.
- **Analytics Language** — selected analytics report language.
- **Custom summary instructions** — shown only if defined.

#### Custom Vocabulary
Pills with a `#` prefix, displayed in a tag cloud. If empty, shows an empty state.

#### Extraction Fields
Each field is shown as a card with:
- Field name (large, bold)
- Type badge (color-coded — see table below)
- Sequential number `#N` (top right)
- Description block

#### Webhooks
URLs are **hidden by default** (shown as 60 `•` dots). A **Show URLs / Hide URLs** toggle reveals them. When visible, each URL has a copy-to-clipboard button.
Empty state shown when no webhooks are configured.

---

### Edit mode — 4 editable cards

All cards gain an amber border and header to signal edit mode.

#### AI Summary & Analysis *(editable)*
- **Tag** — read-only in edit mode (labeled "Not editable").
- **Summary toggle** — same switch as in create form.
- **Summary Language** — searchable dropdown (grayed out when summary is OFF).
- **Analytics Language** — searchable dropdown (always enabled).
- **Custom summary instructions** — textarea (max 300 chars), visible only when summary is ON.

#### Custom Vocabulary *(editable)*
- Text input + **Add** button. Also supports **Enter** key.
- Existing words appear as pills with an **×** to remove.

#### Extraction Fields *(editable)*
- Existing fields show a grey **"Existing field"** badge; newly added fields show a violet **"New"** badge.
- The **description** textarea of each field is editable inline.
- The trash icon to remove a field is hidden by default and appears on hover.
- **"Add field"** button opens an inline form (name, type, description). Requires both name and description — validation shows an error if either is empty.
- Maximum **20 fields** total (button disabled at limit).

#### Webhooks *(editable)*
- Two URL inputs (success and error).
- **"Remove webhooks"** button clears both fields at once.

### Sticky save bar *(edit mode)*

Fixed at the bottom of the screen during editing. Contains:
- **Cancel** (full width, secondary) — discards and exits edit mode.
- **Save changes** (2× wider, blue primary) — saves the configuration. Shows spinner while saving. On success, exits edit mode automatically.

### Delete modal *(read mode)*

| Element | Behavior |
|---|---|
| Title | "Confirm Deletion" |
| Description | Names the configuration. |
| Warning | "This action is irreversible." in a red box. |
| **Yes, delete** button | Spinner while deleting. After success, navigates back to the list. |
| **Cancel** | Closes modal. Not disabled while deleting. |

---

## Clone

Triggered by the **Clone** (copy icon) button in the detail view header. Navigates to `CreateConfigurationView.vue` and pre-fills all fields using `history.state.cloneData`.

| Field | Pre-filled value |
|---|---|
| **Tag** | `"{original_tag} (Copy)"` — truncated to 100 chars if needed |
| **AI Summary toggle** | Copied from original |
| **Summary Language** | Copied from original |
| **Analytics Language** | Copied from original |
| **Custom Summary Instructions** | Copied from original. If text is present, `selectedPreset` is forced to `"custom"` so the textarea renders |
| **Custom Vocabulary** | All words copied (new array, reactivity decoupled) |
| **Extraction Fields** | All fields copied with names, types, descriptions. New `_id` values assigned via `Date.now() + Math.random()` |
| **Webhooks** | Copied from original |

The data is transferred entirely in memory via `history.state` (no API call, no draft stored). If the user refreshes the browser, the pre-fill is lost and a blank form is shown. The clone is not created until the user clicks **Create Configuration**.

---

## Extraction Field Types

| Type | Badge color | Use for |
|---|---|---|
| `string` | Blue | Free text, short answers, named categories |
| `number` | Emerald green | Scores, counts, numeric ratings |
| `boolean` | Violet | Yes/no, true/false outcomes |
| `array` | Indigo | Lists of items, multiple selections |
| `date` | Orange | Dates and timestamps |
| `object` | Slate | Nested structured data |

---

## Notes

- The **Tag** field cannot be changed after creation — it is read-only in edit mode. Plan naming conventions before creating configurations at scale.
- **Extraction field descriptions** are the primary signal the AI uses to decide what to extract. The more specific and bounded the description (including example values), the more consistent the results.
- Webhook URLs are sent a POST request by the backend when a transcription finishes. Both success and error URLs are independent and optional.
- A configuration can be used by multiple transcription jobs simultaneously. Deleting a configuration does not affect already-processed transcriptions.
