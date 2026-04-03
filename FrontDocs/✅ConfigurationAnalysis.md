# Configuration Analysis

**Route:** `/analytics/:id`
**Component:** `AnalyticsView.vue`
**Access:** Authenticated. Sandbox must be ready. Quota-gated.

The Configuration Analysis page generates a comprehensive, AI-powered analytics report for a selected configuration. It aggregates all transcriptions processed under that configuration and organizes the results into seven collapsible dashboard sections covering AI insights, distribution metrics, data quality, anomaly detection, extraction field analysis, temporal trends, and cross-variable relationships.

> Each analysis generation consumes one unit of the account's analytics quota.

---

## Configuration selector form

A simple form at the top of the page (hidden when in fullscreen mode).

| Element | Behavior |
|---|---|
| **ConfigurationSelector** | Searchable dropdown listing all available configurations. Takes up 2/3 of the row on desktop. |
| **Generate Analysis** button | Submits the form and triggers the analysis. Takes up 1/3 of the row on desktop. |

**Generate Analysis button states:**

| State | Appearance |
|---|---|
| Default | Sparkles icon + "Generate Analysis" + arrow that fades in on hover |
| Loading | Spinner + "Analyzing" + bouncing dots |
| Disabled | Grey; shown when loading, Sandbox is not ready, or quota is exhausted |

**Validation:** If submitted without a configuration selected, an inline error appears below the selector: *"You must select or enter a configuration ID."*

**URL behavior:** Selecting a configuration and generating an analysis updates the URL to `/analytics/:configurationId`. If the page is loaded with an ID in the URL, the configuration selector is pre-filled and any previously cached data for that ID is shown immediately.

---

## Scroll progress bar

A thin, fixed bar at the very top of the browser window. Visible only when analysis data is loaded.

- **Blue** while scrolling through the dashboard.
- **Turns green** once the user has reached the bottom of the page.
- Tracks scroll position inside the fullscreen container when fullscreen mode is active.

A floating **chevron-down** indicator also appears at the bottom-right of the screen when there is more content below. It disappears when you scroll to the bottom.

---

## Dashboard header

Rendered by the **AnalyticsHeader** component. Visible once results are successfully loaded.

### KPI chips
| Chip | Value |
|---|---|
| **Files Analyzed** | Total number of audios included in this analysis |
| **Total Duration** | Combined duration of all analyzed audios |

### Controls
| Control | Behavior |
|---|---|
| **Last updated** | Relative time since the last analysis fetch (e.g. "Updated 5 minutes ago") |
| **Timezone selector** | Dropdown to change the reference timezone for all time-based charts in the dashboard |
| **Full Screen** button | Enters native browser fullscreen mode. The dashboard fills the entire screen and all surrounding page chrome is hidden. |
| **Export** button | Opens a dropdown with three export options (see below) |

### Export options
Clicking **Export** opens a dropdown menu:

| Option | Icon | Format | Behavior |
|---|---|---|---|
| **As HTML** | Blue document | `.html` file | Renders all dashboard sections to DOM and packages them into a self-contained HTML file. |
| **Nativa** | Blue chart | PDF via print | Renders all sections and opens the browser print dialog. Choose "Save as PDF" in the dialog. |
| **Ligera** | Green zap | Structured PDF | Generates a lightweight, structured PDF report using `analyticsPdfReportService` (does not render the live DOM). |

During any export, a **progress notification bar** is shown at the bottom of the screen with a progress percentage and step-by-step messages (e.g. *"Forcing render of all sections..."*, *"Collecting styles..."*). This bar is separate from the global notification system and dismisses automatically when the export completes or fails.

---

## Fullscreen mode

When fullscreen is active:
- The dashboard container occupies the entire screen.
- A floating **Exit Fullscreen** button (Minimize icon) is fixed at the top-right corner of the screen.
- Clicking it (or pressing Escape) exits fullscreen.
- The configuration form is hidden in fullscreen.

---

## Dashboard sections

All sections share a common structure:

- Wrapped in a **CollapsibleCard** — click the header bar to expand or collapse the section.
- Sections are only rendered if their corresponding data is present in the API response.
- The **Synthesis** section is expanded by default. All other sections start collapsed.
- An **Observer** component tracks which section is currently visible in the viewport and updates the sticky section-navigation sub-menu in the dashboard header.

