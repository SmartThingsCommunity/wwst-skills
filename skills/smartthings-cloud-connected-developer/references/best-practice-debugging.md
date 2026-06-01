# Debugging FAQ and Troubleshooting Guide

This document compiles problems that may occur during SmartThings Cloud-Connected integration development and their solutions.

## Table of Contents

1. [Development Testing Phase Issues](#development-testing-phase-issues)
2. [Discovery Issues](#discovery-issues)
3. [StateRefresh Issues](#staterefresh-issues)
4. [Command Issues](#command-issues)
5. [OAuth Issues](#oauth-issues)
6. [Callback Issues](#callback-issues)
7. [Logging and Debugging](#logging-and-debugging)
8. [Common Error Codes](#common-error-codes)

---

## Development Testing Phase Issues

### Problem 0: SmartThings CLI Command Not Found
**Cause**: PATH environment issue.
**Solution**: Check common paths (`/usr/local/bin`, `C:\Program Files\SmartThings`) or run `npm config get prefix` to find the binary and use its absolute path.

---

### Problem 1: Integration Not Visible in My Testing Device

**Symptom:**
The integration does not appear under SmartThings App → Add device → My Testing Device

**Possible Causes:**

1. **SmartThings Schema Not Registered**
   - Check Schema registration in Console
   - Check registration status via CLI: `smartthings schema`

2. **Developer Mode Disabled**
   - SmartThings App → Settings → Developer mode needs to be enabled

3. **Incorrect Region Setting**
   - Mismatch between SmartThings account region and Schema registration region

**Solutions:**

```bash
# Check Schema registration
smartthings schema

# Register Schema if not present
smartthings schema:create -i schema-config.json
```

**Enable Developer Mode:**
1. Open SmartThings App
2. Go to Settings
3. Enable Developer mode
4. Restart the app

---

### Problem 2: OAuth Login Screen Not Appearing

**Symptom:**
After selecting the integration, the OAuth login screen does not appear in the webview

**Possible Causes:**

1. **Schema Registration Information Error**
   - redirectUri mismatch
   - clientId/clientSecret mismatch

2. **OAuth Authentication Server Issue**
   - No response from authentication server
   - Incorrect OAuth URL

**Solutions:**

1. Check the webview top link
   - Verify the link is the correct OAuth URL

2. Re-check Schema information in Console
   - OAuth Authorization URL
   - OAuth Token URL
   - OAuth Client ID / Secret

3. Check OAuth server logs

**Test OAuth with curl:**

```bash
# Test Authorization URL
curl -v "https://your-oauth-server.com/oauth/authorize?client_id=YOUR_CLIENT_ID&response_type=code&redirect_uri=https://c2c-us.smartthings.com/oauth/callback"

# Test Token URL (after code is issued)
curl -X POST "https://your-oauth-server.com/oauth/token" \
  -d "grant_type=authorization_code" \
  -d "code=AUTHORIZATION_CODE" \
  -d "redirect_uri=https://c2c-us.smartthings.com/oauth/callback" \
  -u "CLIENT_ID:CLIENT_SECRET"
```

---

### Problem 3: Device List Empty After Login

**Symptom:**
OAuth login was successful but nothing is displayed in the device list

**Possible Causes:**

1. **Discovery Handler Error**
   - API call failure
   - Response format mismatch

2. **StateRefresh Handler Error**
   - State query failure

3. **Token Issue**
   - OAuth token expired
   - User identification via token failed

**Solutions:**

1. Check `interactionResultHandler` logs

```javascript
async function interactionResultHandler(request) {
  console.log('[InteractionResult] Error:', JSON.stringify(request.interactionResult, null, 2));
}
```

2. Debug Discovery handler

```javascript
async function discoveryHandler(request) {
  console.log('[Discovery] Request:', JSON.stringify(request, null, 2));

  try {
    const devices = await deviceApi.getDevices(request.authentication.token);
    console.log('[Discovery] Devices from API:', JSON.stringify(devices, null, 2));
    // ...
  } catch (error) {
    console.error('[Discovery] Error:', error);
  }
}
```

3. Check response format

```javascript
// Correct Discovery response format
{
  "headers": {
    "schema": "st-schema",
    "version": "1.0",
    "interactionType": "discoveryResponse"
  },
  "authentication": {
    "token": "user-oauth-token"
  },
  "devices": [
    {
      "externalDeviceId": "device-001",
      "friendlyName": "Living Room Light",
      "deviceHandlerType": "c2c-rgb-color-bulb" // Standard Device Handler Type OR Custom Device Profile ID
    }
  ]
}
```

---

## Discovery Issues

### Problem 4: Device Appears But Name/Icon Is Wrong

**Symptom:**
Device is displayed but the name or icon does not appear as intended

**Causes:**

1. **friendlyName not set**
2. **deviceHandlerType mismatch**

**Solutions:**

```javascript
// Include all fields in Discovery response
{
  "externalDeviceId": "device-001",
  "friendlyName": "Living Room Light",   // User-friendly name
  "deviceHandlerType": "c2c-rgb-color-bulb", // Standard Device Handler Type OR Custom Device Profile ID (UUID)
  "deviceUniqueId": "serial-12345"       // Optional
}
```

---

### Problem 5: Only Specific Devices Not Discovered

**Symptom:**
Only some devices are missing from Discovery

**Causes:**

1. **Device type mismatch**
2. **Device is offline**
3. **Excluded from API response**

**Solutions:**

1. Output full API response log
2. Check deviceHandlerType for each device
3. Check whether offline devices are being filtered

---

## StateRefresh Issues

### Problem 6: Device State Not Updated

**Symptom:**
Device state is displayed differently from actual state

**Possible Causes:**

1. **StateRefresh handler not implemented**
2. **State value format error**
3. **Capability mismatch**

**Solutions:**

1. Check StateRefresh response format

```javascript
// Correct StateRefresh response
{
  "headers": {
    "schema": "st-schema",
    "version": "1.0",
    "interactionType": "stateRefreshResponse"
  },
  "authentication": {
    "token": "user-oauth-token"
  },
  "deviceState": [
    {
      "externalDeviceId": "device-001",
      "deviceError": null,
      "states": [
        {
          "component": "main",
          "capability": "switch",
          "attribute": "switch",
          "value": "on"  // "on" or "off" (not boolean)
        },
        {
          "component": "main",
          "capability": "switchLevel",
          "attribute": "level",
          "value": 80  // number
        }
      ]
    }
  ]
}
```

2. Check state value format

| Capability | Attribute | Correct Value Format |
|------------|-----------|---------------------|
| switch | switch | `"on"`, `"off"` (string) |
| switchLevel | level | `0` - `100` (number) |
| colorControl | hue | `0` - `100` (number) |
| colorControl | saturation | `0` - `100` (number) |
| colorTemperature | colorTemperature | `2200` - `6500` (number) |
| temperatureMeasurement | temperature | Number (unit can be included) |
| lock | lock | `"locked"`, `"unlocked"` (string) |
| healthCheck | healthStatus | `"online"`, `"offline"` (string) |

---

### Problem 7: Device Continuously Shows "Checking Status"

**Symptom:**
Device continuously displays loading state

**Causes:**

1. **StateRefresh timeout**
2. **API response delay**
3. **Handler exception occurred**

**Solutions:**

1. Check timeout settings

```javascript
// axios timeout setting
const response = await axios.get(url, {
  headers: { 'Authorization': `Bearer ${token}` },
  timeout: 5000  // 5 seconds
});
```

2. Add error handling

```javascript
async function stateRefreshHandler(request) {
  try {
    // ...
  } catch (error) {
    if (error.code === 'ECONNABORTED') {
      // Timeout handling
      return {
        headers: { ... },
        authentication: { ... },
        deviceState: [
          {
            externalDeviceId: 'device-001',
            deviceError: [
              {
                errorEnum: 'DEVICE-UNAVAILABLE',
                detail: 'Request timeout'
              }
            ]
          }
        ]
      };
    }
    throw error;
  }
}
```

---

## Command Issues

### Problem 8: Command Not Executed

**Symptom:**
Command was sent from the app but the device did not respond

**Possible Causes:**

1. **Command handler not implemented**
2. **Capability/Command mismatch**
3. **API call failure**

**Solutions:**

1. Add logging to Command handler

```javascript
async function commandHandler(request) {
  console.log('[Command] Request:', JSON.stringify(request, null, 2));

  for (const device of request.devices) {
    for (const cmd of device.commands) {
      console.log(`[Command] Capability: ${cmd.capability}, Command: ${cmd.command}, Args: ${JSON.stringify(cmd.arguments)}`);
    }
  }
}
```

2. Correct Command response format

```javascript
{
  "headers": {
    "schema": "st-schema",
    "version": "1.0",
    "interactionType": "commandResponse"
  },
  "authentication": {
    "token": "user-oauth-token"
  },
  "deviceState": [
    {
      "externalDeviceId": "device-001",
      "deviceError": null,
      "states": [
        // Updated state after command execution
        {
          "component": "main",
          "capability": "switch",
          "attribute": "switch",
          "value": "on"
        }
      ]
    }
  ]
}
```

---

### Problem 9: Command Executes But State Not Updated

**Symptom:**
Device operates but state does not change in the app

**Causes:**

1. **State not included in Command response**
2. **State update missing**

**Solutions:**

```javascript
async function commandHandler(request) {
  // Execute command
  await executeCommand(...);

  // Get latest state and include in response
  const updatedState = await deviceApi.getDeviceState(authToken, deviceId);

  return {
    headers: { ... },
    authentication: { ... },
    deviceState: [{
      externalDeviceId: deviceId,
      states: formatDeviceStates(updatedState)
    }]
  };
}
```

---

## OAuth Issues

### Problem 10: Re-authentication Fails After Token Expiration

**Symptom:**
Integration disconnects or stops working after a certain period

**Causes:**

1. **Refresh Token not implemented**
2. **Token refresh logic missing**

**Solutions:**

SmartThings automatically attempts re-authentication when a token expires. Your own OAuth server must support Refresh Tokens.

```
1. SmartThings calls API with token
2. Token expired response (401)
3. SmartThings requests token refresh with Refresh Token
4. Call API again with new token
```

---

### Problem 11: Authentication Fails in Specific Regions

**Symptom:**
Integration fails only in specific regions

**Causes:**

1. **Regional OAuth servers not separated**
2. **Callback URL region mismatch**

**Solutions:**

SmartThings uses different callback URLs for each region:

| Region | Callback URL |
|--------|-------------|
| US | `https://c2c-us.smartthings.com/oauth/callback` |
| EU | `https://c2c-eu.smartthings.com/oauth/callback` |
| AP | `https://c2c-ap.smartthings.com/oauth/callback` |

Your OAuth server must allow callback URLs from all regions.

---

## Callback Issues

### Problem 12: State Change Callback Not Working

**Symptom:**
Device state changes are not reflected in the SmartThings app

**Possible Causes:**

1. **callbackAccessHandler not implemented**
2. **Callback URL not saved**
3. **ST_CLIENT_ID/SECRET not set**

**Solutions:**

1. Implement callbackAccessHandler

```javascript
async function callbackAccessHandler(request) {
  const { authentication, callbackUrls, callbackAuth } = request;

  // Save callback info to DB
  await saveCallbackInfo(authentication.token, {
    callbackUrls,
    callbackAuth
  });

  return {
    headers: {
      schema: 'st-schema',
      version: '1.0',
      interactionType: 'callbackAccessResponse'
    },
    authentication: { token: authentication.token },
    callbackUrls
  };
}
```

2. Implement callback sending code

```javascript
async function sendStateCallback(authToken, deviceStates) {
  // Retrieve saved callback info
  const callbackInfo = await getCallbackInfo(authToken);

  // Get SmartThings OAuth token
  const stToken = await getSmartThingsToken(
    callbackInfo.callbackAuth.clientId,
    callbackInfo.callbackAuth.clientSecret
  );

  // Send state callback
  await axios.post(callbackInfo.callbackUrls.state, {
    headers: {
      schema: 'st-schema',
      version: '1.0',
      interactionType: 'stateCallback'
    },
    authentication: { token: authToken },
    deviceState: deviceStates
  }, {
    headers: { 'Authorization': `Bearer ${stToken}` }
  });
}
```

---

## Logging and Debugging

### Logging Configuration

```javascript
// Log all requests/responses
async function logInteraction(type, data) {
  console.log(`[${new Date().toISOString()}] ${type}:`);
  console.log(JSON.stringify(data, null, 2));
}

// Use in handlers
async function discoveryHandler(request) {
  logInteraction('Discovery Request', request);
  // ...
  logInteraction('Discovery Response', response);
  return response;
}
```

### Local Debugging with ngrok

```bash
# Run ngrok
ngrok http 3000

# Check ngrok logs (separate terminal)
ngrok http 3000 --log stdout
```

### Request Tracing

```javascript
// Trace by request ID
async function discoveryHandler(request) {
  const requestId = request.headers.requestId;
  console.log(`[${requestId}] Discovery started`);
  // ...
  console.log(`[${requestId}] Discovery completed`);
}
```

---

## Common Error Codes

### SmartThings Error Codes

#### Global Error Enums (Examples)

| Error Enum | Description | Common Cause |
|------------|-------------|--------------|
| `BAD-REQUEST` | Bad Request | Malformed request, missing headers, or failed JSON parsing |
| `INVALID-TOKEN` | Invalid Token | Token is malformed or wrong callback URL was used |
| `TOKEN-EXPIRED` | Token Expired | The provided access token is no longer valid |
| `INTEGRATION-DELETED` | Integration Deleted | User removed the integration on the partner side |

#### Device Error Enums (Examples)

| Error Enum | Description | Common Cause |
|------------|-------------|--------------|
| `DEVICE-UNAVAILABLE` | Device Unavailable | Device is temporarily unreachable or offline |
| `DEVICE-OFFLINE` | Device Offline | Device is powered off or disconnected from network |
| `DEVICE-DELETED` | Device Deleted | Device was removed from the partner backend |
| `CAPABILITY-NOT-SUPPORTED` | Capability Not Supported | Requested capability or command is not supported by the device |
| `RESOURCE-CONSTRAINT-VIOLATION`| Resource Constraint | Requested action violates a constraint (e.g. out of bounds value) |

### HTTP Status Codes

| Code | Meaning | Response |
|------|---------|----------|
| 200 | Success | Normal processing |
| 400 | Bad Request | Check request format |
| 401 | Unauthorized | Check token |
| 403 | Forbidden | Check permissions |
| 404 | Not Found | Check device ID |
| 429 | Rate Limited | Reduce request frequency |
| 500 | Server Error | Check server logs |
| 503 | Service Unavailable | Check server status |

---

## Troubleshooting Checklist

### Discovery Issues

- [ ] Check Schema registration
- [ ] Check Developer mode enabled
- [ ] Check OAuth token validity
- [ ] Check Device API response
- [ ] Check Discovery response format

### StateRefresh Issues

- [ ] Check StateRefresh handler implementation
- [ ] Check state value format (string vs number vs boolean)
- [ ] Check Capability names
- [ ] Check API timeout settings

### Command Issues

- [ ] Check Command handler implementation
- [ ] Check Capability/Command names
- [ ] Check API call success
- [ ] Check updated state included in response

### OAuth Issues

- [ ] Check OAuth URLs
- [ ] Check Client ID/Secret
- [ ] Check Redirect URI
- [ ] Check Refresh Token support

### Callback Issues

- [ ] Check callbackAccessHandler implementation
- [ ] Check callback info saved
- [ ] Check ST_CLIENT_ID/SECRET settings
- [ ] Check callback sending logic