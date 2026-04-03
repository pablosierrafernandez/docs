# Participants

**Route:** `/participants`
**Component:** `ParticipantsView.vue`
**Access:** Authenticated. Sandbox must be ready.

A **Participant** is a named entity — a person, agent, or team member — that can be linked to transcription jobs. Participants allow the platform to attribute quality metrics to specific individuals and track their performance over time.

> Unlike Configurations and Evaluators, there are no separate create or detail routes. All creation and editing happens inside a **side drawer** that slides in from the right without leaving the page.

---

## List View (`/participants`)

### Header controls

| Element | Behavior |
|---|---|
| **Reload** button | Forces a fresh fetch from the API. Shows a spinning icon while loading. Disabled when Sandbox is not ready. |
| **New Participant** button | Opens the Create Drawer. Disabled when Sandbox is not ready. |

### Filter & sort bar

Visible only when there is at least one participant or an active search query.

| Element | Behavior |
|---|---|
| Search input | Filters by name (tag) or participant ID. Results update in real time. |
| **Metadata filters** (sliders icon) | Opens/closes the collapsible metadata filter panel (see below). |
| Sort by **Name** | Toggles ascending/descending alphabetical sort on tag. |
| Sort by **Date** | Toggles ascending/descending sort on last updated date. |
| **Grid** / **List** toggle | Switches between card grid and table layout. |
| Result counter | Shows `X of Y` participants. |
| Last updated | Shows relative time since the last API fetch (e.g. "Updated 5 minutes ago"). |

### Metadata filter panel

Collapsible panel below the filter bar. Dynamically generates one `<select>` dropdown per metadata key found across all participants currently in view.

| Element | Behavior |
|---|---|
| Filter dropdowns | One per unique metadata key (e.g. `role`, `department`). Shows all distinct values for that key as options. Default: `"All"` (no filter). |
| Active filter badge | Number bubble on the sliders icon showing the count of active filters (keys where a non-default value is selected). |
| Filter logic | **AND intersection** — a participant must match every active filter to appear. Combined with search text. |
| **Clear filters** link | Appears in the panel header when at least one filter is active. Resets all metadata dropdowns to `"All"`. |
| Icons per key | Each key's dropdown shows the preset icon for that key (e.g. briefcase for `role`, envelope for `email`). Unknown keys use a hash icon. |

**No results state:** When the current filters + search return zero participants, the empty state shows a button to clear both the search text and all metadata filters at once.

### Avatars

Each participant has a **gradient avatar** showing their initials (first 2 letters of the first word, or first letter of each of the first two words). The gradient color is deterministically assigned from the participant's name — the same name always produces the same color. Six gradient palettes are used (rose/violet/blue, emerald/teal/cyan, amber/orange/red, fuchsia/purple/indigo, sky/blue/indigo, lime/emerald/teal).

On hover or when selected, the avatar fades out and a **checkbox** fades in, allowing selection without a dedicated interaction target.

### Grid view

Cards arranged in a responsive grid (1→2→3 columns). Each card shows:
- Large gradient avatar (top left) with checkbox on hover
- Participant name (tag) — truncated if too long
- Participant ID in monospace
- Metadata pills (first **4** entries shown; "+N more" badge if more exist)
- Last updated date

Clicking a card opens the **Edit Drawer** for that participant.

### List view

Full-width table with columns:

| Column | Contents |
|---|---|
| **Participant** | Small gradient avatar + name (tag) + participant ID in monospace |
| **Metadata** | First **2** metadata entries as pills + "+N" if more |
| **Updated** | Last updated date and time |

Clicking any row opens the **Edit Drawer** for that participant.

### Metadata pills

