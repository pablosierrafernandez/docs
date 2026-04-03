Here is the English translation of your authentication documentation, keeping the same technical tone and structure as the previous component.

# Sandbox Authentication

**Route:** `/login`
**Component:** `LoginView.vue`
**Access:** Public (no prior authentication required)

The authentication screen is the entry point to the Heify Sandbox. It uses a two-step flow based on a passwordless **email OTP** (One-Time Password). The user receives a 6-digit code in their email and enters it to gain access.

---

## Authentication Flow

```text
[Step 1] Enter email
         ↓
         System sends OTP code to email (valid for 3 minutes)
         ↓
[Step 2] Enter 6-digit code
         ↓
         Successful verification → redirect to /sandbox
```

---

## Step 1 — Email

**Card header title:** `Autenticación` / `Authentication`

### UI Elements

| Element | Behavior |
|---|---|
| Email field | `email` type input. Required. Browser autocomplete enabled. |
| T&C Checkbox | Must be checked to submit. If unchecked and submission is attempted, it shows an error. |
| T&C Link | Opens `heify.com` in a new tab. |
| **Send Code** button | Disabled if: (1) the request is in progress, or (2) the T&C checkbox is not checked. During loading, it shows a spinner + `Enviando código...` / `Sending code...` text. |
| Error block | Appears below the checkbox when there is an error. Displays the message alongside an alert icon. |
| **No account?** | Link in the card footer → `heify.com/contact` in a new tab. |

### Actions

- **Submit** (`Enter` or button click): calls `requestOTP(email)`. If the email does not exist in the system, it shows an error. If successful, it advances to Step 2.

---

## Step 2 — OTP

**Card header title:** `Verificación` / `Verification`

### UI Elements

| Element | Behavior |
|---|---|
| Confirmation banner | Blue strip showing the email to which the code was sent. Non-editable. |
| 6 numeric inputs | Each box accepts 1 digit. Only numeric characters are allowed. Focus automatically advances to the next input when typing a digit. |
| Timer | Shows the remaining time in an `M:SS` format. When it reaches zero, it displays the expired code message in amber. |
| **Verify Code** button | Disabled until all 6 digits are filled or while the verification request is in progress. |
| **Resend code** | Disabled while the timer is running or a request is in progress. Clicking it clears the inputs and resends the code. |
| **Change email** | Returns to Step 1. Clears the OTP inputs and resets the state. |
| Verification overlay | Covers the entire card while the code is being checked. Shows a rotating spinner + shield icon + `Verificando...` / `Verifying...` text and subtitle. Uses a fade transition. |

### Keyboard Navigation in OTP inputs

| Action | Result |
|---|---|
| Type a digit | Fills the box and moves focus to the next one |
| `Backspace` on an empty box | Moves focus to the previous box |
| `←` / `→` | Moves focus between boxes |
| Paste (`Ctrl+V` / `Cmd+V`) | Extracts digits from the clipboard, distributes them across the boxes, and sets focus on the last filled box |

---

## Error States

Errors are displayed in a red block with a triangular alert icon, located below the active form.

| Error Code | ES Message | EN Message | When it occurs |
|---|---|---|---|
| `userNotFound` | Usuario no encontrado. Contacta con el administrador. | User not found. Contact the administrator. | The entered email is not registered in the system. |
| `unexpectedResponse` | Respuesta inesperada del servidor. | Unexpected server response. | The backend returns an unrecognized response. |
| `sendCodeError` | Error al enviar el código. | Error sending the code. | Generic failure when calling the send endpoint. |
| `incorrectCode` | Código incorrecto. Inténtalo de nuevo. | Incorrect code. Try again. | The entered code does not match the one sent. |
| `tooManyAttempts` | Demasiados intentos. Solicita un nuevo código. | Too many attempts. Request a new code. | Cognito blocks the user due to excessive failed attempts. When this occurs, the system automatically resets to Step 1. |
| `expiredCode` | El código ha expirado. Solicita uno nuevo. | The code has expired. Request a new one. | More than 3 minutes have passed since the code was sent. |
| `verifyCodeError` | Error al verificar el código. | Error verifying the code. | Generic failure when calling the verification endpoint. |
| `noEmailRegistered` | No hay email registrado. Introduce tu email de nuevo. | No email registered. Enter your email again. | An attempt is made to resend the code, but there is no email saved in the session. |
| `termsRequired` | Debes aceptar los Términos y Condiciones. | You must accept the Terms and Conditions. | An attempt is made to submit the form without checking the T&C checkbox. |

---

## Language Switch

In the top right corner, there is a button with a globe icon that toggles between **ES** and **EN**. This preference is saved in `localStorage` under the key `locale` and persists across sessions.

---

## What happens after login

Once the OTP code is successfully verified:

1. The system obtains the AWS Cognito **session token**.
2. The user's client/organization information (`clientInfo`) is loaded.
3. The **Sandbox API Key is auto-provisioned** (a background process that may take a few seconds; the "Configurando sandbox..." banner reflects this within the app).
4. The user is redirected to **`/sandbox`**.

> Access to the Sandbox is linked to an email previously registered by the administrator. If the email is not registered, the login fails with the `userNotFound` error. To request access, users should use the **"¿Aún no tienes cuenta?"** (Don't have an account yet?) link in the screen's footer.

---

## Technical Notes

- Identity Provider: **AWS Cognito** (`CUSTOM_WITHOUT_SRP` flow).
- OTP code duration: **3 minutes** (180 seconds). The timer is calculated in real-time from the expiration timestamp, rather than using a fixed interval counter, ensuring accuracy even if the browser tab is in the background.
- The OTP code consists of exactly **6 numeric digits**.