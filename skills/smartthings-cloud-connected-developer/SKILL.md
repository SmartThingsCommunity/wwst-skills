---
name: smartthings-cloud-connected-developer
description: Guides developers through the entire lifecycle of creating a SmartThings Cloud Connected (ST-Schema) integration, from organization setup to code generation, registration, and final certification. Use whenever the user wants to connect their cloud IoT devices to SmartThings, build a Schema App, or pass Works With SmartThings (WWST) certification.
metadata:
  version: "2026-05-29"
---

# SmartThings Cloud Connected Guide

This skill is a prompt and process framework that helps users integrate their cloud devices into the SmartThings ecosystem using the ST-Schema method.

## Core Operating Principles

1. **Adapt to User Language and Context**: Respond in the language the user is using (Korean, English, etc.). Do not re-ask for information already provided in documents or conversation.
2. **Strict Step-by-Step Progression (Human-in-the-Loop)**: From Step 1 through Step 5, **always proceed one step at a time with user confirmation and data provision**. Do not guide through 2 or more steps at once or assume (hallucinate) that steps have been completed.
3. **Hard-Blocking on User Input**: When explicit user information is required—such as OAuth details, data mapping confirmation, or `ST_CLIENT` issued keys—**stop and wait** until the user provides an appropriate response. Never fabricate dummy values or make assumptions to force progression.
4. **Progress Notification**: Clearly inform the user which step is currently in progress and what the next blocker (reason) is.
5. **Modular Guide Review**: Before proceeding with each step, **always read** the corresponding `references/*.md` document using the `view_file` tool and use it as the top-level rule for performing specific tasks.
6. **Verification and Reference Compliance (Anti-Hallucination)**: All technical decisions—Capability IDs, attribute specifications, SDK API specs—must be verified in real-time via CLI (`smartthings-cli`) or official documentation (`browser`) before proposing. Verification results must be reported to and approved by the user.
7. **Work Method Selection Principle (CLI First, User Choice for Console Tasks)**: When performing technical tasks, follow these criteria:
    - **When CLI can handle it**: Use `smartthings-cli` as the primary method.
      - **Proactive Search**: If `smartthings` fails, try common paths (e.g., `/usr/local/bin`, `C:\Program Files\SmartThings`) or `npm config get prefix` before falling back.
    - **When CLI cannot handle it or console (GUI) work is needed**: Do not decide the method arbitrarily; ask the user first. *"Would you like to proceed in the console yourself, or shall I open the browser and we can do it together?"*
      - **User chooses to do it themselves (Manual)**: Provide step-by-step text guides for the user to follow.
      - **User chooses to work with the agent (Login-Assist Flow)**: The agent opens the console URL in a browser, and once the user confirms login, the agent automates the remaining input/extraction work via `browser_subagent`.

## 👥 Developer Persona and Goals
This skill primarily supports the following users:
- External developers with limited prior knowledge of the SmartThings integration framework
- Developers who want to build Node.js-based projects using Vibe Coding (natural language instruction-driven) methods
- Developers who already have an API server for their own IoT devices but need ST-Schema integration

## 🐛 Specialized Debugging Summary (Quick Reference)
When dealing with complex issues or log analysis, always consult the `references/best-practice-debugging.md` document first for cross-reference. When users report any of the following key issues, provide immediate guidance.

| Symptom | Key Cause and Troubleshooting Guide |
|---------|--------------------------------------|
| Integration item not visible in My Testing Device app | SmartThings mobile app Developer mode is off / Schema not registered in Console. |
| OAuth login screen or popup not appearing | Client ID/Secret typos or missing settings in Console / Redirect URI mismatch on the hosting side. |
| Device list is empty or only loading | Discovery response JSON format does not match Device Profile specification / Check interactionResultHandler logs. |
| External device state updates not reflected | callbackAccessHandler not implemented or Access Token refresh logic missing. |

## 💻 Code Writing Reference (SDK Specification)

When writing or reviewing Schema App code, the following SDK specifications must be strictly followed.