---

### Section 1 — Synthesis

*Sparkles icon (blue) · Open by default*

AI-generated top-level analysis of the entire dataset.

| Sub-section | Content |
|---|---|
| **Executive Narrative** | A prose summary describing the dataset's key patterns, notable findings, and overall trends. |
| **Actionable Insights** | A bulleted list of specific, prioritized recommendations derived from the data. |

---

### Section 2 — General Distribution

*ChartPie icon (grey)*

Key distribution metrics across all analyzed audios.

#### Duration
Minimum, average, and maximum audio duration across all transcriptions.

#### Speakers
- Average number of speakers per audio.
- Distribution chart/table showing how many audios have 1, 2, 3, ... N speakers.

#### Languages
Distribution of languages detected in the dataset — count and percentage per language.

---

### Section 3 — Quality and Confidence

*CheckCircle icon (green)*

Measures the completeness and reliability of the extracted data.

#### Completeness
A horizontal progress bar for each extraction field defined in the configuration:
- Bar fill = percentage of audios where that field was successfully populated.
- Label: `validCount / totalCount valid`.

#### Audios to Review
A paginated table of audios that have an unusually high number of missing extraction fields — these are the records most likely to require manual inspection.

| Column | Detail |
|---|---|
| Missing Fields | Count of missing fields (e.g. "3 of 5 missing fields") |
| ID | Transcription ID with a Copy button |
| Group | Group badge with a **Change Group** button (updates directly from this table) |
| Date | Creation date |
| Details | External link to the transcription detail view (new tab) |

**Pagination:** Previous / Next / Page X of Y.

**Empty state:** *"All is well! No audios in need of review were found."*

---

### Section 4 — Anomalies

*AlertTriangle icon (amber)*

Detects audios with values that deviate significantly from the normal pattern using the **IQR (Interquartile Range) method**.

Three anomaly categories are analyzed:

| Category | What is analyzed |
|---|---|
| **Daily Volume** | Days with an abnormally high or low number of submitted audios |
| **Metadata** | Metadata fields (e.g. duration) with outlier values |
| **Extracted Data** | Extracted fields with values outside the expected range |

For each category, the expected normal range is shown along with a table of outlier items:

| Column | Detail |
|---|---|
| Anomalous Value | The outlier value with a **High** or **Low** indicator badge |
| ID | Transcription ID with a Copy button |
| Group | Group badge with Change Group button |
| Date | Date |
| Details | External link (new tab) |

**Pagination:** Previous / Next / Page X of Y.

**Method note** (at the bottom): *"The limits are calculated using the IQR (Interquartile Range) method, which identifies values that deviate significantly from the normal pattern."*

**Empty state:** *"No anomalies detected. All analyzed values are within the normal ranges."*

---

### Section 5 — Extraction Fields

*FileJson icon (blue)*

A dedicated analysis card for each extraction field defined in the configuration.

**Card header:** Field name + **Completeness %** + `validCount / totalCount valid`.

**Card body** varies by field type:

#### Number fields
- **Descriptive Statistics**: Average, Median, Sum, Minimum, Maximum
- **Distribution histogram**: bar chart grouping values into ranges

#### String fields
- **Unique Values Found**: total count of distinct values
- **Value Frequency chart**: horizontal bar chart of the top N most frequent categories
- Truncation warning if not all categories are shown: *"Showing the top N of M most frequent categories."*

#### Boolean fields
- **Positive Percentage**: large prominent number showing `true` rate
- **Count**: `trueCount of totalCount`
- **Distribution bar**: proportional True / False visual

**Empty state** (if no fields configured): *"No Fields to Analyze — This configuration has no extraction fields defined or there is no data for them."*

---

### Section 6 — Temporal Analysis

*TrendingUpDown icon (indigo)*

Two sub-analyses inside a single collapsible card, separated by a divider.

#### Temporal Evolution

A multi-line time-series chart showing how metrics evolved over the period covered by the dataset.

- **Volume line**: daily count of submitted audios.
- **Metric lines**: one line per extracted numeric field, showing its daily average.
- Trend lines are **interpolated** — they remain continuous even on days with no data.
- Requires at least **2 temporal data points** to render; shows *"No data available"* otherwise.

