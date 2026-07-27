# Troubleshooting and Security Best Practices

Troubleshooting references, security requirements, and diagnostics commands for App-to-App Account Linking.

---

## 🛠️ 1. Troubleshooting Reference

For dynamic troubleshooting tips and edge-case handling, refer to the [Official App-to-App Documentation](https://developer.smartthings.com/docs/devices/cloud-connected/app-to-app-linking).

| Symptom | Common Cause | Resolution |
| :--- | :--- | :--- |
| **Falls back to browser** | App not installed, or OS-level domain validation failed. | - Inspect `assetlinks.json` or AASA file hosting and CORS.<br>- Ensure signing keystore matches registrations. |
| **SmartThings redirection fails** | 1. Skipped URL-decoding.<br>2. Hardcoded redirect URI. | - Decode `redirect_uri` using UTF-8 before appending parameters.<br>- Build redirect targets dynamically. |
| **"State Token Mismatch"** | `state` token was modified or skipped. | - Store and return the exact `state` token received from the incoming intent. |
| **Works in US, fails in EU/AP** | Hardcoded `redirect_uri` domain host. | - Do **not** hardcode callback domains. Parse the `redirect_uri` dynamically. |

---

## 🔒 2. Security Requirements

1.  **Client ID Validation**: Always validate the incoming `client_id` parameter against your integration's expected Client ID. Abort linking immediately if they mismatch.
2.  **HTTPS Only**: Ensure all redirection URIs and endpoints run exclusively over TLS (HTTPS).
3.  **State Token Lifetime**: Treat the `state` parameter as single-use. Do not persist or reuse it across multiple login sessions.

---

## 🔍 3. Diagnostics Commands

Refer to Google and Apple developer guides for detailed verification diagnostics:

*   **Android (ADB Verification)**:
    1.  Check App Link Verification Status:
        `adb shell pm get-app-links com.example.yourpartnerapp` *(Status must be `verified`)*
    2.  Trigger App Link Manually via ADB:
        `adb shell am start -a android.intent.action.VIEW -c android.intent.category.BROWSABLE -d "https://applink.yourdomain.com/smartthings-link?client_id=MY_CLIENT&response_type=code&state=MY_STATE&redirect_uri=https%3A%2F%2Fc2c-us.smartthings.com%2Fc2c-app-to-app-account-linking" com.example.yourpartnerapp`
*   **iOS (Universal Links)**:
    1.  Inspect console logs and filter by process `swcd` to capture AASA parsing errors.
    2.  Toggle **Associated Domains Development** under **Settings > Developer** on your test device to bypass CDN cache.
