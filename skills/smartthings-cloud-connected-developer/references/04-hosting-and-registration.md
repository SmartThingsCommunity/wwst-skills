# Step 4: Hosting and Schema App Registration Guide

This step involves setting up a deployment environment so that the developed Schema App can actually communicate with SmartThings, and registering it in the SmartThings Developer Center.

## 1. Hosting and Webhook URL Preparation

Ask if the goal is local testing, and if so, guide them to generate an HTTPS-accessible Webhook URL using `ngrok` or similar tools.

```bash
# Run local server and generate public URL with ngrok
node src/index.js &
ngrok http 3000

# Check the generated URL via CLI (ngrok API)
curl -s http://localhost:4040/api/tunnels | python3 -m json.tool
```

> [!IMPORTANT]
> **[Production Hosting Precautions]**
> If the user is deploying for production rather than local testing, ensure they are aware of the following:
> - **AWS Lambda**: You must add permissions for the SmartThings principal ID (`148790070172`) via AWS CLI so SmartThings can invoke the function.
> - **Webhook**: Must be served over HTTPS with a valid public certificate.
> - Detailed instructions can be found in the [Schema App Hosting & Registration Guide](https://developer.smartthings.com/docs/devices/cloud-connected/schema-app).

---

## 2. SmartThings Schema App Registration

Combine the Webhook URL, your own OAuth authentication information, and the Product information from Step 2 to register.

### Schema App Registration Method

**[First Choice] CLI (Recommended)**

The agent collects the following information, writes `connector.json`, and registers it via CLI.

Required information:
- `webhookUrl`: Webhook URL secured via ngrok etc.
- `oAuthAuthorizationUrl`: Your own OAuth Authorization endpoint
- `oAuthTokenUrl`: Your own OAuth Token endpoint
- `oAuthClientId`: Your own OAuth Client ID
- `oAuthClientSecret`: Your own OAuth Client Secret
- `oAuthScope`: OAuth Scope to request (e.g., `device:read device:write`)
- `appName`: Schema app display name
- `partnerName`: Company name
- `userEmail`: Developer email

```bash
# Create Schema App based on connector.json file
smartthings schema:create -i connector.json

# Query ST Client credentials after creation
smartthings schema
smartthings schema <appId>
```

> **connector.json Example**
> ```json
> {
>   "appName": "My IoT Platform",
>   "partnerName": "My Company",
>   "hostingType": "webhook",
>   "userEmail": "developer@mycompany.com",
>   "webhookUrl": "https://xxxx.ngrok-free.app",
>   "oAuthAuthorizationUrl": "https://myplatform.example.com/oauth/authorize",
>   "oAuthTokenUrl": "https://myplatform.example.com/oauth/token",
>   "oAuthClientId": "my-client-id",
>   "oAuthClientSecret": "my-client-secret",
>   "oAuthScope": "device:read device:write"
> }
> ```

**[CLI Not Supported or Console Preferred]** Ask the user about their preferred method first.
*"Would you like to proceed in the console yourself, or shall I open the browser and we can do it together?"*

- **User chooses to do it themselves (Manual)**: Guide them to [SmartThings Console - Integrations](https://developer.smartthings.com/console/integrations), select the in-progress integration, and manually enter the Webhook and OAuth information.
- **User chooses to work with the agent (Login-Assist)**: The agent opens [SmartThings Console - Integrations](https://developer.smartthings.com/console/integrations) in a browser, and once the user logs in, the agent performs the OAuth field input and key extraction on their behalf.

---

> **Agent Mandatory Notice (Distinguishing Two OAuth Keys)**
> Clearly explain the "bidirectional authentication key" concept that developers most commonly confuse.
> 1. The **OAuth settings just entered (your own server keys prepared in advance)**: Used when SmartThings logs into your server.
> 2. The newly issued **`ST_CLIENT_ID` and `ST_CLIENT_SECRET`** after registration: Used when your server proactively sends state callbacks to SmartThings.
> Make sure the user understands these two are completely different, and instruct them to safely copy the newly issued 'ST_CLIENT' key pair.

---

## 3. Product Registration and Schema-Profile Mapping (Console Task)

> [!IMPORTANT]
> **[Prerequisite for Test Device Visibility]**
> Simply registering a Schema App via CLI or Console will not make it appear under the mobile app's "My Testing Devices" list. You must register the final product (Integration) in the **Integrations** menu and map all components.
> 
> Guide the user to complete the following 4-element mapping steps:
> 1. Navigate to the [SmartThings Console - Integrations](https://developer.smartthings.com/console/integrations).
> 2. In the **Device Integrations** menu, click **Create** ➔ Select **Cloud Connected**.
> 3. Fill in the **Product Overview** fields under the **Product Details** menu:
>    - **Product name** (e.g., `Lumos Smart Color Bulb`)
>    - **Model number** (e.g., unique model identifier or SKU)
>    - **Product category** (Select the appropriate category, e.g., `Light`)
>    - **Product description** (Brief description up to 500 characters)
>    - **Product image** (A transparent PNG, minimum 584x584 pixels)
> 4. Select the registered **Brand** (created in Step 1).
> 5. Select the registered **Schema App** (created in Step 4).
> 6. Map the corresponding **Device Profile** (created in Step 2).
> 7. Click **Save** to finalize. Without this mapping, the SmartThings platform will not recognize your test setup, and it will not appear under "My Testing Devices".

---

## 4. Device Callback (Proactive Callback) Code Update

The issued `ST_CLIENT_ID` and `ST_CLIENT_SECRET` must be injected into the code written in Step 3 for complete device state tracking.

1. **Inject Keys into Connector Initialization Code**:
   When using the `st-schema` Node library, inject `clientId` and `clientSecret` as environment variables.
   (Reference: [Proactive State Callbacks Github](https://github.com/SmartThingsCommunity/st-schema-nodejs/tree/master?tab=readme-ov-file#proactive-state-callbacks))

2. **Activate `callbackAccessHandler`**:
   To enable the logic for proactively sending device state changes to SmartThings (stateCallback) when the device state changes on the server side (e.g., when a user turns off a light in their own app), or pinging SmartThings to discover a device (discoveryCallback) when a new device is manually added, explain the usage syntax of the corresponding handler.

   > [!TIP]
> Now go back to the **[callbackAccessHandler Implementation Section](./03-schema-app.md#3-supplementarydebugging-handler-implementation)** from Step 3 and guide the user to complete the actual logic using the issued keys.

> **Agent Mandatory Obligation (Local Debugging Support)**
> Do not stop at just explaining the code settings. **Be sure to generate and provide a `curl`-based test snippet** so the user can test whether the callback (stateCallback) works correctly in a terminal environment.

Once all the above items are reflected in the code and the server is running, inform the user that they will proceed to Step 5 (05-certification.md) to test whether the integration actually works.