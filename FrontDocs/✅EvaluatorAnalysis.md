# Evaluator Analysis

**Route:** `/analytics/evaluator/:id`
**Component:** `AnalyticsEvaluatorView.vue`
**Access:** Authenticated. Sandbox must be ready. Quota-gated.

The Evaluator Analysis page generates a performance dashboard for a selected evaluator. It aggregates all transcriptions that were processed with that evaluator and organizes the results into KPI summaries, trend charts, a weekly quality heatmap, criteria failure breakdowns, participant rankings, and a filterable transcription detail table.

> Each analysis generation consumes one unit of the account's `analytics_evaluator` quota.

---

## Page header

| Element | Behavior |
|---|---|
| **Evaluator Analysis** title | Always shown. |
| Evaluator name (tag) | Shown in indigo below the title once an evaluator is active. Before selection, the subtitle *"Analyze the performance Evaluator"* is shown instead. |
| **Last updated** | Relative time since the last fetch (e.g. *"Updated 3 minutes ago"*). Only shown when data is loaded. |
| **Download PDF** (FileDown icon) | Generates and downloads a structured evaluator PDF report. Disabled when no data is available. |
| **Refresh** (RefreshCw icon) | Re-fetches the analysis for the current evaluator. Icon spins while loading. |

The Download PDF and Refresh buttons are only visible when analysis data has been loaded.

---

## Evaluator selector

A white card with a form at the top of the content area.

| Element | Behavior |
|---|---|
| **EntitySelector** | Searchable modal listing all available evaluators. Each evaluator in the modal shows: name (tag) + ID in monospace + criteria count badge. Includes a Reload button to refresh the evaluator list. |
| **Generate Analysis** button | Submits the form and triggers the analysis. |

**Generate Analysis button states:**

| State | Appearance |
|---|---|
| Default | Sparkles icon + "Generate Analysis" + arrow that fades in on hover |
| Loading | Spinner + "Generating..." |
| Disabled | Grey; shown when loading, no evaluator is selected, or quota is exhausted |

If submitted without an evaluator selected, a **warning notification** is shown: *"You must select an evaluator to generate the analysis."*

**URL behavior:** Selecting an evaluator and generating updates the URL to `/analytics/evaluator/:evaluatorId`. Loading the page with an ID in the URL auto-fetches the analysis. If cached data for that evaluator already exists in the store, it is restored without consuming quota.

---

## Query limit banner

An amber banner shown at the top of the dashboard when `metadata.query_limit_reached === true`:

> *"Limit reached: N records analyzed. There may be more data not included."*

This indicates that the analysis is based on a subset of the available transcriptions due to API record limits.

---

## KPI Grid

Four metric cards displayed in a 2×2 grid on mobile or a 4-column row on desktop.

| KPI | Value | Color logic | Extra |
|---|---|---|---|
| **Total Calls** | Count of all evaluated transcriptions | — | Tooltip: *"Total calls evaluated in the selected period"* |
| **Average Score** | Score out of 100 with a gradient progress bar | Emerald ≥91 · Green 71–90 · Amber 51–70 · Orange 31–50 · Red <31 | Tooltip explains the metric |
| **Pass Rate** | Percentage of calls that passed the threshold | Status badge: **Excellent** ≥80% · **Good** 60–79% · **Fair** 40–59% · **Low** <40% | Tooltip: *"Percentage of calls that passed the threshold"* |
| **Criticals** | Count of calls that failed a Strict criterion | Red badge if >0. *"No critical incidents"* if 0 | Tooltip: *"Calls that failed critical criteria"* |

### Critical alert banner

If `pending_critical_reviews > 0`, a red-tinted clickable banner appears above or near the KPI grid:

> *"N pending critical calls — Pending review · Click for details"*

Clicking it **scrolls to the Transcription Details table** and pre-filters it to show only pending critical calls.

---

## Dashboard Charts

A multi-panel chart section with two rows.

### Row 1 — Monthly Comparison and Duration Analysis (2 columns)

#### Monthly Comparison

Two side-by-side period cards: **This Month** and **Previous Month**.