**Controls:**
| Control | Behavior |
|---|---|
| Metric checkboxes | Show or hide individual metric lines. Select-all / Deselect-all shortcuts available. |
| **Reset zoom** button | Returns the chart to its default zoom level after the user zooms in. |

**Summary table** (below the chart):

| Column | Value |
|---|---|
| Metric | Name of the extracted field |
| Maximum | Peak value in the period |
| Minimum | Lowest value in the period |
| Average | Mean across all days |
| Trend | Direction (improving / degrading / stable) |

#### Activity Heatmap

A day-of-week × hour-of-day grid showing when audios are most frequently submitted.

- **X-axis**: Hours of the day (0–23)
- **Y-axis**: Days of the week (Monday–Sunday)
- **Color intensity**: light (low activity) → dark (high activity)
- A color legend (Low → High) is shown below the grid.

**Key Insights panel** (auto-generated):

| Insight | Content |
|---|---|
| **Peak Time** | The specific day and hour with the highest audio count |
| **Busiest Day** | The weekday with the highest total across all hours |
| **Most Frequent Hour Range** | The hour of day (across all days) with the most accumulated audios |

**Empty state:** *"No audios with a date were found for this analysis."*

---

### Section 7 — Relationships and Patterns

*Workflow icon (orange)*

Two advanced analyses inside a single collapsible card.

#### Correlations

A correlation matrix comparing all numerical extraction fields against each other.

- Each cell shows a correlation coefficient (−1 to +1).
- **Cell colors**: blue = positive correlation · red = negative correlation · grey = weak/no correlation.
- Tooltip shows the number of data pairs used to calculate each value.

**Key Insights panel:**

| Insight | Description |
|---|---|
| **Strongest Positive Correlation** | The pair of fields that move most together |
| **Strongest Negative Correlation** | The pair of fields that move most inversely |
| **Weakest Association** | The pair with the least relationship |

**Empty state:** *"At least two numerical fields with data are required for this analysis."*

#### Co-occurrence

One co-occurrence matrix per array or categorical extraction field. Shows how often two values from the same field appear together in the same audio.

- **Grid**: rows and columns are the distinct values of the field; cells show co-occurrence count.
- **Color levels**: Very Low → Low → Medium → High (4 levels).
- Tooltip: total appearances + co-occurrence count.

**Key Insights panel:**

| Insight | Description |
|---|---|
| **Most Frequent Pair** | The two values that appear together most often, with their co-occurrence count |
| **Most Common Item** | The single value that appears most frequently across all audios |

**Truncation warning:** If there are more unique values than the matrix can display, a warning notes *"The matrix displays the N most frequent items out of M."*

**Empty state:** *"At least two items with joint appearances are required for this analysis."*

---

## Page states

| State | What is shown |
|---|---|
| Loading | **AnalyticsSkeleton** — animated skeleton placeholder cards mimicking the dashboard layout |
| Error | Red banner with the API error message and a **Retry** button that re-runs the analysis |
| No data (0 audios) | Dashboard header is shown, but the sections area shows *"Analysis complete, no data to display — No processed files were found."* |
| Ready (no query yet) | Blue card: *"Ready to analyze — Select a configuration and generate a complete report."* |
| Sandbox not ready | Yellow `SandboxSetupWarning` banner; form and button are disabled |

---

## Notifications

| Event | Type | Message |
|---|---|---|
| Analysis generated successfully | Success | *"Analysis Generated"* · *"The analysis has been generated successfully."* |
| Analysis error | Error | *"Analysis Error"* · API error message or *"An unexpected error occurred."* |
| Export completed (PDF / HTML) | Success | *"PDF/HTML Generated Successfully"* · *"The file has been downloaded."* |
| Export error | Error | *"Export Error"* · *"Could not generate the {type} file."* |

---

## Notes

- The analytics quota is consumed **per generation** — re-generating the same configuration a second time costs another unit. Previously cached results for a given configuration ID are shown for free when navigating back to the same URL.
- Not all sections are always present. A section is only rendered if the corresponding data was returned by the API (e.g. the Correlations section only appears if at least two numerical extraction fields exist).
- The **Export Nativa** (PDF) option requires the browser not to block the print pop-up window. If blocked, a fallback message describes how to use Ctrl+P / Cmd+P manually.
- The **Timezone selector** in the dashboard header affects only the time-based sections (Activity Heatmap and Temporal Analysis). It does not reload data from the API.
