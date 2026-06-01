This document explains the standard interface and implementation best practices when developing a Node.js-based Schema App using the **SmartThings Schema SDK (st-schema)**.

> [!TIP]
> The ST-Schema SDK provides abstracted handler signatures and builder patterns for each SmartThings Interaction Type (Discovery, State, Command, etc.). Using SDK methods instead of constructing HTTP headers and response bodies directly helps prevent specification errors and makes maintenance easier.

## Table of Contents

1. [Project Structure](#project-structure)
2. [Package Installation](#package-installation)
3. [Code Templates](#code-templates)
4. [Handler Implementation](#handler-implementation)
5. [Own Device API Integration](#own-device-api-integration)
6. [Environment Variable Configuration](#environment-variable-configuration)
7. [Local Testing](#local-testing)
8. [Deployment Guide](#deployment-guide)
9. [Code Writing and Review Principles](#code-writing-and-review-principles)

---

## Code Writing and Review Principles

When generating or reviewing code using the ST-Schema SDK, the following principles must be strictly observed.

### 1. Capability ID Prefix Usage (st.)
When adding states or processing commands via SDK methods (`addState`, `addDevice`, etc.), all **standard Capability IDs (SmartThings Official Capabilities) must include the `st.` prefix.**

- **Correct**: `st.switch`, `st.switchLevel`, `st.temperatureMeasurement`
- **Incorrect**: `switch`, `switchLevel`, `temperatureMeasurement` (In this case, the SDK may not recognize them or may treat them as custom Capabilities)

### 2. Attribute and Command Names
Attribute names and Command names use standard naming without the `st.` prefix.
- **Example**: capability: `st.switch`, attribute: `switch`, command: `on`/`off`

### 3. Interaction Result Verification
During development, `interactionResultHandler` must be implemented to monitor specification errors (Invalid format, etc.) delivered from the SmartThings app in real-time.

---

## Project Structure

```
schema-app/
├── package.json              # Project configuration
├── src/
│   ├── index.js              # Express server entry point
│   ├── connector.js          # ST-Schema connector instance
│   ├── handlers/
│   │   ├── discovery.js      # Discovery handler
│   │   ├── state-refresh.js  # StateRefresh handler
│   │   ├── command.js        # Command handler
│   │   ├── callback-access.js    # CallbackAccess handler
│   │   ├── integration-deleted.js # IntegrationDeleted handler
│   │   └── interaction-result.js # InteractionResult handler (optional)
│   └── api/
│       └── device-api.js     # Own Device API client
├── .env.example              # Environment variable template
├── .gitignore
└── README.md
```

---

## Package Installation

### package.json

```json
{
  "name": "smartthings-schema-app",
  "version": "1.0.0",
  "description": "SmartThings Cloud Connected Schema App",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "jest"
  },
  "dependencies": {
    "st-schema": "^1.5.1",
    "uuid": "^9.0.1",
    "express": "^4.18.2",
    "axios": "^1.6.0",
    "dotenv": "^16.3.1",
    "body-parser": "^1.20.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.7.0"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### Installation Command

```bash
npm init -y
npm install st-schema express axios dotenv body-parser
npm install -D nodemon jest
```

---

## Code Templates

### src/index.js (Express Server Entry Point)

```javascript
require('dotenv').config();
const express = require('express');
const bodyParser = require('body-parser');
const { connector } = require('./connector');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(bodyParser.json());

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok', timestamp: new Date().toISOString() });
});

// SmartThings ST-Schema endpoint
app.post('/', (req, res) => {
  connector.handleHttpCallback(req, res);
});

// Start server
app.listen(PORT, () => {
  console.log(`Schema App server running on port ${PORT}`);
  console.log(`Health check: http://localhost:${PORT}/health`);
  console.log(`ST-Schema endpoint: http://localhost:${PORT}/`);
});
```

### src/connector.js (ST-Schema Connector Instance)

```javascript
const { SchemaConnector } = require('st-schema');
const discoveryHandler = require('./handlers/discovery');
const stateRefreshHandler = require('./handlers/state-refresh');
const commandHandler = require('./handlers/command');
const callbackAccessHandler = require('./handlers/callback-access');
const integrationDeletedHandler = require('./handlers/integration-deleted');
const interactionResultHandler = require('./handlers/interaction-result');

const connector = new SchemaConnector()
  .clientId(process.env.ST_CLIENT_ID)
  .clientSecret(process.env.ST_CLIENT_SECRET)
  .discoveryHandler(discoveryHandler)
  .stateRefreshHandler(stateRefreshHandler)
  .commandHandler(commandHandler)
  .callbackAccessHandler(callbackAccessHandler)
  .integrationDeletedHandler(integrationDeletedHandler)
  .interactionResultHandler(interactionResultHandler);

module.exports = { connector };
```

---

## Handler Implementation

### src/handlers/discovery.js (Discovery Handler)

```javascript
const deviceApi = require('../api/device-api');

/**
 * Discovery Handler
 * Called when SmartThings requests the device list
 */
async function discoveryHandler(accessToken, response) {
  console.log('[Discovery] Request received for token:', accessToken.substring(0, 10) + '...');

  try {
    const devices = await deviceApi.getDevices(accessToken);

    for (const device of devices) {
      const stDevice = response.addDevice(
        device.id,                    // externalDeviceId
        device.name,                  // friendlyName
        device.deviceHandlerType      // deviceHandlerType: Standard Device Handler Type (e.g., 'c2c-switch') OR Custom Device Profile ID (UUID)
      );

      stDevice.manufacturerName(process.env.MANUFACTURER_NAME || 'MyCompany')
        .modelName(device.modelName || 'SmartDevice-V1')
        .deviceUniqueId(device.serialNumber || device.id);

      if (device.roomName) {
        stDevice.roomName(device.roomName);
      }
    }

    console.log(`[Discovery] Success: ${devices.length} devices found.`);
  } catch (error) {
    console.error('[Discovery] Error:', error.message);
    response.setError(error.message, 'INTEGRATION-OFFLINE');
  }
}

module.exports = discoveryHandler;
```

### src/handlers/state-refresh.js (StateRefresh Handler)

```javascript
const deviceApi = require('../api/device-api');

/**
 * StateRefresh Handler
 * Called when SmartThings requests device state
 */
async function stateRefreshHandler(accessToken, response, data) {
  console.log('[StateRefresh] Request received for devices:', data.devices.map(d => d.externalDeviceId));

  try {
    for (const requestedDevice of data.devices) {
      const externalDeviceId = requestedDevice.externalDeviceId;
      try {
        const deviceState = await deviceApi.getDeviceState(accessToken, externalDeviceId);
        addStatesToResponse(response, externalDeviceId, deviceState);
      } catch (deviceError) {
        console.error(`[StateRefresh] Error for device ${externalDeviceId}:`, deviceError.message);
        response.addDevice(externalDeviceId).setError(deviceError.message, 'DEVICE-OFFLINE');
      }
    }
    console.log('[StateRefresh] Success');
  } catch (error) {
    console.error('[StateRefresh] Global Error:', error.message);
    response.setError(error.message, 'INTEGRATION-OFFLINE');
  }
}

function addStatesToResponse(response, externalDeviceId, deviceState) {
  const device = response.addDevice(externalDeviceId);
  const component = device.addComponent('main');

  if (deviceState.power !== undefined) {
    component.addState('st.switch', 'switch', deviceState.power ? 'on' : 'off');
  }
  if (deviceState.level !== undefined) {
    component.addState('st.switchLevel', 'level', deviceState.level);
  }
  if (deviceState.temperature !== undefined) {
    component.addState('st.temperatureMeasurement', 'temperature', deviceState.temperature, 'C');
  }
  if (deviceState.humidity !== undefined) {
    component.addState('st.relativeHumidityMeasurement', 'humidity', deviceState.humidity, '%');
  }
  component.addState('st.healthCheck', 'healthStatus', deviceState.online ? 'online' : 'offline');
}

module.exports = stateRefreshHandler;
```

### src/handlers/command.js (Command Handler)

```javascript
const deviceApi = require('../api/device-api');

/**
 * Command Handler
 * Called when SmartThings sends a device control command
 */
async function commandHandler(accessToken, response, devices, data) {
  console.log('[Command] Request received for devices:', devices.map(d => d.externalDeviceId));

  try {
    for (const requestedDevice of devices) {
      const externalDeviceId = requestedDevice.externalDeviceId;
      const commands = requestedDevice.commands;

      try {
        for (const command of commands) {
          const { capability, command: cmd, arguments: args } = command;
          console.log(`[Command] Executing: ${cmd} on device ${externalDeviceId}`);
          await executeCommand(accessToken, externalDeviceId, capability, cmd, args);
        }
        const deviceState = await deviceApi.getDeviceState(accessToken, externalDeviceId);
        addStatesToResponse(response, externalDeviceId, deviceState);
      } catch (deviceError) {
        console.error(`[Command] Device Error for ${externalDeviceId}:`, deviceError.message);
        response.addDevice(externalDeviceId).setError(deviceError.message, 'DEVICE-OFFLINE');
      }
    }
    console.log('[Command] Success');
  } catch (error) {
    console.error('[Command] Global Error:', error.message);
    response.setError(error.message, 'INTEGRATION-OFFLINE');
  }
}

async function executeCommand(authToken, deviceId, capability, cmd, args) {
  switch (capability) {
    case 'st.switch':
      return deviceApi.setPower(authToken, deviceId, cmd === 'on');
    case 'st.switchLevel':
      if (cmd === 'setLevel') return deviceApi.setLevel(authToken, deviceId, args[0]);
      break;
    case 'st.colorControl':
      if (cmd === 'setColor') return deviceApi.setColor(authToken, deviceId, args[0]);
      if (cmd === 'setHue') return deviceApi.setHue(authToken, deviceId, args[0]);
      if (cmd === 'setSaturation') return deviceApi.setSaturation(authToken, deviceId, args[0]);
      break;
    case 'st.colorTemperature':
      if (cmd === 'setColorTemperature') return deviceApi.setColorTemperature(authToken, deviceId, args[0]);
      break;
    case 'st.thermostatMode':
      if (cmd === 'setThermostatMode') return deviceApi.setThermostatMode(authToken, deviceId, args[0]);
      break;
    case 'st.thermostatCoolingSetpoint':
      if (cmd === 'setCoolingSetpoint') return deviceApi.setCoolingSetpoint(authToken, deviceId, args[0]);
      break;
    case 'st.thermostatHeatingSetpoint':
      if (cmd === 'setHeatingSetpoint') return deviceApi.setHeatingSetpoint(authToken, deviceId, args[0]);
      break;
    case 'st.lock':
      return deviceApi.setLock(authToken, deviceId, cmd === 'lock' ? 'locked' : 'unlocked');
    case 'st.doorControl':
    case 'st.garageDoorControl':
      return deviceApi.setDoorState(authToken, deviceId, cmd === 'open' ? 'open' : 'closed');
    case 'st.windowShade':
      if (cmd === 'open') return deviceApi.setWindowShade(authToken, deviceId, 'open');
      if (cmd === 'close') return deviceApi.setWindowShade(authToken, deviceId, 'closed');
      if (cmd === 'pause') return deviceApi.setWindowShade(authToken, deviceId, 'partially_opened');
      break;
    case 'st.windowShadeLevel':
      if (cmd === 'setShadeLevel') return deviceApi.setShadeLevel(authToken, deviceId, args[0]);
      break;
    case 'st.refresh':
      return Promise.resolve();
    default:
      throw new Error(`Unsupported capability: ${capability}`);
  }
}

module.exports = commandHandler;
```

### src/handlers/callback-access.js (CallbackAccess Handler)

```javascript
/**
 * CallbackAccess Handler
 * Called when SmartThings delivers callback URLs and tokens
 * Used to send push notifications to SmartThings when device state changes
 */
async function callbackAccessHandler(accessToken, callbackAuthentication, callbackUrls, body) {
  console.log('[CallbackAccess] Request received');

  try {
    const callbackInfo = {
      userAccessToken: accessToken,
      callbackUrls,
      callbackAuth: callbackAuthentication,
      updatedAt: new Date().toISOString()
    };

    // IMPORTANT: You must save these tokens (callbackUrls, callbackAuth) to send proactive state updates later.
    // They should be stored uniquely per user (e.g., keyed by userAccessToken).

    // For local testing, you can save them to a file:
    // const fs = require('fs');
    // let callbacksDB = {};
    // if (fs.existsSync('callbacks.json')) {
    //   callbacksDB = JSON.parse(fs.readFileSync('callbacks.json', 'utf8'));
    // }
    // callbacksDB[accessToken] = callbackInfo;
    // fs.writeFileSync('callbacks.json', JSON.stringify(callbacksDB, null, 2));

    // For production: Save to DB
    // await db.saveCallbackInfo(accessToken, callbackInfo);

    console.log('[CallbackAccess] Callback info stored for proactive updates');
  } catch (error) {
    console.error('[CallbackAccess] Error:', error.message);
  }
}

module.exports = callbackAccessHandler;
```

### src/handlers/integration-deleted.js (IntegrationDeleted Handler)

```javascript
/**
 * IntegrationDeleted Handler
 * Called when the user removes the integration from the SmartThings App
 */
async function integrationDeletedHandler(accessToken, response) {
  console.log('[IntegrationDeleted] Request received');

  try {
    // TODO: Clean up integration-related data
    // await db.removeUser(accessToken);
    console.log('[IntegrationDeleted] Cleanup complete');
  } catch (error) {
    console.error('[IntegrationDeleted] Error during cleanup:', error.message);
  }
}

module.exports = integrationDeletedHandler;
```

### src/handlers/interaction-result.js (InteractionResult Handler)

```javascript
/**
 * InteractionResult Handler
 * Called when SmartThings notifies the result of a request processing
 * Used for debugging purposes
 */
async function interactionResultHandler(accessToken, response, data) {
  console.log('[InteractionResult] Request received');

  const { interactionResult } = data;

  if (interactionResult?.errorState) {
    console.error('[InteractionResult] Error state:', interactionResult.errorState);
    console.error('[InteractionResult] Error reason:', interactionResult.errorReason);
    console.error('[InteractionResult] Description:', interactionResult.errorDescription);
  } else {
    console.log('[InteractionResult] Success');
  }
}

module.exports = interactionResultHandler;
```

---

## Own Device API Integration

### src/api/device-api.js

```javascript
const axios = require('axios');

const DEVICE_API_BASE_URL = process.env.DEVICE_API_URL || 'https://api.yourcompany.com/v1';

const deviceApi = {
  async getDevices(authToken) {
    try {
      const response = await axios.get(`${DEVICE_API_BASE_URL}/devices`, {
        headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' }
      });
      return response.data.devices || response.data;
    } catch (error) {
      console.error('[DeviceAPI] getDevices error:', error.message);
      throw new Error('Failed to get device list');
    }
  },

  async getDeviceState(authToken, deviceId) {
    try {
      const response = await axios.get(`${DEVICE_API_BASE_URL}/devices/${deviceId}/state`, {
        headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' }
      });
      return response.data;
    } catch (error) {
      console.error('[DeviceAPI] getDeviceState error:', error.message);
      throw new Error(`Failed to get device state: ${deviceId}`);
    }
  },

  async setPower(authToken, deviceId, power) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/power`, {
        power: power ? 'on' : 'off'
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setPower error:', error.message);
      throw new Error(`Failed to set power: ${deviceId}`);
    }
  },

  async setLevel(authToken, deviceId, level) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/level`, {
        level: level
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setLevel error:', error.message);
      throw new Error(`Failed to set level: ${deviceId}`);
    }
  },

  async setColor(authToken, deviceId, color) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/color`, {
        hue: color.hue, saturation: color.saturation
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setColor error:', error.message);
      throw new Error(`Failed to set color: ${deviceId}`);
    }
  },

  async setColorTemperature(authToken, deviceId, colorTemperature) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/color-temperature`, {
        colorTemperature: colorTemperature
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setColorTemperature error:', error.message);
      throw new Error(`Failed to set color temperature: ${deviceId}`);
    }
  },

  async setThermostatMode(authToken, deviceId, mode) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/thermostat/mode`, {
        mode: mode
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setThermostatMode error:', error.message);
      throw new Error(`Failed to set thermostat mode: ${deviceId}`);
    }
  },

  async setCoolingSetpoint(authToken, deviceId, temperature) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/thermostat/cooling-setpoint`, {
        temperature: temperature
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setCoolingSetpoint error:', error.message);
      throw new Error(`Failed to set cooling setpoint: ${deviceId}`);
    }
  },

  async setHeatingSetpoint(authToken, deviceId, temperature) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/thermostat/heating-setpoint`, {
        temperature: temperature
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setHeatingSetpoint error:', error.message);
      throw new Error(`Failed to set heating setpoint: ${deviceId}`);
    }
  },

  async setLock(authToken, deviceId, state) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/lock`, {
        state: state
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setLock error:', error.message);
      throw new Error(`Failed to set lock: ${deviceId}`);
    }
  },

  async setDoorState(authToken, deviceId, state) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/door`, {
        state: state
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setDoorState error:', error.message);
      throw new Error(`Failed to set door state: ${deviceId}`);
    }
  },

  async setWindowShade(authToken, deviceId, state) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/window-shade`, {
        state: state
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setWindowShade error:', error.message);
      throw new Error(`Failed to set window shade: ${deviceId}`);
    }
  },

  async setShadeLevel(authToken, deviceId, level) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/shade-level`, {
        level: level
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setShadeLevel error:', error.message);
      throw new Error(`Failed to set shade level: ${deviceId}`);
    }
  },

  async setHue(authToken, deviceId, hue) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/hue`, {
        hue: hue
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setHue error:', error.message);
      throw new Error(`Failed to set hue: ${deviceId}`);
    }
  },

  async setSaturation(authToken, deviceId, saturation) {
    try {
      await axios.post(`${DEVICE_API_BASE_URL}/devices/${deviceId}/saturation`, {
        saturation: saturation
      }, { headers: { 'Authorization': `Bearer ${authToken}`, 'Content-Type': 'application/json' } });
    } catch (error) {
      console.error('[DeviceAPI] setSaturation error:', error.message);
      throw new Error(`Failed to set saturation: ${deviceId}`);
    }
  }
};

module.exports = deviceApi;
```

---

## Environment Variable Configuration

### .env.example

```env
# Server Configuration
PORT=3000
NODE_ENV=development

# SmartThings Configuration
ST_CLIENT_ID=your-smartthings-client-id
ST_CLIENT_SECRET=your-smartthings-client-secret
MANUFACTURER_NAME=YourCompany

# Device API Configuration
DEVICE_API_URL=https://api.yourcompany.com/v1

# OAuth2 Configuration (Own auth server)
OAUTH_AUTHORIZE_URL=https://auth.yourcompany.com/oauth/authorize
OAUTH_TOKEN_URL=https://auth.yourcompany.com/oauth/token
OAUTH_CLIENT_ID=your-oauth-client-id
OAUTH_CLIENT_SECRET=your-oauth-client-secret
```

### .gitignore

```
node_modules/
.env
*.log
.DS_Store
```

---

## Local Testing

### Local Testing with ngrok

1. **Install ngrok**
```bash
npm install -g ngrok
```

2. **Run server**
```bash
npm run dev
```

3. **Create ngrok tunnel**
```bash
ngrok http 3000
```

4. **Check ngrok URL**
```
Forwarding  https://xxxx-xx-xx-xxx-xx.ngrok.io -> http://localhost:3000
```

5. **Register in SmartThings Schema**
   - Register the ngrok URL as the SmartThings Schema endpoint
   - Example: `https://xxxx-xx-xx-xxx-xx.ngrok.io`

### Testing with curl

#### Discovery Test
```bash
curl -X POST http://localhost:3000/ \
  -H "Content-Type: application/json" \
  -d '{
    "headers": {
      "schema": "st-schema",
      "version": "1.0",
      "interactionType": "discoveryRequest",
      "requestId": "test-request-id"
    },
    "authentication": {
      "token": "test-oauth-token",
      "tokenType": "Bearer"
    }
  }'
```

#### StateRefresh Test
```bash
curl -X POST http://localhost:3000/ \
  -H "Content-Type: application/json" \
  -d '{
    "headers": {
      "schema": "st-schema",
      "version": "1.0",
      "interactionType": "stateRefreshRequest",
      "requestId": "test-request-id"
    },
    "authentication": {
      "token": "test-oauth-token",
      "tokenType": "Bearer"
    },
    "devices": [
      {
        "externalDeviceId": "device-001"
      }
    ]
  }'
```

#### Command Test
```bash
curl -X POST http://localhost:3000/ \
  -H "Content-Type: application/json" \
  -d '{
    "headers": {
      "schema": "st-schema",
      "version": "1.0",
      "interactionType": "commandRequest",
      "requestId": "test-request-id"
    },
    "authentication": {
      "token": "test-oauth-token",
      "tokenType": "Bearer"
    },
    "devices": [
      {
        "externalDeviceId": "device-001",
        "commands": [
          {
            "component": "main",
            "capability": "switch",
            "command": "on",
            "arguments": []
          }
        ]
      }
    ]
  }'
```

---

## Deployment Guide

### AWS Lambda Deployment

1. **Install Serverless Framework**
```bash
npm install -g serverless
```

2. **Create serverless.yml**
```yaml
service: smartthings-schema-app

provider:
  name: aws
  runtime: nodejs18.x
  stage: dev
  region: us-east-1

functions:
  connector:
    handler: src/index.handler
    events:
      - http:
          path: /
          method: post
          cors: true
      - http:
          path: health
          method: get
          cors: true
```

3. **Deploy**
```bash
serverless deploy
```

### Docker Deployment

1. **Create Dockerfile**
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY src/ ./src/

EXPOSE 3000

CMD ["node", "src/index.js"]
```

2. **Build Docker image**
```bash
docker build -t smartthings-schema-app .
```

3. **Run Docker**
```bash
docker run -p 3000:3000 --env-file .env smartthings-schema-app
```

---

## Proactive State Callbacks

How to send callbacks to SmartThings when device state changes.

### Callback Implementation Example

```javascript
const axios = require('axios');

/**
 * Send state change callback to SmartThings
 *
 * @param {string} callbackUrl - Callback URL received from callbackAccessHandler
 * @param {string} clientId - Client ID issued by SmartThings
 * @param {string} clientSecret - Client secret issued by SmartThings
 * @param {string} authToken - User OAuth token
 * @param {Array} deviceStates - List of changed device states
 */
async function sendStateCallback(callbackUrls, clientId, clientSecret, callbackAuth, deviceStates) {
  try {
    const { StateUpdateRequest } = require('st-schema');

    // Use SDK's StateUpdateRequest class
    const callback = new StateUpdateRequest(clientId, clientSecret);

    await callback.updateState(callbackUrls, callbackAuth, deviceStates);

    console.log('[Callback] State callback sent successfully via SDK');
  } catch (error) {
    console.error('[Callback] Error sending state callback:', error.message);
    throw error;
  }
}
```

### Usage Example

```javascript
// When device state change is detected
const deviceStates = [
  {
    externalDeviceId: 'device-001',
    states: [
      {
        component: 'main',
        capability: 'switch',
        attribute: 'switch',
        value: 'on'
      }
    ]
  }
];

await sendStateCallback(
  callbackUrls, // Saved callbackUrls object
  process.env.ST_CLIENT_ID,
  process.env.ST_CLIENT_SECRET,
  callbackAuth, // Saved callbackAuth/callbackAuthentication object
  deviceStates
);
```

---

## Error Handling

### Error Types (Examples)

| Error Enum | Description |
|------------|-------------|
| `DEVICE-OFFLINE` | Device is offline and cannot accept commands. |
| `DEVICE-UNAVAILABLE` | Device is temporarily unavailable (e.g. firmware update). |
| `DEVICE-DELETED` | Device is deleted and cannot accept commands. |
| `CAPABILITY-NOT-SUPPORTED` | Requested capability or command is not supported by the device. |
| `RESOURCE-CONSTRAINT-VIOLATION` | Requested action violates a resource constraint (e.g. out of bounds value). |
| `BAD-REQUEST` | Bad request or missing st-schema headers/authentication. |
| `TOKEN-EXPIRED` | Token has expired. |
| `INTEGRATION-OFFLINE` | All devices in the integration are offline, or the integration is unreachable. |
| `INTEGRATION-DELETED` | User has removed the integration. |

### Error Response Format

```javascript
{
  headers: {
    schema: 'st-schema',
    version: '1.0',
    interactionType: 'commandResponse'
  },
  authentication: {
    token: authToken
  },
  deviceState: [
    {
      externalDeviceId: 'device-001',
      deviceError: [
        {
          errorEnum: 'DEVICE-OFFLINE',
          detail: 'Device is offline'
        }
      ]
    }
  ]
}
```

---

## References

- [ST-Schema SDK (Node.js)](https://github.com/SmartThingsCommunity/st-schema-nodejs)
- [Interaction Types](https://developer.smartthings.com/docs/devices/cloud-connected/interaction-types)
- [Proactive State Callbacks](https://github.com/SmartThingsCommunity/st-schema-nodejs/tree/master?tab=readme-ov-file#proactive-state-callbacks)