Each card shows:
- **Calls** — total count
- **Average Score** — numeric
- **Passed** — pass rate %

Below the two cards, a **delta summary** shows the change between periods:
- *"Improvement of X pts in average score"* (green, upward arrow)
- *"Decrease of X pts in average score"* (red, downward arrow)
- Pass rate change percentage

| State | What is shown |
|---|---|
| One month has no data | Amber warning: *"No data in [month]"* |
| Both months have no data | *"Data in both months is needed to show the comparison"* |

#### Duration Analysis

Three tiles showing call duration tiers, with thresholds dynamically calculated from the **p25 and p75 percentiles** of the dataset:

| Tier | Threshold |
|---|---|
| **Short** | Below p25 |
| **Medium** | Between p25 and p75 |
| **Long** | Above p75 |

Each tile shows: call count + average score for that tier. Tooltip: *"Dynamically calculated thresholds with P25 and P75 percentiles."*

### Row 2 — Timeline Evolution and Score Distribution

#### Timeline Evolution

A dual-axis **Chart.js** line + bar chart:
- **Left Y-axis**: Average Score (0–100) — line series
- **Right Y-axis**: Call Volume — bar series
- Combined hover tooltips show both metrics at once

Empty state: centered icon + *"No data for this period"*

#### Score Distribution

A bar chart grouping all transcriptions into 20-point score ranges (0–20, 20–40, 40–60, 60–80, 80–100). Shows how the scores are distributed across the dataset.

Tooltip: *"Histogram of score distribution in 20-point ranges."*

---

## Weekly Heatmap + Top Failed Criteria

Two components displayed side-by-side in a 2-column grid (stacked on mobile).

### Weekly Heatmap

A day-of-week × hour-of-day grid showing the **average quality score** for each time slot across all evaluated calls.

| Color | Score range |
|---|---|
| Grey | No data |
| Emerald | ≥ 80 |
| Green | 60–79 |
| Amber | 40–59 |
| Red | < 40 |

A color scale legend is shown below the grid (from worst to best).

**Key Insights** (shown above the grid):
| Insight | Icon | Content |
|---|---|---|
| **Best Performing Hour** | TrendingUp (green) | Day + time + score |
| **Worst Performing Hour** | TrendingDown (red) | Day + time + score |

**Cell tooltips:** hover over any cell to see *"[Day] [HH:00] → Score: X.X"* or *"No data"*.

The grid is scrollable horizontally on mobile (min width 600 px).

### Top Failed Criteria

A ranked list of the **top 5 most frequently failing criteria** across all evaluated transcriptions.

| Element | Detail |
|---|---|
| Rank badge | Numbered 1–5 with a gradient background (red-100 → orange-100) |
| Criterion name | Truncated with a tooltip showing the full name |
| Evaluator tag | Color-coded badge (deterministic color by name hash) |
| Fail count | Absolute number of failures |
| Fail rate | Percentage with a red/orange progress bar |

Subtitle: *"Priority areas for training."*

This section is only rendered when criteria failure data is present.

---

## Participants Ranking

A full-width table of the **top 8 participants** evaluated under this evaluator, sorted by average score.

| Column | Content |
|---|---|
| **#** | Rank number. Top 3 show Trophy / Medal / Award icons. Rows where `critical_fail_rate > 30%` have a red-tinted background. |
| **Participant** | Name tag + ID in monospace + external link icon → opens the participant's analytics dashboard in a new tab |
| **Calls** | Total evaluated calls for this participant |
| **Score** | Average score — color-coded badge (same thresholds as KPI grid) |
| **Criticals** | Critical fail count. Red badge if >0; "—" if none. Tooltip: *"Calls with critical fail"* |
| **Failed** | Count of failed criteria. Tooltip shows the full list of failing criterion names. |
| **Consistency** | Standard deviation of scores as a badge. Tooltip explains the metric. |

**Consistency levels** (based on standard deviation):

| Badge | Threshold |
|---|---|
| Very consistent | σ ≤ 5 |
| Consistent | 5 < σ ≤ 10 |
| Variable | 10 < σ ≤ 15 |
| Very variable | σ > 15 |

