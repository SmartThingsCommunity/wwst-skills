# Step 5: Development Testing and Certification Guide

This step involves performing self-testing on the app/device and applying for official Works With SmartThings (WWST) certification.

## 1. Development Testing Phase (Developer Mode)
Guide the user on how to [enable Developer Mode](https://developer.smartthings.com/docs/devices/enable-developer-mode) in the SmartThings App:
1. Launch the SmartThings App.
2. Tap the *Menu* tab on the bottom navigation bar.
3. Tap the *Settings* (gear icon) in the upper right-hand corner to open *SmartThings settings*.
4. Long-press the *About SmartThings* option for 10 seconds.
5. Enable the *Developer mode* toggle at the bottom of the menu.
6. Restart the SmartThings App.

Key test scenarios and debugging FAQ to guide:

- **When the app doesn't appear under "My Testing Devices":**
  - Verify two prerequisites:
    1. **Product-Schema Mapping**: Make sure the Integration is registered at [Developer Console - Integrations](https://developer.smartthings.com/console/integrations) and all four components (Product Info, Brand, Schema App, Device Profile) are linked.
    2. **Developer Mode**: Confirm that Developer Mode is enabled in the mobile app.
- **When your own OAuth login page doesn't appear after selecting integration:**
  - Check the webview top link and recommend checking the webhook/address spelling in the registration step (04-hosting-and-registration.md)
- **When the device list doesn't appear even after successful login:**
  - It is highly likely a response format error in `discoveryRequest` and `stateRefreshRequest`. Review the code from Step 3.
  - Guide the user to check the server terminal for any error logs coming through `interactionResultHandler`

## 2. Self-Checklist Before Certification
> **[Best Practice Review Instruction]**
> Before proceeding with certification and STTS testing, be sure to review the `best-practice-stts-checklist.md` document and guide the user through the detailed checklist for each feature.

> **Agent Mandatory Obligation (Production Migration Reminder)**
> Before applying for certification, you MUST proactively remind the user to migrate their local or test environment to a production-ready state.
> *Examples of production migration items you should mention include:*
> - Hosting/Security (SSL) (e.g., replacing `ngrok` with a reliable cloud infrastructure and a valid HTTPS SSL certificate)
> - DB expansion for multi-user mapping (e.g., replacing `callbacks.json` with a scalable Database like RDBMS/NoSQL)
> - Console settings update (e.g., updating the SmartThings Developer Console with the new production Webhook URL)

Before applying for certification in earnest, ask the user whether they have confirmed the following items.
- [ ] **Production Domain Replacement Confirmed (Required)**: Addresses like `ngrok` used for local testing will always be rejected during certification review. **Make sure to replace the integration app's Webhook address with a valid single production cloud domain (AWS, GCP, own server, etc.) with an SSL certificate applied.**
- [ ] **Production DB for Callback Tokens Confirmed (Required)**: Local file storage (like `callbacks.json`) used during testing MUST be replaced with a robust Database (DynamoDB, MongoDB, RDBMS, etc.). The DB must map `deviceId` ➔ `partnerUserId` ➔ `ST accessToken` to accurately route state callbacks for multiple users.
- [ ] Regional account login: Verify that your own OAuth callback works properly even after changing the Samsung account country (US, EU, AP, etc.)
- [ ] Device list and state display confirmed: Devices are displayed normally in the app, and `online/offline` status handling is perfectly applied
- [ ] Bidirectional state synchronization: When the device state is changed in your own app, it is immediately reflected in the SmartThings app via `stateCallback`

## 3. Certification Application and Submission
Guide the user to navigate to the **'Certification' tab** on the SmartThings Developer Console's integration project detail page and click the Submit button to proceed with the STTS (SmartThings Test Suite) submission.
- [STTS Certification Test Guide](https://developer.smartthings.com/docs/certification/test-suite)

## 4. Special Certification Process: Certification by Similarity (CbS) Guide
When the user wants to register derivative models of devices with similar specifications, provide a tip about the "Certification by Similarity" policy to save time and cost.

- **What is CbS?**: A program where only one main model goes through full test certification for a family of devices with the same functionality/base (e.g., products that differ only in color/shape), and derivative models can pass certification without paying fees.
- **Required Conditions**:
  - The Brand must be the same.
  - The Device Category must be the same.
  - The Connector ID (Schema ID) must be the same.
  - The Capability list must be a subset of or exactly match the existing main model. (Emphasize that even one additionally added Capability will disqualify it as a derivative model)
- **How to Apply**: First submit the base model for full certification review and pass it, then check the CbS eligibility when submitting variant models in the Developer Console.

Once all processes are completed, offer congratulations and conclude the integration guide.