# Participant Analysis

**Route:** `/analytics/participant/:id`
**Component:** `AnalyticsParticipantView.vue`
**Access:** Authenticated. Sandbox must be ready. Quota-gated.

The Participant Analysis page generates a personal performance dashboard for a selected participant. It aggregates all transcriptions evaluated for that participant and organizes the results into a degradation alert, KPI summaries, an evaluator-based ranking with click-to-filter, trend charts, a weekly quality heatmap, criteria failure breakdowns, and a filterable transcription detail table.

> Each analysis generation consumes one unit of the account's `analytics_participant` quota.

The page uses a **teal/cyan color scheme** throughout, distinguishing it visually from the Evaluator Analysis (indigo/purple).

---

## Page header

| Element | Behavior |
|---|---|
| **Participant Analysis** title | Always shown. |
| Participant name (tag) | Shown in teal below the title once a participant is active. Before selection, the subtitle *"Analyze the individual performance of a participant"* is shown instead. |
| **Last updated** | Relative time since the last fetch (e.g. *"Updated 3 minutes ago"*). Only shown when data is loaded. |
| **Download PDF** (FileDown icon) | Generates and downloads a structured participant PDF report. Disabled when no data is available. |
| **Refresh** (RefreshCw icon) | Re-fetches the analysis for the current participant. Icon spins while loading. |

The Download PDF and Refresh buttons are only visible when analysis data has been loaded.

---

## Participant selector

A white card with a form at the top of the content area.

| Element | Behavior |
|---|---|
| **EntitySelector** | Searchable modal listing all available participants. Each participant in the modal shows: name (tag) + ID in monospace + calls count badge. Includes a Reload button to refresh the participant list. |
| **Generate Analysis** button | Submits the form and triggers the analysis. |

**Generate Analysis button states:**

| State | Appearance |
|---|---|
| Default | Sparkles icon + "Generate Analysis" + arrow that fades in on hover |
| Loading | Spinner + "Generating..." |
| Disabled | Grey; shown when loading, no participant is selected, or quota is exhausted |

If submitted without a participant selected, a **warning notification** is shown: *"You must select a participant to generate the analysis."*

**URL behavior:** Selecting a participant and generating updates the URL to `/analytics/participant/:participantId`. Loading the page with an ID in the URL auto-fetches the analysis. If cached data for that participant already exists in the store, it is restored without consuming quota.

---

## Query limit banner

An amber banner shown at the top of the dashboard when `metadata.query_limit_reached === true`:

> *"Limit reached: N records analyzed. There may be more data not included."*

This indicates that the analysis is based on a subset of the available transcriptions due to API record limits.

---

## Degradation Banner

A red/rose alert banner rendered **directly above the KPI grid** when the participant's recent performance has dropped significantly.

**Trigger condition:** The trend comparison between the last 20 evaluated records and the previous 20 records shows a decline greater than 10 points in average score.

| Element | Content |
|---|---|
| Icon | AlertTriangle in a red-to-rose gradient box |
| Title | *"Degradation Alert"* |
| Description | Message indicating that a significant performance drop has been detected |

This banner is purely informational — it has no interactive actions. It is hidden when there is no significant decline.

---

## KPI Grid

Four metric cards displayed in a 2×2 grid on mobile or a 4-column row on desktop. This component receives `kpis`, `trend`, and `rawData` props.

| KPI | Value | Color logic | Extra |
|---|---|---|---|
| **Total Calls** | Count of all evaluated transcriptions | — | Phone icon, teal theme. Tooltip: *"Total calls evaluated in the selected period"* |
| **Average Score** | Score out of 100 | Emerald ≥91 · Green 71–90 · Amber 51–70 · Orange 31–50 · Red <31 | **Sparkline chart** of the last 20 scores (if ≥2 data points); **trend badge** showing direction |
| **Pass Rate** | Percentage of calls that passed the threshold | Status badge: **Excellent** ≥80% · **Good** 60–79% · **Fair** 40–59% · **Low** <40% | Trending icon (up/down/flat) |
| **Criticals** | Count of calls that failed a Strict criterion | Red badge if >0. *"No critical incidents"* if 0 | Tooltip: *"Calls that failed critical criteria"* |

### Sparkline (Average Score card)

