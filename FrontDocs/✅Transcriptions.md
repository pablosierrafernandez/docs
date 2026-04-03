# Transcriptions

**Routes:** `/transcriptions` · `/details/:id`
**Components:** `TranscriptionsView.vue` · `DetailsView.vue`
**Access:** Authenticated. Sandbox must be ready.

This section covers two related views: the **List** page where you manage all transcription jobs, and the **Detail** page where you inspect the results of a single transcription in depth.

---

## Part 1 — List View (`/transcriptions`)

### Header

| Element | Behavior |
|---|---|
| **Export** | Downloads the currently filtered transcriptions as a CSV file. Disabled if no data exists. |
| **Filters** | Toggles the advanced filter panel. Shows a blue badge with the count of active filters. |
| **Refresh** | Forces a fresh fetch from the API. Icon spins while loading. |
| **New Transcription** | Navigates to the `/transcribe` form to start a new job. |

All header buttons are disabled when the Sandbox is not ready.

---

### Group tabs

A horizontal tab bar below the header provides instant one-click group filtering:

| Tab | Color | Icon |
|---|---|---|
| **All** | Dark/slate | Bar chart |
| **No group** | Grey | Dashed circle |
| **Pending Review** | Amber | Clock |
| **Under Review** | Blue | Eye |
| **Archived** | Purple | Archive |

Each tab shows a count badge reflecting how many items are in that group. Clicking a tab applies the group filter immediately. The active tab is highlighted with a colored background.

---

### Filters panel

Clicking **Filters** slides open an animated panel below the header. Submit with **Apply Filters** or dismiss with **Clear**.

#### Search field
Free-text input that matches against transcription ID, name, or configuration tag. After typing 2 or more characters, an autocomplete dropdown shows up to 5 matching suggestions (IDs, names, or config tags). Clicking a suggestion applies it immediately.

#### Additional filters (3-column grid on desktop)

| Filter | Type | Options |
|---|---|---|
| **Status** | Dropdown | All statuses / Completed / In Progress / Failed |
| **Configuration** | ConfigurationSelector | Searchable dropdown of all configurations |
| **Participant** | EntitySelector | List of participants present in loaded transcriptions |
| **Evaluator** | EntitySelector | List of evaluators present in loaded transcriptions |

#### Critical Failures Only
A red-highlighted toggle row. When enabled, only transcriptions where a **Strict** criterion failed are shown.

#### Date Range
Two date inputs (**Date from** / **Date to**) with quick-preset buttons:

| Preset | Range |
|---|---|
| Today | Current day only |
| This week | Last 7 days |
| This month | First day of current month to today |
| This quarter | First day of current quarter to today |

#### Footer
- Result count ("X result found")
- **Clear** button — resets all filter fields and reloads
- **Apply Filters** button — applies all current values and closes the panel

> All active filters are synced to the URL query string. Sharing or bookmarking the URL preserves the exact filter state.

---

### Controls bar

Visible only when data is loaded and not in a loading state.

- **"Showing X of Y transcriptions"** — current page item count vs. total matching count
- **Sort by Date** — toggles ascending/descending order; active sort shows an arrow icon
- **Grid / List toggle** — switches between card grid and table layout (persisted in URL)
- **Footer:** "Updated [time ago]" — relative time since the last API fetch

---

### Bulk selection

Hovering over any card or row reveals a **checkbox** that replaces the audio icon. Selecting at least one item shows a blue **bulk action bar** (animated slide-down):

| Element | Behavior |
|---|---|
| Count | "N selected" |
| **Change group** | Opens ChangeGroupModal for all selected items |
| **Delete** | Opens the bulk delete confirmation modal |
| **✕** | Clears all selections |

The table header also contains a **select all** checkbox to select every item on the current page at once.

---

### Grid view

Cards arranged in a responsive grid (1 → 2 → 3 columns). Each card shows:

