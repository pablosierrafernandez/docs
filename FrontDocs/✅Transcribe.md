# Transcribe

**Route:** `/transcribe`
**Component:** `CreateTranscriptionView.vue`
**Access:** Authenticated. Sandbox must be ready.

The Transcribe page is where you submit audio or video content to the Heify processing pipeline. You choose a **Configuration** (required) that defines how the content will be processed, optionally attach an **Evaluator** to score the transcript against quality criteria, optionally link a **Participant** to attribute the result to a specific person, and then provide the audio either as a local file upload or a public URL.

---

## Transcription Parameters section

The form is presented as a single card with a blue/indigo header ("Transcription Parameters").

### Parameter layout — tree structure

The three parameters are arranged as a visual tree to communicate their relationship:

```
[Configuration]  ← Required
       |
  ┌────┴────┐
[Evaluator] [Participant]  ← Both optional
```

On desktop the evaluator and participant selectors sit side-by-side in a 2-column grid. On mobile they stack vertically.

---

### Configuration *(required)*

Blue-tinted card with a **REQUIRED** badge.

A searchable **ConfigurationSelector** component loads the list of existing configurations. Selecting one associates all its settings (language, summary, extraction fields, webhooks) with this transcription job.

| State | Behavior |
|---|---|
| Loading | Shows a spinner inside the selector. |
| Loaded | Searchable dropdown of configuration tags. |
| Error | Shows an error message with a Reload button. |
| None selected + submit | Inline error: *"You must select a configuration."* |

---

### Evaluator *(optional)*

Violet-tinted card with an **OPTIONAL** badge. Uses the **EntitySelector** component.

Clicking opens a modal titled **"Select Evaluator"** with subtitle *"Choose the evaluation criteria template"*. The modal lists all available evaluators showing:
- Evaluator name (tag)
- ID in monospace
- Criteria count badge (e.g. `5 criteria`)
- A Reload button to refresh the list

Selecting an evaluator causes the AI to score the transcript against that evaluator's criteria after processing completes.

---

### Participant *(optional)*

Emerald-tinted card with an **OPTIONAL** badge. Uses the **EntitySelector** component.

Clicking opens a modal titled **"Select Participant"** with subtitle *"Associate the transcription with an existing participant"*. The modal lists all participants showing:
- Participant name (tag)
- ID in monospace
- Department badge (if the participant has a `department` metadata field)
- A Reload button to refresh the list

Selecting a participant links the transcription result to that person for analytics and reporting.

---

## Upload Method toggle

Below the parameter selectors, a pill toggle switches between two upload modes:

| Option | Icon | Behavior |
|---|---|---|
| **Local File(s)** | Upload cloud | Upload one or more files from the local device. Default selection. |
| **Public URL** | Link | Provide a publicly accessible URL to an audio/video file. |

The active button scales up slightly to indicate selection.

---

## Local File(s) mode

### Drop zone

A large dashed area. Interaction options:
- **Drag & drop** files onto it (border turns violet while dragging over).
- **Click** anywhere on the zone to open a file picker.

Displays:
- Upload icon
- Text: *"Drag files here or click to select"* (with "click to select" underlined in violet)
- Supported formats label listing all accepted extensions

**Supported formats:** `aac`, `aiff`, `amr`, `asf`, `flac`, `mp3`, `ogg`, `wav`, `webm`, `m4a`, `mp4`

The drop zone becomes disabled and dimmed once a batch upload has started.

**File validation:**
- Files with unsupported MIME types or extensions are silently filtered out. A warning notification appears: *"N file(s) were rejected due to an unsupported format."*
- Maximum **25 files** per submission. Attempting to add more shows: *"You can only process a maximum of 25 files at a time."*

---

### Processing Queue

Appears as soon as at least one file is added. Disappears if all files are removed.

**Header bar:**
- "Processing Queue" title with a count badge (e.g. `3`) and `/ 25` limit indicator
- **Clear Queue** button — removes all pending/error files. Disabled while upload is in progress.