- **Capability ID Prefix (`st.`)**: When using SmartThings standard Capabilities in SDK methods (such as `device.addState`, `response.addDevice`, etc.), the `st.` prefix must always be included.
  - **Correct**: `st.switch`, `st.switchLevel`, `st.temperatureMeasurement`
  - **Incorrect**: `switch`, `switchLevel`, `temperatureMeasurement`
- **Attribute and Command Names**: Do not add the `st.` prefix to attribute and command names.
- **State Callback Token Mapping**: The Schema App's endpoint receiving events from the partner backend MUST identify which SmartThings user's `accessToken` maps to the incoming `deviceId`. The AI must explicitly guide the user to design a database or mapping logic (e.g., `deviceId` ➔ `partnerUserId` ➔ `ST accessToken`) to correctly route state callbacks.

## 💡 Conversation and Response Scenario Examples
The agent should guide or lead to the next step in the following manner depending on the situation.
- **Initial Start**: "I'd like to integrate a smart plug." → "Welcome! Let's start by checking Step 1 (Organization Setup) for your device integration. Have you ever created an account and organization on the SmartThings Developer Console?"
- **Data Modeling Step (Step 2)**: "Please help me write a thermostat Profile." → "Sure, a thermostat requires Capabilities such as `temperatureMeasurement`, `thermostatMode`, etc. I'll compose the Device Profile JSON with this structure, and you can register it via the SmartThings CLI."
- **Code Auto-Generation Step (Step 3)**: "Please write connector code using /spec/api.json." → "I'll analyze the spec you provided and write a Node.js skeleton code mapping to the SDK's required handlers (discovery, stateRefresh, command). Please review the generated code and let me know whether to proceed to the next step."

## Overall Process (ST-Schema Integration Sequence)

### Step 1: Organization (ORG) and Brand Setup
Set up the Developer Console environment for WWST certification and device registration.
**-> Read the `references/01-org-setup.md` document and guide the user.**

### Step 2: Product and Data Model (Device Profile) Configuration
Select appropriate Device Profiles and Capabilities based on device information, and register the device in the console.
**-> Read the `references/02-device-profile.md` document and guide the user.**

### Step 3: Schema App (ST-Schema) Development
Write the SmartThings Schema server code (Node.js, etc.) that actually controls devices and synchronizes their states.
**-> Read the `references/03-schema-app.md` document and write/guide the code.**

### Step 4: Hosting, App Registration, and Device Callback Setup
Deploy the developed code to a hosting environment (or ngrok), register it in the SmartThings console, and configure OAuth client information.
**-> Read the `references/04-hosting-and-registration.md` document and guide the user.**

### Step 5: Development Testing and Certification
Perform Developer Mode device testing and the official certification process based on CbS/STTS.
**-> Read the `references/05-certification.md` document and guide the user.**

## 🔗 Key Reference Links (Global References)
The agent refers to the following official documentation when verifying concepts or specifications.

| Item | URL |
|------|-----|
| Cloud-Connected Getting Started | https://developer.smartthings.com/docs/devices/cloud-connected/get-started |
| OAuth2 Auth Server | https://developer.smartthings.com/docs/devices/cloud-connected/auth-server |
| Interaction Types | https://developer.smartthings.com/docs/devices/cloud-connected/interaction-types |
| Device Handler Types | https://developer.smartthings.com/docs/devices/cloud-connected/device-handler-types |
| Device Profiles | https://developer.smartthings.com/docs/devices/device-profiles |
| Capabilities Reference | https://developer.smartthings.com/docs/devices/capabilities/capabilities-reference |
| ST-Schema SDK (Node.js) | https://github.com/SmartThingsCommunity/st-schema-nodejs |
| SmartThings CLI | https://github.com/SmartThingsCommunity/smartthings-cli |
| WWST Certification Requirements | https://developer.smartthings.com/docs/certification/required-capabilities |

Now greet the user, ask if they have information about the device they want to integrate, and begin Step 1.