| Element | Detail |
|---|---|
| Icon / Checkbox | Blue audio-lines icon; transitions to a selectable checkbox on hover or when selected. Selected card gets a blue ring. |
| Name | Bold title. Shows *"No name"* in italic grey if the transcription has no label. |
| Transcription ID | Monospace, truncated. |
| Configuration tag | Clickable — opens the configuration detail in a new browser tab. |
| Quality score | `X.X / 100` — green when ≥ 50, red when < 50. A red octagon icon is shown if a Critical Failure occurred. |
| Participant tag | Plain text (shown only if a participant was linked). |
| Evaluator tag | Clickable — opens the evaluator detail in a new browser tab (shown only if an evaluator was linked). |
| Status badge | **Completed** (green) · **In Progress** (blue, spinning icon) · **Failed** (red) |
| Group badge | Color-coded badge with an inline pencil icon to change the group. Shows *"No group"* placeholder if unassigned. |
| Date | Creation date (or failure date for failed transcriptions). |

Clicking anywhere on the card (except interactive elements) navigates to the detail view.

---

### List view (table)

#### Desktop (≥lg breakpoint)

Full-width table with **resizable columns** — drag the thin blue handle at the right edge of any column header to resize it.

| Column | Contents |
|---|---|
| **Name** | Select-all checkbox (header) + audio icon/checkbox + name + ID (small monospace) |
| **Configuration** | Config tag — clickable, opens in new tab |
| **Status** | Status badge |
| **Group** | Group icon + label + pencil edit button |
| **Quality** | Score (green/red) + Critical Failure icon if applicable |
| **Participant / Evaluator** | Stacked: participant row on top, evaluator row below (evaluator is clickable) |
| **Creation** | Date/time — column header is clickable to sort asc/desc |

Clicking any row navigates to the detail view.

#### Mobile (<lg breakpoint)

Simplified stacked rows showing name, configuration tag, ID, status badge, group, score, participant/evaluator, and date.

---

### Pagination

Shown when the total number of results spans more than one page.

- **"Page X of Y"** label
- **← Prev** / **Next →** buttons (disabled at first/last page)
- Page number buttons with `...` ellipsis for large page counts
- Current page is highlighted in blue

---

### States

| State | What is shown |
|---|---|
| Loading | 6 animated skeleton pulse cards |
| Error | Red banner with error message and **Retry** button |
| No matches (filters active) | *"No matches found"* with a **Clear filters** button |
| Empty (no transcriptions yet) | *"No transcriptions yet"* with a **Refresh** button |
| Sandbox not ready | Yellow `SandboxSetupWarning` banner |

---

### Export CSV

Triggered by the **Export** header button. Exports the currently filtered transcription list as a `.csv` file named `transcriptions_YYYY-MM-DD.csv`.

CSV columns: `ID`, `Name`, `Configuration`, `Status`, `Group`, `Duration (s)`, `Date`.

If no transcriptions match the active filters, a warning notification is shown instead of downloading.

---

### Notifications

| Event | Type | Message |
|---|---|---|
| Group updated (single) | Success | *"The transcription's group has been updated successfully."* |
| Group update error | Error | *"Could not update the group."* |
| No data to export | Warning | *"There are no transcriptions matching the current filters."* |
| Deleted successfully | Success | *"The transcription has been deleted."* |
| Delete error | Error | *"Could not delete the transcription."* |
| Bulk delete summary | Success / Warning | *"N of M transcriptions deleted."* |
| Bulk group summary | Success / Warning | *"N of M groups updated."* |

---

### Modals

#### Single delete modal
Triggered from a card or row's delete action.

| Element | Behavior |
|---|---|
| Title | "Delete Transcription" |
| Body | Shows the transcription name or ID + *"This action cannot be undone."* warning. |
| **Delete** button | Spinner while deleting. Reloads the list on success. |
| **Cancel** / click outside | Closes without deleting. |

#### Bulk delete modal
Triggered from the bulk action bar.

| Element | Behavior |
|---|---|
| Title | "Delete multiple transcriptions" |
| Body | *"Are you sure you want to delete N transcriptions?"* + irreversible warning. |
| **Delete N** button | Spinner while processing. Shows summary notification on completion. |
| **Cancel** / click outside | Closes without deleting. |

#### Group change modal (single & bulk)
Uses the shared **ChangeGroupModal** component. Presents group options (Pending Review / Under Review / Archived / No group). Applies to one or all selected transcriptions depending on context.

---

## Part 2 — Detail View (`/details/:id`)

### Search form

A query form at the top of the page for loading any transcription by ID.

