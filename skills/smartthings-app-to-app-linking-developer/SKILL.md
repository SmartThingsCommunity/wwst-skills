---
name: smartthings-app-to-app-linking-developer
description: Guides developers through setting up and implementing SmartThings App-to-App Account Linking for Android and iOS mobile devices, specifically for Cloud Connected (ST-Schema) integrations. Enforces dynamic lookup of the official documentation as the source of truth.
metadata:
  version: "2026-06-09"
---

# SmartThings App-to-App Account Linking Guide

This skill helps developers implement seamless App-to-App Account Linking between their native mobile applications (Android/iOS) and the SmartThings app.

> [!IMPORTANT]
> - App-to-App Account Linking is supported exclusively for **Cloud Connected (ST-Schema)** integrations.
> - **Prerequisite**: This is **optional, but recommended when applicable** — if the developer's native Android/iOS app can already authenticate the user (i.e., it can substitute for the browser-based OAuth login screen), implementing this is recommended for the smoother UX/higher onboarding completion it gives users. It requires a Schema App to already be registered in the SmartThings Developer Console (created via the **`smartthings-cloud-connected-developer`** skill's Step 4, `04-hosting-and-registration.md`). If the user has not registered a Schema App yet, direct them to that skill first and return here afterward.
> - **Source of Truth Rule**: The AI agent **MUST** fetch and read the live content of `https://developer.smartthings.com/docs/devices/cloud-connected/app-to-app-linking` at the beginning of the task to retrieve the absolute source-of-truth code blocks, intent parsing logic, and regional redirect URIs.

## Core Operating Principles

1. **Source of Truth Dynamic Lookup**: Always fetch `https://developer.smartthings.com/docs/devices/cloud-connected/app-to-app-linking` to read the official Kotlin/Swift code implementations, query parameter names, and callback domains. Do not output stale code or static assumptions; instead, inspect the official documentation dynamically during execution.
2. **Language Adaptation**: Respond in the developer's language of choice (e.g., Korean, English) and reuse existing context.
3. **Phase-by-Phase Progression**: App-to-App linking requires both environment configuration and code changes. Progress step-by-step with the user, ensuring domain configurations are set before code is written.
4. **Dynamic Callback Enforcement**: Always instruct the developer to parse parameters (`client_id`, `state`, `redirect_uri`, `response_type`) from incoming intents dynamically. Do not allow hardcoding of callback URLs, as SmartThings hosts regional domains. When returning the authorization code, the query parameter key must be strictly fixed to `code`.
5. **Certified/Published App Rule**: If the Schema App is already certified/published, the developer must contact SmartThings support/WWST team to apply the App-Link/Universal Link rather than modifying it directly.
6. **Merge Digital Asset Files**: When generating `assetlinks.json` or `apple-app-site-association`, do not overwrite existing files. Read the existing file first, parse its JSON, and append/merge the new SmartThings association block to preserve other integrations (e.g., Google/Apple login or other deep links).
7. **User Consent & Disclosure Enforcement & Checklists**: Focus on guiding the implementation using detailed checklists (Android/iOS Phase 2 & 3 guidelines) rather than copy-pasting local code templates. Before issuing an authorization code, the partner app **MUST** display a clear consent screen explaining that the user's devices will be linked to and controllable by the SmartThings app. Explicit user action is required, and the cancel path must map to `error=unauthorized`.
8. **Vibe Coding Compliance & Spec Adherence**: When generating or writing integration codes dynamically (often termed "Vibe Coding" or AI code generation), the AI assistant and developer must strictly adhere to the technical specifications defined in the Phase 2 & 3 checklists. Do not omit critical security and integration details (such as `client_id` validation, `state` parameter consistency, returning the authorization code strictly via the `code` query parameter key, and `universalLinksOnly` options) for the sake of simplified code output. Every checkpoint in the reference checklists must be rigorously met to prevent runtime failures and comply with WWST certification standards.

## Persona and Goals
This skill supports:
- Native mobile developers (Kotlin, Swift) integrating their smart home application with the SmartThings ecosystem.
- Developers looking to improve user onboarding completion rates by avoiding browser-based OAuth flows.

> [!IMPORTANT]
> **UX Responsibility**: App-to-App Account Linking transfers authority over the user's physical devices to SmartThings. The partner app is responsible for presenting a proper consent UI to the user before completing the OAuth flow. The AI assistant must remind developers of this UX obligation at the implementation phase.

## Specialized Debugging Summary (Quick Reference)

| Symptom | Cause | Resolution |
| :--- | :--- | :--- |
| SmartThings app falls back to browser OAuth | Native app not installed, or OS verification of App Links/Universal Links failed. | Check `assetlinks.json` or `apple-app-site-association` hosting and format. |
| Account linking completes but state is missing | Partner app did not return the exact `state` token received from SmartThings. | Ensure incoming `state` param is saved and appended to callback. |
| Redirect back to SmartThings fails | Invalid or hardcoded callback URL. | Decode the incoming `redirect_uri` param dynamically using UTF-8. |

## Code Writing Reference (Deep Link Parameters)

Refer to the official documentation for parsing logic. Expect these query parameters from the incoming link:
*   `client_id` (String): Validate this against your known Client ID for security.
*   `response_type` (String): Fixed to `code`. The parameter key name you must use when returning your authorization code is strictly fixed to `code`.
*   `state` (String): Session identifier. Must be passed back exactly as-is.
*   `redirect_uri` (String): Encoded SmartThings callback URL. Decode using UTF-8 before appending parameters.

## Overall Process (App-to-App Linking Sequence)

### Phase 1: Environment Setup and Domain Verification
Establish secure association between your app domain and the mobile app binary.
**➔ Read `references/01-environment-setup.md` to guide the user.**

### Phase 2: Android App Links Implementation
Configure your Android app Manifest, register intent filters, parse incoming params, and trigger login.
**➔ Read `references/02-android-implementation.md` to write/guide the code.**

### Phase 3: iOS Universal Links Implementation
Configure iOS Associated Domains, handle user activities in Swift, and redirect back.
**➔ Read `references/03-ios-implementation.md` to write/guide the code.**

### Phase 4: SmartThings Console Registration & Testing
After code implementation, the developer **must** register the App-to-App Link URLs in the SmartThings Developer Console **before** any device testing.

**Console navigation path**:
[SmartThings Developer Console](https://developer.smartthings.com/console) → **Device Integrations** → **Schema Apps** tab → Select your Schema App → **App-to-App Linking (optional)** section

> The AI assistant **must walk the developer through this console step** after Phase 2/3 code implementation is complete, and **before** instructing them to run device tests. Do not skip directly to testing.

**➔ Read `references/04-registration-and-testing.md` to guide the user.**

---

## Key Reference Links (Global References)

*   Official App-to-App Linking Guide: `https://developer.smartthings.com/docs/devices/cloud-connected/app-to-app-linking` (Source of Truth)
*   **Related skill (prerequisite)**: `smartthings-cloud-connected-developer` — use this first to create/register the Schema App (OAuth client, `connector.json`, Console registration) that App-to-App Linking attaches to.
