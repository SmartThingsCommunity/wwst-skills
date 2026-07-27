# Phase 3: iOS Coding Checkpoints

When implementing iOS Universal Links for SmartThings App-to-App Linking, the AI assistant must implement and verify the following checkpoints:

---

## 🔑 Coding Checkpoints

*   **[ ] Associated Domains & AASA Path Matching**:
    Enable the `Associated Domains` capability in Xcode and add the entry `applinks:yourdomain.com`. The `apple-app-site-association` file on your server must allow the matching path (e.g., `/smartthings-auth`) that matches the **iOS App-to-App Link** registered in the SmartThings Developer Console.
*   **[ ] Universal Link Interception**:
    Intercept incoming Universal Links inside your app delegate entry points.
    - If using `AppDelegate`, override `application(_:continue:restorationHandler:)` and verify that `userActivity.activityType` is equal to `NSUserActivityTypeBrowsingWeb`.
    - If using `SceneDelegate`, check `scene(_:willConnectTo:options:)` (for launch-time links) or override `scene(_:continue:)` (for runtime links).
*   **[ ] Query Parameter Parsing**:
    Extract the incoming Universal Link URL from `userActivity.webpageURL`. Parse this URL (e.g., via Swift's `URLComponents(url:resolvingAgainstBaseURL:)` API) and dynamically retrieve the standard OAuth parameters:
    - `client_id`: The client ID of your integration.
    - `redirect_uri`: The target callback URL hosted by SmartThings.
    - `response_type`: Expected token key (usually `code`).
    - `state`: Verification token to prevent CSRF.
*   **[ ] Client ID Security Check**:
    Validate the incoming `client_id` query parameter against the expected Integration Client ID. If they mismatch, abort the process immediately to prevent link hijacking.
*   **[ ] User Consent & Disclosure Screen** ⚠️:
    Before presenting the login screen or issuing any authorization code, **display a clear consent screen** informing the user that:
    - Their account will be linked to the SmartThings app.
    - SmartThings will be able to discover and control their registered devices.
    - They can revoke this access at any time via their account settings.

    The consent screen must require an **explicit user action** (e.g., tapping "Agree" / "Authorize" button). If the user declines or cancels, trigger the **failure callback** (`error=unauthorized`).
*   **[ ] Dynamic Callback Assembly**:
    Do not hardcode the callback URL. Dynamically build the final redirect target by appending parameters to the received `redirect_uri` parameter. Because Swift's `URLComponents` automatically handles URL-decoding, use it to safely build the final redirect URL.
*   **[ ] Fixed Callback Query Key ('code')**:
    Append the authorization code to the success callback strictly using the `code` query parameter key (e.g., `?code=AUTH_CODE`). Do not dynamically map other key names based on `response_type`.
*   **[ ] State Parameter Match**:
    Retrieve the exact `state` parameter value from the incoming activity, and return it exactly as-is under the query parameter key `state` in the callback.
*   **[ ] Failure Handoff Handling**:
    If authentication fails, is rejected, or is cancelled, build the redirect callback URL by appending the `error` query parameter with a descriptive error code string (e.g., `error=unauthorized`) and the exact `state` received, then redirect back to SmartThings.
*   **[ ] Universal Link Redirection via Native API**:
    Open the dynamically constructed redirection URL using Apple's native opening API:
    `UIApplication.shared.open(url, options: [UIApplication.OpenExternalURLOptionsKey.universalLinksOnly: true]) { success in ... }`
    Setting `universalLinksOnly` to `true` is critical to force iOS to route the link directly back to the native SmartThings app instead of launching Safari. Implement a completion handler to handle navigation failures gracefully.

---


## 💻 CLI Verification (Testing your Universal Link)

To test if your iOS app is configured correctly to receive the Universal Link on an iOS Simulator, run the following command in your terminal:

```bash
xcrun simctl openurl booted "https://{your-domain}/smartthings-auth?client_id={your-client-id}&response_type=code&state=xyz123&redirect_uri=https%3A%2F%2Foauth.smartthings.com"
```

*Replace `{your-domain}` and `{your-client-id}` with your actual parameters. If the setup is correct, the booted simulator will immediately launch your iOS application and pass the activity to your handler.*