| State | UI |
|---|---|
| No result loaded | Full card with label "Transcription Detail", UUID text input, and **Query** button. |
| Result loaded | Compact bar: "Current transcription" label + truncated ID + **Change** button to reset and query a different ID. |

Navigating directly to `/details/:id` (e.g., from the list view or a notification link) automatically loads the transcription and shows the compact bar.

---

### Results header

Displayed once a transcription is successfully loaded.

| Element | Behavior |
|---|---|
| Transcription name | Primary title. Shows the ID if no name was given. |
| **Copy ID** | Copies the transcription ID to the clipboard. Button turns green briefly showing "Copied!". |
| **Export PDF** | Generates and downloads a formatted quality report. Only visible when a Quality Audit exists. |
| **Delete** (trash icon) | Opens the delete confirmation modal. |

---

### Metadata grid

A two-column information grid below the header.

| Field | Value |
|---|---|
| **Status** | Badge: Completed (green) · In Progress (blue) · Failed (red) |
| **Group** | Color-coded group badge + **Change Group** pencil button to open the group modal |
| **Duration** | Processing duration in seconds |
| **Creation Date** | Full date and time |
| **Language** | Language detected in the audio |
| **Speakers** | Number of distinct speakers identified |

Below the grid, the **Configuration tag** is shown with a link icon — clicking it opens the configuration detail in a new tab.

---

### Applied Modules tree

A visual tree diagram showing which optional modules were applied to this transcription:

```
[Configuration]  ← Always present
       |
  ┌────┴────┐
[Evaluator] [Participant]
```

- Each node that was applied is clickable and opens its detail page in a new tab.
- Nodes that were not applied show a *"Not applied"* placeholder.
- This mirrors the visual layout of the Transcribe submission form.

---

### Processing error panel *(FAILED status only)*

Shown when `status === 'FAILED'`. A red-tinted card with a collapsible **"View technical details"** section that exposes the raw error information returned by the API.

---

### AI Summary card

An orange-gradient collapsible card labeled **"AI Summary"**.

- Content is rendered as formatted **markdown** — supports headings, bullet lists, bold text, and links.
- A **Copy summary** button copies the raw markdown text to the clipboard.
- The card is hidden if no summary was generated.

---

### Quality Audit card

An indigo-gradient collapsible card labeled **"Evaluation"**. Only shown when an evaluator was applied and the audit has completed.

#### Score gauge

An SVG circle gauge displaying the score (0–100) in the center. The ring color reflects the score:

| Score range | Color |
|---|---|
| ≥ 80 | Green |
| 50–79 | Amber |
| < 50 | Red |

#### KPI chips

Three summary chips displayed alongside the gauge:

| Chip | Color | Meaning |
|---|---|---|
| **Passed** | Green | Number of criteria scored as a pass |
| **Critical** | Red | Number of Strict criteria that failed (triggers automatic overall failure) |
| **Failures** | Amber | Number of weighted (non-strict) criteria that failed |

#### Context info

- **Evaluator** — name tag with a link to the evaluator detail (new tab)
- **Audited Participant** — participant name (if linked)
- **Evaluation Date** — when the audit was generated
- **Language** — language of the evaluation feedback

#### Executive Summary

AI-generated feedback text summarizing the overall performance of the call.

#### Criteria Breakdown

A list of all evaluation criteria with their individual results.

**Focus Mode** toggle — when enabled, the list filters to show only criteria that **failed**. Useful for quickly identifying improvement areas.

Each criterion is an expandable accordion row:

| Element | Description |
|---|---|
| `#N` | Sequential number |
| Name | Criterion label |
| Type badge | `Yes/No` (Boolean) · `Scale` · `Critical` (Strict) |
| Pass / Fail icon | Checkmark or X |
| Weight % | Contribution to total score (0% for Strict criteria) |

**Expanded body:**

| Section | Content |
|---|---|
| **Reasoning** | The AI's explanation of why this criterion passed or failed. |
| **Quote** | The exact phrase from the transcript that the AI cited as evidence. Includes a **"Search in transcription"** button — clicking it scrolls the page down to the matching conversation segment and highlights it. |
| **Improvement Opportunity** | Specific coaching advice for the evaluated participant. |

---

### Extracted Fields card