Each metadata entry is shown as a pill with an icon based on its key (see [Metadata keys reference](#metadata-keys-reference)), the key label in small uppercase, and the value. Custom keys not in the preset list use a generic Hash icon.

If a participant has no metadata, a *"No metadata"* placeholder is shown instead.

### Bulk selection & delete

- Hovering a row/card reveals a **checkbox** in place of the avatar.
- Selecting at least one item shows a blue **bulk action bar** with a count and a red **Delete** button.
- The table header has a **select all** checkbox for the current filtered results.

> To delete a **single participant**, select it via its checkbox and use the bulk Delete button.

### States

| State | What is shown |
|---|---|
| Loading | Animated skeleton cards/rows (no spinner) |
| Empty (no participants) | Empty state with icon, message, and "New Participant" button |
| Error | Red banner with error message and Retry link |
| No search results | "No results found" with a clear-search button |
| Sandbox not ready | Yellow `SandboxSetupWarning` banner |

### Delete modal

Triggered by the **Delete** button in the bulk action bar. Works for both single and multiple selections.

| Element | Behavior |
|---|---|
| Title | "Delete Participant" (single) or "Delete Participants" (bulk) |
| Description | Names the participant (single) or count (bulk). |
| Warning | "This action is irreversible." in a red alert box. |
| **Delete** button | Executes deletion. Shows a success or error notification per result. |
| **Cancel** / click outside | Closes without deleting. |

---

## Drawer — Create & Edit

The drawer slides in from the right edge of the screen. It is **max 400 px wide** and spans the full viewport height. Clicking the backdrop closes it without saving.

The same drawer is used for both creating and editing a participant. The title and save button label differ between modes.

### Drawer header

| Element | Create mode | Edit mode |
|---|---|---|
| Title | "New Participant" | "Edit Participant" |
| Participant ID | Not shown | Shown in monospace below the title, with a copy-to-clipboard button (shows a green checkmark for 2 s after copying) |
| Close button | ✕ — closes drawer | ✕ — closes drawer |

### Identity section

A single required field: **Participant Name** (tag). A live gradient avatar preview sits on the left of the input, updating in real time as the user types — showing the initials and the color that will be assigned to this participant. If the field is submitted empty, an error message appears below: *"Name is required"*.

### Tags section (Metadata)

Structured key-value pairs that describe the participant (e.g. department, role, email). A counter shows `N / 10` to indicate how many fields have been added. Maximum **10 metadata fields**.

#### Preset quick-add chips

9 predefined field types appear as pill buttons at the top of the section. Clicking a chip instantly adds a new row with that key pre-filled. A chip becomes disabled (blue, dimmed) once its key is already present in the form.

| Chip | Key | Example value placeholder |
|---|---|---|
| Email | `email` | `example@company.com` |
| Role | `role` | `Supervisor, Agent...` |
| Phone | `phone` | `+1 600 000 000` |
| Department | `department` | `Sales, Support...` |
| Region | `region` | `New York, London...` |
| Language | `language` | `EN, ES, FR...` |
| Timezone | `timezone` | `UTC+1` |
| Skills | `skills` | `SQL, Customer Service...` |
| Employee ID | `employee_id` | `EMP-001` |

#### Dynamic rows

Each metadata row contains:
- **Key input** (left third) — the field name. Shows a context-aware icon based on the key. When a preset chip was used, the key is pre-filled.
- **Value input** (right two-thirds) — the field value. Placeholder adapts to the key if it matches a preset.
- **Trash button** — removes the row.
- Pressing **Enter** in the value input adds a new blank row.

**Validation errors** shown inline below the row:
- *"Complete both fields"* — if only key or only value is filled.
- *"Duplicate key"* — if the same key appears more than once.

Rows with both key and value empty are silently ignored on submit.

#### Add custom field button

A dashed **"+ Add custom field"** button appears below the rows and adds a blank row (key and value both empty). Hidden when the 10-field limit is reached.

### Drawer footer (sticky)

Two buttons fixed at the bottom:

| Button | Behavior |
|---|---|
| **Cancel** | Closes the drawer without saving. |
| **Create Participant** / **Save Changes** | Submits the form. Shows a spinner while the API call is in progress. Disabled during submission. |

On success the drawer closes automatically. On failure the drawer stays open (the store handles error notifications).

---

## Metadata Keys Reference

| Key | Icon | Example value |
|---|---|---|
| `email` | Mail | `agent@company.com` |
| `role` | Briefcase | `Supervisor` |
| `phone` | Phone | `+1 600 000 000` |
| `department` | Building | `Customer Support` |
| `region` | Map Pin | `New York` |
| `language` | Languages | `EN` |
| `timezone` | Globe | `UTC+1` |
| `skills` | Star | `SQL, Customer Service` |
| `employee_id` | Fingerprint | `EMP-001` |
| *(any other key)* | Hash | — |

---

## Notes

- The participant **name (tag)** can be changed after creation via the Edit Drawer.
- Metadata is stored as a flat key-value map. Keys must be unique within a participant. Rows with both fields empty are ignored on save.
- Participants are linked to transcription jobs at submission time. Deleting a participant does not affect already-processed transcriptions.
- There is no dedicated detail view for a participant — all information is managed through the drawer on the list page.