When at least 2 score records are available, the Average Score card renders a small inline SVG line chart of the last 20 scores in chronological order (left to right). The line color matches the score color of the most recent value. If fewer than 2 data points exist, a gradient progress bar is shown instead.

### Trend badge (Average Score card)

Shows the direction of recent performance based on comparing the last 20 records against the previous 20:

| Direction | Icon | Color | Tooltip |
|---|---|---|---|
| Improving | TrendingUp | Green | Recent avg, previous avg, sample size |
| Declining | TrendingDown | Red | Recent avg, previous avg, sample size |
| Stable | Minus | Grey | Recent avg, sample size |
| Insufficient data | — | Grey | Number of additional records needed |

### Critical alert banner

If `pending_critical_reviews > 0`, a clickable amber/orange banner appears above the KPI grid:

> *"N pending critical calls — Pending review · Click for details"*

Clicking it **scrolls to the Transcription Details table** and pre-filters it to show only pending critical calls.

---

## Evaluators Ranking

A full-width ranked table showing this participant's performance **broken down by evaluator/campaign**. Only rendered when evaluator data is present.

| Column | Content |
|---|---|
| **#** | Rank number. Top 3 show Trophy / Medal / Award icons with gold/silver/bronze gradient badges. Rows where `critical_fail_rate > 30%` have a red-tinted background. |
| **Evaluator** | Tag + ID in monospace (truncated) |
| **Calls** | Total evaluated calls for this evaluator |
| **Score** | Color-coded badge (same thresholds as KPI grid) |
| **Criticals** | Critical fail count. Red badge if >0; "—" if none. |
| **% Criticals** | Critical fail rate — red text if >30%, amber if >15%, grey otherwise |
| **Consistency** | Standard deviation badge: Very consistent (≤5) / Consistent (5–10) / Variable (10–15) / Very variable (>15). Tooltip explains the metric. |
| **Duration** | Average call duration in MM:SS with a timer icon |
| **Link** | External link icon → opens the evaluator's analytics dashboard in a new tab |

Shows the top 8 evaluators.

### Click-to-filter behavior

Clicking any row activates an **evaluator filter** for that campaign:

- The clicked row is highlighted in teal; all other rows are dimmed to indicate the active filter.
- An **active filter tag** (evaluator name + X button) appears in the table header.
- Clicking the X button, or clicking the same row again, **clears the filter**.
- While a filter is active, **all dashboard sections below** (KPIs, charts, heatmap, criteria, transcription table) are recalculated **client-side** from the raw data filtered to that evaluator — no new API call is made.

This allows rapid exploration of how the participant performs within a specific campaign without consuming additional quota.

---

## Dashboard Charts

A multi-panel chart section with two rows. When an evaluator filter is active, all chart data is sourced from the filtered dataset.

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

A bar chart grouping all transcriptions into 20-point score ranges (0–20, 20–40, 40–60, 60–80, 80–100).

Tooltip: *"Histogram of score distribution in 20-point ranges."*

---

## Weekly Heatmap + Top Failed Criteria

Two components displayed side-by-side in a 2-column grid (stacked on mobile).

### Weekly Heatmap

A day-of-week × hour-of-day grid showing the **average quality score** for each time slot. Unlike Evaluator Analysis where the heatmap data comes directly from the API, here the heatmap is built **client-side** from the filtered raw data — it automatically updates when an evaluator filter is active.

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

A ranked list of the **top 5 most frequently failing criteria** across all (filtered) evaluated transcriptions.

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

## Transcription Details table

A full-width, paginated table of every individual transcription included in the analysis. Participant-specific variant — the **Evaluator** column replaces the Participant column found in Evaluator Analysis.

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
| Search | Text input — matches by ID or evaluator tag |
| From | Date input — lower bound for transcription date |
| To | Date input — upper bound for transcription date |
| Min Score | Number input |
| Max Score | Number input |
| Critical only | Checkbox — shows only calls with a critical fail |

A **Clear** button resets all filters.

### Table columns

| Column | Format |
|---|---|
| **Date** | Full date and time (DD MMM YYYY HH:mm) |
| **Score** | Color-coded badge (same thresholds as KPI grid) |
| **Duration** | `MM:SS` formatted from seconds |
| **Evaluator** | Evaluator tag — clickable link opens the evaluator's detail view in a new tab |
| **Critical** | Red alert triangle icon if a Strict criterion failed; dash otherwise |
| **Failed** | Amber failure count badge. Tooltip lists the names of all failed criteria. |
| **Group** | Group badge + pencil icon to **edit the group** |

