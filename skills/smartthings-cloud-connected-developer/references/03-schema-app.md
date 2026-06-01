# Step 3: Schema App (ST-Schema) Development Guide

This step involves writing the core logic (Schema App) that integrates the user's IoT cloud with the SmartThings cloud.

## Integration Architecture Lifecycle (Important Context)
Before writing code and providing guidance, the agent must understand the following event call sequence and construct defensive logic:
1. **OAuth Authentication**: The user logs in through the app to link their account.
2. **`discovery` ➔ `stateRefresh`**: Immediately after linking, the SmartThings cloud requests the device list (`discoveryRequest`), followed by a request for initial state values (`stateRefreshRequest`).
3. **`command`**: When the user taps a switch in the app, a control request (`commandRequest`) is sent.
4. **`callbackAccess` Issuance**: During initial linking, the `grantCallbackAccess` interaction provides token issuance authority from SmartThings to us. The connector uses this authority to obtain the `accessToken` that will be used for future push communications.
5. **`stateCallback` (Proactive)**: When the user operates the device via a physical button or their own external app, the server proactively pushes the changed state to the SmartThings app using the `accessToken` obtained in step 4.
   > **[Architectural Consideration for AI]**: The Schema App's endpoint receiving events from the partner backend MUST be able to distinguish which SmartThings user's `accessToken` corresponds to the incoming event. You must explicitly remind the user to design a mapping logic (e.g., `deviceId` ➔ `partnerUserId` ➔ `ST accessToken`) to correctly route state callbacks without mixing up users.

## 🏗 Default Schema App Architecture to Be Generated
The structure and required handlers of the Node.js server that will be created through this skill's guidance are shown below. Always keep this structure in mind when generating code and providing template guidance.
- **`src/index.js`**: Express server entry point and environment variable configuration
- **`src/connector.js`**: `st-schema` SDK instance configuration
- **`src/handlers/`**
  - `discovery.js`: Device list and profile return handler
  - `state-refresh.js`: Current device state return handler
  - `command.js`: Control command processing handler
  - `callback-access.js`: Token storage for asynchronous state synchronization (`st-schema` callback)
  - `integration-deleted.js`: Integration removal (deletion) handling

## 1. Development Environment Setup
> **[Best Practice Review Instruction]**
> - When writing handlers and code, **always** review `best-practice-schema-app-code.md` and use the provided template code and snippets.
> - When mapping your own API specs to SmartThings attributes, review `best-practice-api-mapping.md` first to understand the standard mapping rules.

- Ask the user about their preferred environment, such as Node.js. Recommend Node.js and guide development based on the `st-schema` SDK.
- (Reference link: [ST-Schema NodeJS SDK GitHub](https://github.com/SmartThingsCommunity/st-schema-nodejs))

## 2. Required Interaction Types Handler Implementation
The Schema App must handle various types of requests (Interaction Types) coming from SmartThings. Ask if the user has their own Device API specification document.
- **If a specification exists**: Write code based on that specification, following the [Agent Mandatory Obligation] below.
- **If the specification is unclear or unavailable**: If the user wants to test or prototype, first generate a **Mock-Up (dummy) Schema App code** that uses memory data or arbitrary dummy variables instead of the actual API, so they can test the overall structure.

> **Agent Mandatory Obligation (Data Transformation Map Verification - Required When Specification Is Available)**
> If an actual API specification is provided, do not generate code immediately. **First, output a "data transformation mapping table" (e.g., `1/0` ➔ `on/off`) between the user's device attributes and SmartThings Capabilities in markdown table format, obtain user confirmation (approval), and then** write the actual control logic. **This mapping table must include official IDs and attribute names verified through verification tools (CLI preferred for speed).** This fundamentally eliminates runtime value mismatch errors.

The following handlers are mandatory and must be implemented.

1. **`discoveryHandler` (Discovery Request)**
   - Called by SmartThings during initial setup or on a 24-hour cycle.
   - Retrieves the device list via the manufacturer's cloud API and responds in `DiscoveryResponse` format.
   - Emphasize that the Profile ID (Device Handler Type Name) must be accurately mapped.

2. **`stateRefreshHandler` (State Refresh Request)**
   - Called when the SmartThings app requests the current state of a device.
   - Queries the device state using the manufacturer's cloud API and responds with `StateRefreshResponse`.

3. **`commandHandler` (Command Request)**
   - Called when the user issues a command such as turning a device switch on or off via the SmartThings app or automation flow.
   - Calls the manufacturer's cloud API to send the actual control command to the device and responds with `CommandResponse`.

## 3. Supplementary/Debugging Handler Implementation
Guide the implementation of the following items for long-term stable integration.

1. **`callbackAccessHandler`**
   - A handler that handles token exchange, where the user's cloud receives and stores `callbackUrls` and `accessToken` information for pushing information to SmartThings (e.g., stateCallback).

   > [!IMPORTANT]
   > **Must Return After Step 4 Registration Is Complete**
   > At this stage, leave it as a Placeholder and proceed. `ST_CLIENT_ID` and `ST_CLIENT_SECRET` are only issued during the **Step 4 (Schema App Registration)** process. Once app registration is complete, you must inject these keys into the `SchemaConnector` initialization (`.clientId()` and `.clientSecret()`) for asynchronous state synchronization to work.
   > **Note on Testing:** `grantCallbackAccess` interactions cannot be successfully mocked with dummy data locally. The SDK strictly validates the callback tokens against the real SmartThings API. Developers MUST perform a real device installation via the SmartThings App or Developer Workspace to receive valid tokens and avoid `400/401` errors during testing.

2. **`integrationDeletedHandler`**
   - Called when the user removes the integration from their SmartThings account.
   - Guide the user to perform database deletion or invalidation of the integration token on their own API.

3. **`interactionResultHandler`**
   - Receives detailed reasons when errors (Invalid format, etc.) occur during console or server testing.
   - Guide implementation to output logs (e.g., `console.log()`) by default to assist with debugging.

Once code scaffolding is complete and the user considers development finished, inform them that they will proceed to Step 4 (04-hosting-and-registration.md) to host the code on an actual server or ngrok and establish a communication path with SmartThings.