This section is only rendered when participant data is present.

---

## Transcription Details table

A full-width, paginated table of every individual transcription included in the analysis.

### Group tabs

A horizontal tab bar filters records by workflow group:

| Tab | Behavior |
|---|---|
| **All** | Shows all transcriptions (default) |
| **No Group** | Shows only ungrouped transcriptions |
| **Pending Review** | Amber · Clock icon |
| **Under Review** | Blue · Eye icon |
| **Archived** | Purple · Archive icon |

Each tab displays a count badge.

### Filters panel

Toggled by the **Filters** button. When active, shows:

| Filter | Type |
|---|---|
| Search | Text input — matches by ID or participant tag |
| Critical only | Checkbox — shows only calls with a critical fail |
| Min Score | Number input |
| Max Score | Number input |

A **Clear** button resets all filters.

### Table columns

| Column | Format |
|---|---|
| **Date** | Full date and time (browser-localized) |
| **Score** | Color-coded badge (same thresholds as KPI grid) |
| **Duration** | `MM:SS` formatted from seconds |
| **Participant** | Name tag + ID in monospace |
| **Critical** | Red alert triangle icon if a Strict criterion failed |
| **Failed** | Failure count badge. Tooltip lists the names of all failed criteria. |
| **Group** | Group badge + pencil icon to **edit the group** |

### Row actions

- **Click row** → opens the transcription detail view in a new tab.
- **Pencil icon** on the Group column → opens **ChangeGroupModal** to assign or change the workflow group.

### Footer

- *"Showing X–Y of Z"* — indicates the current page range and total count.
- **Previous / Next** pagination buttons. 10 records per page.

---

## Page states

| State | What is shown |
|---|---|
| Loading | Animated skeleton: 4 KPI cards + 2 chart panels + 1 table skeleton |
| Error | Red banner with API error message + **Retry** button |
| Data loaded | Full dashboard (KPIs → Charts → Heatmap/Criteria → Participants → Table) |
| Initial (no query yet) | Selector card only; no dashboard content |
| Sandbox not ready | Yellow `SandboxSetupWarning` banner; selector and button disabled |

---

## Notifications

| Event | Type | Message |
|---|---|---|
| Submit without evaluator | Warning | *"You must select an evaluator to generate the analysis."* |
| Analysis generated | Success | *"Analysis generated — Dashboard data has loaded successfully."* |
| Data refreshed | Success | *"Data updated — Dashboard has been refreshed."* |
| Group updated | Success | *"Group updated — Status has been updated."* |
| Group update error | Error | API error message or *"Could not update."* |
| PDF generated | Success | *"PDF Generated — Report downloaded successfully."* |
| PDF error | Error | *"Could not generate the PDF."* |

---

## PDF Export

Triggered by the **Download PDF** button (FileDown icon) in the page header. Only available when analysis data is loaded.

Generated via `evaluatorPdfReportService`. Report filename: `Evaluator_Report_[name]_[date].pdf`.

**Report contents:**
- Title: **"EVALUATOR REPORT"**
- Evaluator name + Evaluation date
- Main KPIs: Total Calls · Average Score · Pass Rate · Critical Fails
- Critical fails warning banner (if applicable)
- Top failed criteria table
- Participants ranking table
- Transcription detail table (showing ID, date, score, duration, participant, critical, group)

---

## Notes

- Navigating to `/analytics/evaluator/:id` auto-loads the analysis if the ID is in the URL and no cached data for that ID exists. If the store already has data for that evaluator (from a previous navigation), it is restored without consuming quota.
- The **Critical alert banner** in the KPI grid is a direct shortcut: clicking it jumps to the Transcription Details table and pre-applies the *"Pending Critical"* filter, eliminating the need to scroll and filter manually.
- The **Consistency** column in the Participants Ranking reflects score variance — a participant with a low standard deviation is predictable, while a high value means their performance fluctuates significantly between calls.
- Group changes made within the Transcription Details table are applied immediately and reflected in the table without requiring a full re-fetch.
