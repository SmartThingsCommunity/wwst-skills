# Phase 4: Registration and Testing

Procedures for Developer Console configurations and end-to-end device testing workflows.

---

## ⚙️ 1. Developer Console Setup

In the [SmartThings Developer Console](https://developer.smartthings.com/console), navigate as follows:

**Console navigation path**:
**Device Integrations** → **Schema Apps** tab → Select your Schema App → **App-to-App Linking (optional)** section

Configure the following fields in that section:

*   **Android App-to-App Link**: Enter the base App Link URL of your Android app that handles the authentication flow (e.g., `https://{your-domain}/smartthings-auth`). SmartThings will launch this URL and append the required OAuth query parameters to it.
*   **iOS App-to-App Link**: Enter the base Universal Link URL of your iOS app that handles the authentication flow (e.g., `https://{your-domain}/smartthings-auth`). SmartThings will launch this URL and append the required OAuth query parameters to it.


> [!IMPORTANT]
> **URL Matching & Domain Ownership Reminder**:
> *   **Own Domain**: The `{your-domain}` placeholder in the URLs above must be replaced with the developer's **own verified HTTPS domain** (e.g., `yourcompany.com`).
> *   **Exact Match**: The registered App-to-App Link URLs here **must exactly match** the scheme, host, and path prefix patterns configured in your mobile apps during **Phase 2 (Android)** and **Phase 3 (iOS)**. If they mismatch, the deep link handoff will fail and fall back to the browser.


### B. Already Certified/Published Schema Apps
> [!CAUTION]
> If the Schema App has **already been verified/certified**, settings are locked and cannot be edited in the console.
> 
> **Action Required**: You must **contact the SmartThings / WWST support team** directly and submit a request to apply the App-Link / Universal Link configurations to your live app.

---

## 🔍 2. Pre-Testing Diagnostics

*   **Android Link Validation**: Refer to Google Digital Asset Links API to check the mapping.
*   **iOS Link Validation**: Verify that your AASA file is served over HTTPS with no redirects and a strict `application/json` Content-Type header.

---

## 📱 3. Device Testing Procedures

To test the App-to-App Account Linking flow on a physical device, you must enable Developer Mode on your SmartThings mobile app.

*   **Developer Mode Activation**: Refer to the [SmartThings Developer Mode Guide](https://developer.smartthings.com/docs/devices/enable-developer-mode) for the detailed step-by-step procedure to enable Developer Mode on the SmartThings app.
*   **No Invitations**: App-to-App Account Linking testing is **not** supported via Schema Invitations.
*   **Core Testing Scenarios**: Verify the following integration flows:
    - **Normal Success Handoff**: The native app launches, completes authentication, and returns the authorization code to SmartThings via the `code` query parameter.
    - **Failure / Cancel Handoff**: If the user declines or authentication fails, return `error=unauthorized` back to SmartThings.
    - **App Not Installed Fallback**: Verify that the flow falls back to standard browser OAuth when the native app is not installed.
*   **Market/Production Signing Verification**: Before submitting the app for WWST certification, verify the flow using a build signed with your **Production/Market keys** (Google Play App Signing key or iOS Distribution profile). Make sure the production SHA-256 fingerprint or App ID matches the live `assetlinks.json` or AASA configurations.