### Row actions

- **Click row** → opens the transcription detail view in a new tab.
- **Pencil icon** on the Group column → opens **ChangeGroupModal** to assign or change the workflow group.

### Programmatic filter access

The parent view can call `applyFilters()` on this component to programmatically pre-set filters. This is used when clicking the **critical alert banner** in the KPI grid — it scrolls to this table and pre-applies the *"Pending Critical"* filter automatically.

### Footer

- *"Showing X–Y of Z"* — indicates the current page range and total count.
- **Previous / Next** pagination buttons. 10 records per page.

---

## Page states

| State | What is shown |
|---|---|
| Loading | Animated skeleton: 4 KPI cards + 2 chart panels + 1 table skeleton |
| Error | Red banner with API error message + **Retry** button |
| Data loaded | Full dashboard (Degradation Banner → KPIs → Evaluators Ranking → Charts → Heatmap/Criteria → Table) |
| Initial (no query yet) | Selector card only; no dashboard content |
| Sandbox not ready | Yellow `SandboxSetupWarning` banner; selector and button disabled |

---

## Notifications

| Event | Type | Message |
|---|---|---|
| Submit without participant | Warning | *"You must select a participant to generate the analysis."* |
| Analysis generated | Success | *"Analysis generated — Dashboard data has loaded successfully."* |
| Data refreshed | Success | *"Data updated — Dashboard has been refreshed."* |
| Group updated | Success | *"Group updated — Status has been updated."* |
| Group update error | Error | API error message or *"Could not update."* |
| PDF generated | Success | *"PDF Generated — Report downloaded successfully."* |
| PDF error | Error | *"Could not generate the PDF."* |

---

## PDF Export

Triggered by the **Download PDF** button (FileDown icon) in the page header. Only available when analysis data is loaded.

Generated via `participantPdfReportService`. Report filename: `Participant_Report_[name]_[date].pdf`.

**Report contents:**
- Title: **"OPERATOR REPORT"**
- Participant name + Evaluation date
- Main KPIs: Total Calls · Average Score · Pass Rate · Critical Fails
- Critical fails warning banner (if applicable)
- Top failed criteria table
- Evaluators ranking table
- Transcription detail table (showing ID, date, score, duration, evaluator, critical, group)

---

## Key differences from Evaluator Analysis

| Feature | Evaluator Analysis | Participant Analysis |
|---|---|---|
| Color scheme | Indigo/purple | Teal/cyan |
| Degradation alert | — | DegradationBanner (red/rose, above KPI grid) |
| KPI sparkline | — | Last 20 scores on Average Score card |
| KPI trend badge | — | Improving / Declining / Stable / Insufficient data |
| Rankings section | Participants Ranking | Evaluators Ranking (click row to filter) |
| Client-side filter | — | `activeEvaluatorFilter` recalculates all sections |
| Heatmap data source | API response | Built client-side from filteredRawData |
| Table search | By participant tag | By evaluator tag |
| Table date filter | — | From / To date range inputs |
| Table subject column | Participant (name + ID) | Evaluator (tag, clickable link) |
| PDF title | "EVALUATOR REPORT" | "OPERATOR REPORT" |

---

## Notes

- Navigating to `/analytics/participant/:id` auto-loads the analysis if the ID is in the URL and no cached data for that ID exists. If the store already has data for that participant (from a previous navigation), it is restored without consuming quota.
- The **evaluator filter** (Evaluators Ranking rows) is a purely client-side operation. It does not trigger a new API call and does not consume quota. All sections update reactively in real time as the filter is toggled.
- The **Degradation Banner** is computed from the same filtered raw data — if an evaluator filter is active, the trend comparison also applies only to records from that evaluator.
- The **Consistency** column in the Evaluators Ranking reflects score variance for this participant within that campaign — a low standard deviation means the participant performs predictably under that evaluator, while a high value indicates fluctuation.
- Group changes made within the Transcription Details table are applied immediately and reflected in the table without requiring a full re-fetch.
