Here is the translation of your documentation into English. I used standard UI/UX and Vue.js terminology (like "spinner," "badge," and "auto-provisioning") to ensure it sounds natural for a development context. 
# API Keys, Sandbox & Team

**Route:** `/api-keys`
**Component:** `ApiKeysView.vue`
**Access:** Authenticated. The **Team** section is only visible to users with the **admin** role.

Central page for API access configuration. It groups three independent blocks: Sandbox environment status, personal API keys management, and team member management.

---

## Section 1 — Sandbox Status

The first block of the page displays the current status of the testing environment (Sandbox). The Sandbox auto-configures upon login: the system automatically creates or reuses a reserved API key (`API_KEY_SANDBOX_RESERVED`) and validates it. The user does not need to do anything manually under normal conditions.

### Possible States

| State | Color | When it appears |
|---|---|---|
| **Validating connection...** | Blue · spinner | Upon page load or when launching a manual validation. |
| **Sandbox ready** · `Auto-Configured` badge | Emerald green | The Sandbox key is active and responding correctly. |
| **Sandbox not configured** | Amber | The auto-configuration process failed or has not been completed. |

### Available Actions

| Action | State when it appears | Description |
|---|---|---|
| **Test Connection** | Only when Sandbox is **Ready** | Launches a manual key validation. Displays a spinner during the check. |
| **Retry Auto-Configuration** | Only when Sandbox is **not configured** | Re-runs the complete auto-provisioning process. |

> The READY state displays a green card border (`ring-2 ring-emerald-100`) as a visual indicator of proper functioning.

---

## Section 2 — API Keys Management

List of all API Keys associated with the account. It includes both the reserved Sandbox key and any custom keys created by the user.

### Keys Table

| Column | Detail |
|---|---|
| **Name** | Descriptive name assigned when creating the key. The Sandbox key displays a blue `Sandbox` badge and always appears first in the list. |
| **Created on** | Creation date formatted according to the user's language. |
| **Actions** | Trash icon button to delete. |

- **Limit:** maximum of **10 keys** per account. The "New API Key" button is disabled when the limit is reached.
- **Empty state:** if there are no keys, it shows an empty state with an icon and description.
- **Loading state:** while fetching the keys, it displays a centered spinner.

### Modal — Create API Key

Opens when clicking **New API Key**.

| Field / Element | Behavior |
|---|---|
| Descriptive name | Free text. Required. Example: `iOS App`, `Zapier Integration`. |
| Validation error | Appears below the field if creation is attempted without a name. |
| **Create Key** button | Displays a spinner during creation. |
| Cancel button / click outside | Closes the modal without creating anything. |

Upon successful creation, the creation modal closes and the **Key viewing modal** opens automatically.

### Modal — View Created Key

> This modal appears **only once** after creating the key. Once closed, the full value of the key will no longer be accessible from the platform.

| Element | Behavior |
|---|---|
| Orange warning | "This is the only time you will see this key. Copy it now and store it in a safe place." |
| Key field | Displays 40 dots (`●`) by default. |
| **Eye** button (show/hide) | Toggles between the actual value and dots. |
| **Copy key** button | Copies the value to the clipboard. Changes to green with a check icon for 2.5s after copying. |
| Confirmation checkbox | "I have copied and saved this key in a safe place." Required to close. |
| **Got it, I saved it** button | Disabled until the checkbox is checked. Upon confirmation, closes the modal. |

### Modal — Delete API Key

Opens when clicking the trash icon in a key's row.

| Element | Behavior |
|---|---|
| Description | Mentions the exact name of the key to be deleted. |
| Red warning | "This action cannot be undone." |
| **Yes, delete** button | Executes the deletion with a spinner. |
| Cancel button / click outside | Closes without deleting. |

---

## Section 3 — Team *(admins only)*

This section **is not visible to sub-users** (invited members). It only appears if the authenticated user has the administrator role.

Allows inviting collaborators to access the platform under the same account, without needing to create separate accounts.

### Quota and limit

The header displays the counter `X / Y members`. The add button is disabled when the limit is reached; its text changes to "Limit reached".

### Members Table

| Column | Detail |
|---|---|
| **Email** | Member's email address, with a purple initials avatar. |
| **Added on** | Formatted registration date. |
| **Actions** | Trash icon button to revoke access. Displays an inline spinner while processing the deletion. |

### Modal — Add member

Opens when clicking **Add member**.

| Element | Behavior |
|---|---|
| Description | "The user will receive an invitation email and will be able to log in immediately with their email via OTP." |
| Restrictions note (blue) | "Members will not be able to delete, add, or view other members." |
| Email field | Required. Email format validation. Supports `Enter` to submit. |
| **Send invitation** button | Displays a spinner + `Sending...` text during the request. |
| Cancel button / click outside | Closes without adding. |

### Modal — Revoke access

Opens when clicking the trash icon in a member's row.

| Element | Behavior |
|---|---|
| Title | Includes the affected user's email: "Remove access for `email`". |
| Description | "This action will immediately revoke this user's access to the platform. They will not be able to log in again until they are invited anew." |
| Red warning | "This action cannot be undone." |
| **Yes, remove access** button | Executes the revocation with a spinner. |

---

## Sub-user (invited members) Restrictions

Users added as team members have restricted access:

- **Cannot see** the Team section (cannot know who else has access).
- **Cannot** invite or delete other members.
- The rest of the platform's features are available normally.

---

## System Notifications

| Event | Type | Message |
|---|---|---|
| Key copied to clipboard | Success | "Key copied to clipboard" |
| Sandbox connection validated successfully | Success | "Successful Connection! The Sandbox key is valid and the testing environment is ready." |
| Error validating the Sandbox key | Error | "Validation Error. The provided key is invalid or could not connect." |
| Error copying | Error | "Copy Error. The key could not be copied. Please do it manually." |
| Member invitation sent | Success | "Invitation sent to `email`" |
| Member access revoked | Success | "Access successfully revoked" |
| Error revoking access | Error | Server error message |