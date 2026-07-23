# Phase 2: Android Coding Checkpoints

When implementing Android App Links for SmartThings App-to-App Linking, the AI assistant must implement and verify the following checkpoints:

---

## 🔑 Coding Checkpoints

*   **[ ] Manifest Intent-Filter URL Matching**:
    Configure the `<intent-filter>` in `AndroidManifest.xml` with `android:autoVerify="true"` for the authentication Activity. The `<data>` element must define the exact `scheme` (`https`), `host` (your domain), and `pathPrefix` (e.g., `/smartthings-auth`) that matches the **Android App-to-App Link** registered in the SmartThings Developer Console.
*   **[ ] Intent Query Parameter Parsing**:
    Extract the incoming deep link URL from the intent using `intent.data` or `intent.dataString`. Parse this URL (e.g., via `Uri.parse()`) and dynamically retrieve the standard OAuth parameters:
    - `client_id`: The client ID of your integration.
    - `redirect_uri`: The target callback URL hosted by SmartThings.
    - `response_type`: Expected token key (usually `code`).
    - `state`: Verification token to prevent CSRF.
    Use standard Android `Uri` query parser methods (e.g., `getQueryParameter("key")`) to extract these values.
*   **[ ] Client ID Security Check**:
    Validate the incoming `client_id` parameter against the expected Integration Client ID. If they mismatch, terminate the authentication immediately to prevent link hijacking.
*   **[ ] User Consent & Disclosure Screen** ⚠️:
    Before presenting the login screen or issuing any authorization code, **display a clear consent screen** informing the user that:
    - Their account will be linked to the SmartThings app.
    - SmartThings will be able to discover and control their registered devices.
    - They can revoke this access at any time via their account settings.

    The consent screen must require an **explicit user action** (e.g., tapping "Agree" / "Authorize" button). If the user declines or cancels, trigger the **failure callback** (`error=unauthorized`).
*   **[ ] Dynamic Callback Assembly & UTF-8 Handling**:
    Do not hardcode the SmartThings callback URL. Extract the `redirect_uri` query parameter from the incoming deep link. Because the Android `Uri` parser automatically URL-decodes query values, parse the extracted `redirect_uri` string to construct the final callback target domain and path.
*   **[ ] Fixed Callback Query Key ('code')**:
    Append the authorization code to the success callback strictly using the `code` query parameter key (e.g., `?code=AUTH_CODE`). Do not dynamically map other key names based on `response_type`.
*   **[ ] State Parameter Match**:
    Retrieve the exact `state` token received from the incoming intent, and return it exactly as-is under the query parameter key `state` in the callback URL.
*   **[ ] Failure Handoff Handling**:
    If authentication fails, is rejected, or is cancelled, build the redirect callback URL by appending the `error` query parameter with a descriptive error string (e.g., `error=unauthorized`) and the exact `state` received, and redirect back to SmartThings.
*   **[ ] Native Redirection via ACTION_VIEW**:
    To hand control back to the SmartThings app, construct the callback URL (e.g., `https://c2c-us.smartthings.com/c2c-app-to-app-account-linking?state=...&code=...` or with `error=unauthorized`) and launch it using a native Intent with `Intent.ACTION_VIEW`. Do not load this URL in a WebView.

---


## 💻 CLI Verification (Testing your App Link)

To test if your Android app is configured correctly to receive the App Link without needing the SmartThings app, run the following ADB command in your terminal:

```bash
adb shell am start -W -a android.intent.action.VIEW \
    -d "https://{your-domain}/smartthings-auth?client_id={your-client-id}&response_type=code&state=xyz123&redirect_uri=https%3A%2F%2Foauth.smartthings.com" \
    {your-package-name}
```

*Replace `{your-domain}`, `{your-client-id}`, and `{your-package-name}` with your actual parameters. If the setup is correct, this command will launch your authentication Activity directly on the connected device.*