An orange/amber collapsible card labeled **"Extracted Fields"** with a count of fields in the header. Only shown if the configuration had extraction enabled and fields were populated.

Each field row shows:
- **Field name** (with a Code icon)
- **Value** — rendered according to its type:
  - **Array** → grey pill tags
  - **Boolean** (`true`/`false`) → green *"Yes"* badge or red *"No"* badge
  - **Text** → plain bold string
- **Copy** button (per field) — shows a checkmark for 2 seconds after copying.

---

### Conversation card

An emerald-gradient card labeled **"Conversation"**. Only shown if the transcription produced conversation segments.

#### Conversation search
A search input in the card header (top right). Typing filters the visible segments and highlights matching terms in yellow within the text. A clear (✕) button appears when a query is active. If no segments match, a *"No results"* state is shown with a **Clear search** button.

#### Conversation segments

Each segment in the transcript is displayed as a bubble:

| Element | Detail |
|---|---|
| Speaker avatar | A colored circle with a mic icon. Each unique speaker gets a consistent color. |
| Speaker label | Speaker name (if diarized) or *"Speaker"* placeholder. |
| Timestamp | `start_time – end_time` in `MM:SS` format, shown in a grey pill. |
| Jump button | A play-circle icon that appears on hover. Clicking it **selects** that segment (ring highlight) and scrolls it into view. |
| Text | The transcript text for this turn. Matching search terms are highlighted in yellow. |

#### Linking from Quality Audit

When you click **"Search in transcription"** on a criterion's quote, the page automatically:
1. Scrolls to the conversation segment that contains the quoted text.
2. Highlights the segment with a ring.
3. Shows a floating **"Back"** button (arrow up, bottom-right corner) that scrolls back to the Quality Audit card when clicked. The button disappears once you scroll back up.

---

### Empty content state

Shown when the transcription has `status === COMPLETED` but produced no AI summary, no extracted fields, and no conversation segments. Displays a placeholder icon with the message *"Transcription completed, but no content"*.

---

### PDF Export report

Triggered by the **Export PDF** button (labeled *"Evaluation"* in the UI). Generates and downloads a structured quality report PDF.

**Report sections:**

| Section | Content |
|---|---|
| Header | "QUALITY REPORT" title + export date |
| Metadata | Process ID · File name · Participant (labeled "Operator") · Evaluator · Evaluation date |
| Score summary | Score value · Passed / Critical / Failed counts |
| Feedback | Executive summary text |
| Criteria Breakdown | Table with columns: Criteria · Status · Points · Feedback · Quote · Advice |

The PDF filename follows the pattern `Report_[name]_[date].pdf`.

---

### Modals

#### Delete modal

| Element | Behavior |
|---|---|
| Title | "Confirm Deletion" |
| Body | *"Are you sure you want to delete transcription **[ID]**?"* |
| Warning | *"This action is irreversible."* in a red alert box. |
| **Confirm Deletion** button | Spinner while deleting (`detailsStore.isDeleting`). On success, navigates back to the list view. |
| **Cancel** / click outside | Closes without deleting (disabled while deletion is in progress). |

#### Group change modal

Same **ChangeGroupModal** component as the list view. Options: Pending Review / Under Review / Archived / No group.

#### API Key warning modal

Shown when the Sandbox is not configured and the user attempts an action that requires it.

| Element | Value |
|---|---|
| Title | "API Key Required" |
| Subtitle | "A Sandbox key is needed" |
| Body | Explanation that a Sandbox API Key must be configured to query transcriptions. |
| **Cancel** | Closes the modal. |
| **Go to API Keys** | Navigates to the API Keys configuration page. |

---

## Notes

- Transcription results are read-only — they cannot be edited after processing. Only the **name** (label), **group**, and **deletion** are mutable from these views.
- The **Quality score** shown in the list is a cached summary value. For the full audit breakdown, open the detail view.
- Deleting a transcription from either the list or the detail view is **permanent** and removes all associated results, summaries, and audit data.
- The **Export PDF** button is only available when a Quality Audit is present. Transcriptions without an evaluator will not have this option.
- The **"Back"** floating button in the detail view only appears after clicking "Search in transcription" from the audit panel — it disappears once you scroll back up past the audit section.
- Filters applied in the list view are URL-synced. You can share a filtered view by copying the browser URL.