**File list** (scrollable, max 28rem height). Each item shows:

| Element | Description |
|---|---|
| Status icon | Violet file-audio icon (pending) · Blue spinner (uploading) · Green checkmark (success) · Red triangle (error) |
| File name | Truncated filename |
| Name input | Optional free-text label for the transcription (e.g. *"Q3 team meeting"*). Disabled while uploading. |
| File size | Shown in MB |
| Status pill | **Queued** (grey) · **Uploading...** (blue) · **Started** (green) · **Error** (red) |
| Transcription ID | Shown after success. Monospace text with: copy button (turns green for 2 s) + external link to the detail view. |
| Error message | Shown in red below the status pill if the upload failed. |
| Remove button (✕) | Removes the file from the queue. Disabled while uploading. |

Files are processed **sequentially** with a 2-second pause between each. The queue does not re-process already-successful files if the form is re-submitted — only pending and errored files are retried.

---

## Public URL mode

Two fields:

| Field | Required | Description |
|---|---|---|
| **File URL** | Yes | Direct public URL to the audio/video file (e.g. `https://my-server.com/audio.mp3`). |
| **Transcription Name** | No | Optional human-readable label for the transcription (e.g. `Q3 team meeting`). |

After successful submission, a green result card appears below the fields showing:
- *"Transcription Started"* message
- The transcription ID in monospace
- Copy button (turns green for 2 s after clicking)
- External link to the detail view

The URL and name inputs are disabled once submission is in progress or complete.

---

## Info cards

Three informational cards appear below the main form:

| Card | Links to |
|---|---|
| **Documentation** | Heify API documentation |
| **Limits** | Processing limits reference page |
| **Support** | Contact support |

---

## Submit button *(sticky)*

Fixed at the bottom of the form. Behavior changes based on state:

| State | Appearance | Label |
|---|---|---|
| Ready (local, files queued) | Violet gradient | **Process N File(s)** |
| Ready (URL mode) | Violet gradient | **Process File** |
| Uploading (local) | Spinner + bouncing dots | **Processing Files** |
| Sending (URL) | Spinner + bouncing dots | **Sending Request** |
| Completed | Dark grey | **Process More Files** |

The button is **disabled** when:
- Sandbox is not ready
- Upload is in progress
- Local mode: no files in the queue

**"Process More Files"** resets the queue (local) or the URL/name fields (URL mode), but **retains** the Configuration, Evaluator, and Participant selections. Click it to start a new batch without re-selecting these settings.

---

## Form validation

Validation runs on submit. Errors clear automatically when the field is corrected.

| Error | Condition |
|---|---|
| *"You must select a configuration."* | No configuration selected. |
| *"You must select at least one file to upload."* | Local mode with empty queue. |
| *"The file URL is required."* | URL mode with empty URL field. |

---

## Notifications

| Event | Type | Message |
|---|---|---|
| Unsupported file(s) dropped | Warning | *"N file(s) were rejected due to an unsupported format."* |
| File limit exceeded | Warning | *"You can only process a maximum of 25 files at a time."* |
| File uploaded successfully | Success (persistent) | *"Transcription Started"* with the transcription ID |
| File upload error | Error | *"Error in File: [filename]"* with the error message |
| URL request error | Error | *"Request Error"* with the server message or a default fallback |

Success notifications for file uploads are **persistent** (do not auto-close) so the user can copy the transcription ID from the notification panel.

---

## Notes

- The **Configuration** field is the only strictly required field. Evaluator and Participant are fully optional and independent of each other.
- Selecting an Evaluator triggers AI quality scoring after the transcript is ready. Results appear in Analytics.
- Selecting a Participant links the transcription to that person. Results appear in the Participant's analytics profile.
- The 2-second delay between file uploads in a batch is intentional — it prevents overwhelming the API with simultaneous requests.
- After a batch completes, only files in `pending` or `error` state are reprocessed on re-submit. Successfully uploaded files are not retried.
