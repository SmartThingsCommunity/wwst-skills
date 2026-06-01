# Step 2: Product Configuration and Data Model (Device Profile) Preparation Guide

This step maps the physical (or virtual) device to be integrated into a logical interface model on SmartThings and registers it.

## 1. Collect Product Information

The following information is needed to register the device in the console. Ask the user if they have a document with this information organized, and if not, request them to provide it.
- **Product name**: Marketing product name displayed in the SmartThings App
- **Product number**: Unique identifier such as EAN, UPC, SKU
- **Product category**: Category used to search for the device in SmartThings
- **Product description**: Brief description of the device
- **Product image**: Image with transparent background, at least 584x584px (A strictly 1:1 aspect ratio is required; even a 1-pixel deviation will cause the upload to fail)
- **Distribution**: Launch countries/regions where the device will be serviced
- **(Optional)** Purchase link

Once the above information is collected, the integration must be registered in the SmartThings console.

> **[CLI Not Supported - Console Task Required]** Product Details registration is not currently supported by the SmartThings CLI. Ask the user about their preferred method first.
> *"Would you like to proceed in the console yourself, or shall I open the browser and we can do it together?"*

- **User chooses to do it themselves (Manual)**: Guide them to access [SmartThings Console - Device Integrations](https://developer.smartthings.com/console/integrations) and click **Create** ➔ select **Cloud Connected** to enter the information directly.
- **User chooses to work with the agent (Login-Assist)**: The agent opens [SmartThings Console - Device Integrations](https://developer.smartthings.com/console/integrations) in a browser, and once the user logs in, the agent performs the **Create** ➔ **Cloud Connected** selection and Product information input on their behalf.

---

## 2. Data Model (Device Profile) Configuration

> **[Best Practice Review Instruction]**
> Before setting up and designing device profile categories, first review the `best-practice-device-profiles.md` document to reference specifications and attribute constraints (recommended/prohibited items).

Determine the Device Type (Profile) by identifying what features the device has.

1. **Obtain Feature Information**: Ask the user exactly what actions the device performs (turn power on/off, adjust temperature, camera streaming, etc.).
2. **Explore Device Handler Types**:
   - Look for a suitable one among the [Device Handler Types](https://developer.smartthings.com/docs/devices/cloud-connected/device-handler-types) provided by default in SmartThings and recommend it.
   - **If a built-in Handler is suitable**: Simply enter the Handler Name in the Schema App's `discoveryResponse`. → No separate Device Profile creation needed.
   - **If a Custom Device Profile needs to be created**: Create one using the method below.

### Device Profile Creation Method

**[Console Task Required]**
To customize the dashboard card layout, choose precise device icons, and configure default detail states in the SmartThings app, the device profile must be created via the **SmartThings Developer Console's Device Profile Builder (Web GUI)**. Creating the profile directly via CLI is restricted in this workflow.

Ask the user about their preferred Console method first.
*"Would you like to proceed in the console yourself, or shall I open the browser and we can do it together?"*

- **User chooses to do it themselves (Manual)**: Guide them to [SmartThings Console - Integrations](https://developer.smartthings.com/console/integrations), click their integration project, select the **Device Profiles** menu, and manually build the profile.
  - Step 1: Click "Create a Device Profile" and enter the profile name.
  - Step 2: Search for the verified Capabilities and drag them into the component.
  - Step 3: Choose the category icon (e.g., Light Bulb, Plug) and configure the state/action displays.
  - Step 4: Click Save and copy the resulting **Device Profile ID (UUID)**.
- **User chooses to work with the agent (Login-Assist)**: The agent opens the console's Device Profile Builder in a browser, and once the user logs in, the agent performs Capability addition and layout configuration on their behalf.

---

## 3. Verification and Constraints When Recommending Capabilities (Core Rule - Must Not Be Skipped)

- **Agent Verification Obligation**: Before recommending a Capability, **you must perform real-time verification via CLI**. Recommending a hallucinated Capability will result in non-functional code and WWST certification failure.
  - `smartthings capabilities -s` — View the full list (Standard capabilities)
  - `smartthings capabilities <id> <version> -j` — Check detailed specifications
- **Standard Only (WWST Required)**: If the CLI result shows a **`namespace` field** or **an ID containing a period (`.`)**, it is a custom Capability and must never be recommended. (e.g., `myorg.myCapability` ❌ `switch` ✅)
- **Status and Version Check**: Capabilities in `deprecated` status will be rejected during certification. Capabilities in `proposed` status can be used for development/testing, but exercise caution as their specifications may change before final release. Cross-check the current status with the [Capabilities Reference](https://developer.smartthings.com/docs/devices/capabilities/capabilities-reference).
- **Camera and video doorbell Device Specific**: Explain that the choice between `videoStream` and `webrtc` Capabilities depends on the streaming protocol (RTSP vs WebRTC).
- [Reference: Certification Required Constraints Document](https://developer.smartthings.com/docs/certification/required-capabilities)

Once this step is complete (Profile creation and decision complete), the agent must report the device's detailed specifications along with verification data (Verification Report) and obtain user approval. After approval, inform the user that they will proceed to Step 3 (03-schema-app.md), the full code development phase.
