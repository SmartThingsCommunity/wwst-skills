# API Spec Mapping Guide

This document explains how to map your own Device API specs to SmartThings Schema App handlers.

## Table of Contents

1. [Overview](#overview)
2. [API Spec Input Methods](#api-spec-input-methods)
3. [Swagger/OpenAPI Parsing](#swaggeropenapi-parsing)
4. [Handler Mapping Rules](#handler-mapping-rules)
5. [State Mapping](#state-mapping)
6. [Command Mapping](#command-mapping)
7. [Mapping Examples](#mapping-examples)

---

## Overview

API mapping proceeds through the following process:

1. **Load API Spec**: Swagger/OpenAPI JSON, text description, or separate file
2. **Analyze Endpoints**: Identify device list, state query, and control APIs
3. **Map to SmartThings Handlers**: Map to Discovery, StateRefresh, and Command handlers
4. **Code Generation**: Generate Schema App code based on the mapped information

---

## API Spec Input Methods

### 1. Swagger/OpenAPI JSON

Save the file in the `/spec/` directory or provide it directly:

```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "Device API",
    "version": "1.0.0"
  },
  "paths": {
    "/devices": {
      "get": {
        "summary": "Get device list",
        "responses": {
          "200": {
            "description": "Device list"
          }
        }
      }
    }
  }
}
```

### 2. Text Description

Provide API description in natural language:

```
Device list query: GET /api/v1/devices
- Header: Authorization: Bearer {token}
- Response: { devices: [{ id, name, type, ... }] }

Device state query: GET /api/v1/devices/{deviceId}/status
- Response: { power: "on", level: 80, ... }

Device control: POST /api/v1/devices/{deviceId}/command
- Body: { command: "setPower", value: true }
```

### 3. Separate Spec Files

Save various format files in the `/spec/` directory:
- `api-spec.json` (OpenAPI/Swagger)
- `api-spec.yaml` (OpenAPI/Swagger YAML)
- `api-docs.md` (Markdown document)
- `endpoints.txt` (Text description)

---

## Swagger/OpenAPI Parsing

### Target Information for Parsing

| Information | Description | Mapping Target |
|-------------|-------------|----------------|
| `GET /devices` | Device list query | discoveryHandler |
| `GET /devices/{id}` | Device detail information | discoveryHandler |
| `GET /devices/{id}/state` | Device state query | stateRefreshHandler |
| `POST /devices/{id}/power` | Power control | commandHandler (st.switch) |
| `POST /devices/{id}/level` | Level control | commandHandler (st.switchLevel) |
| `POST /devices/{id}/color` | Color control | commandHandler (st.colorControl) |
| `POST /devices/{id}/temperature` | Temperature setting | commandHandler (st.thermostatSetpoint) |

### OpenAPI Example Analysis

```json
{
  "openapi": "3.0.0",
  "paths": {
    "/devices": {
      "get": {
        "operationId": "getDevices",
        "tags": ["Device"],
        "parameters": [
          {
            "name": "Authorization",
            "in": "header",
            "schema": { "type": "string" }
          }
        ],
        "responses": {
          "200": {
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "devices": {
                      "type": "array",
                      "items": {
                        "$ref": "#/components/schemas/Device"
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    },
    "/devices/{deviceId}/state": {
      "get": {
        "operationId": "getDeviceState",
        "parameters": [
          { "name": "deviceId", "in": "path", "required": true }
        ],
        "responses": {
          "200": {
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/DeviceState"
                }
              }
            }
          }
        }
      }
    },
    "/devices/{deviceId}/power": {
      "post": {
        "operationId": "setPower",
        "parameters": [
          { "name": "deviceId", "in": "path", "required": true }
        ],
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "power": { "type": "string", "enum": ["on", "off"] }
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "Device": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "name": { "type": "string" },
          "type": { "type": "string" },
          "model": { "type": "string" },
          "online": { "type": "boolean" }
        }
      },
      "DeviceState": {
        "type": "object",
        "properties": {
          "power": { "type": "boolean" },
          "level": { "type": "integer", "minimum": 0, "maximum": 100 },
          "temperature": { "type": "number" }
        }
      }
    }
  }
}
```

---

## Handler Mapping Rules

### Discovery Handler Mapping

| Your API | SmartThings Discovery |
|----------|----------------------|
| `GET /devices` | Device list query |
| `id` → `externalDeviceId` | Device unique ID |
| `name` → `friendlyName` | Display name |
| `type` → `deviceHandlerType` | Device Handler Type |

**Mapping Code Example:**

```javascript
// API response
{
  "devices": [
    { "id": "dev-001", "name": "Living Room Light", "type": "light" }
  ]
}

// Converted to SmartThings Discovery response
{
  "externalDeviceId": "dev-001",
  "friendlyName": "Living Room Light",
  "deviceHandlerType": "light"
}
```

### StateRefresh Handler Mapping

| Your API | SmartThings StateRefresh |
|----------|-------------------------|
| `GET /devices/{id}/state` | Device state query |
| `power` → `st.switch.switch` | Power state |
| `level` → `st.switchLevel.level` | Level |
| `temperature` → `st.temperatureMeasurement.temperature` | Temperature |

### Command Handler Mapping

| Your API | SmartThings Command |
|----------|---------------------|
| `POST /devices/{id}/power` | `st.switch.on/off` |
| `POST /devices/{id}/level` | `st.switchLevel.setLevel` |
| `POST /devices/{id}/color` | `st.colorControl.setColor` |

---

## State Mapping

### Basic State Mapping Table

| Your API Field | SmartThings Capability | Attribute | Value Conversion |
|----------------|------------------------|-----------|------------------|
| `power: true/false` | `st.switch` | `switch` | `"on"` / `"off"` |
| `level: 0-100` | `st.switchLevel` | `level` | Use as-is |
| `hue: 0-100` | `st.colorControl` | `hue` | Use as-is |
| `saturation: 0-100` | `st.colorControl` | `saturation` | Use as-is |
| `colorTemperature: 2200-6500` | `st.colorTemperature` | `colorTemperature` | Use as-is |
| `temperature: number` | `st.temperatureMeasurement` | `temperature` | Check unit (C/F) |
| `humidity: number` | `st.relativeHumidityMeasurement` | `humidity` | Use as-is |
| `mode: string` | `st.thermostatMode` | `thermostatMode` | Mapping required |
| `locked: boolean` | `st.lock` | `lock` | `"locked"` / `"unlocked"` |
| `online: boolean` | `st.healthCheck` | `healthStatus` | `"online"` / `"offline"` |
| `battery: 0-100` | `st.battery` | `battery` | Use as-is |
| `motion: boolean` | `st.motionSensor` | `motion` | `"active"` / `"inactive"` |
| `contact: boolean` | `st.contactSensor` | `contact` | `"open"` / `"closed"` |

### Thermostat Mode Mapping

| Your API Value | SmartThings Value |
|----------------|-------------------|
| `off` | `off` |
| `cool` / `cooling` | `cool` |
| `heat` / `heating` | `heat` |
| `auto` / `automatic` | `auto` |
| `emergency` / `emergencyHeat` | `emergencyHeat` |
| `precool` / `precooling` | `precooling` |

### Air Conditioner Mode Mapping

| Your API Value | SmartThings Value |
|----------------|-------------------|
| `off` | `off` |
| `auto` | `auto` |
| `cool` | `cool` |
| `heat` | `heat` |
| `dry` | `dry` |
| `fan` | `wind` |

---

## Command Mapping

### Switch Commands

| SmartThings Command | Your API Request |
|---------------------|------------------|
| `st.switch.on` | `POST /devices/{id}/power { "power": "on" }` |
| `st.switch.off` | `POST /devices/{id}/power { "power": "off" }` |

### Level Commands

| SmartThings Command | Your API Request |
|---------------------|------------------|
| `switchLevel.setLevel([80])` | `POST /devices/{id}/level { "level": 80 }` |

### Color Commands

| SmartThings Command | Your API Request |
|---------------------|------------------|
| `colorControl.setColor({hue: 50, saturation: 100})` | `POST /devices/{id}/color { "hue": 50, "saturation": 100 }` |
| `colorControl.setHue([50])` | `POST /devices/{id}/color { "hue": 50 }` |
| `colorControl.setSaturation([100])` | `POST /devices/{id}/color { "saturation": 100 }` |

### Thermostat Commands

| SmartThings Command | Your API Request |
|---------------------|------------------|
| `st.thermostatMode.setThermostatMode(["cool"])` | `POST /devices/{id}/mode { "mode": "cool" }` |
| `st.thermostatCoolingSetpoint.setCoolingSetpoint([24])` | `POST /devices/{id}/cooling-setpoint { "temperature": 24 }` |
| `st.thermostatHeatingSetpoint.setHeatingSetpoint([20])` | `POST /devices/{id}/heating-setpoint { "temperature": 20 }` |

---

## Mapping Examples

### Example 1: Smart Light API

**Swagger Spec:**

```json
{
  "openapi": "3.0.0",
  "info": { "title": "Smart Light API", "version": "1.0.0" },
  "paths": {
    "/lights": {
      "get": {
        "operationId": "getLights",
        "responses": {
          "200": {
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": { "$ref": "#/components/schemas/Light" }
                }
              }
            }
          }
        }
      }
    },
    "/lights/{lightId}": {
      "get": {
        "operationId": "getLightState",
        "parameters": [
          { "name": "lightId", "in": "path", "required": true }
        ],
        "responses": {
          "200": {
            "content": {
              "application/json": {
                "schema": { "$ref": "#/components/schemas/LightState" }
              }
            }
          }
        }
      }
    },
    "/lights/{lightId}/power": {
      "post": {
        "operationId": "setLightPower",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "on": { "type": "boolean" }
                }
              }
            }
          }
        }
      }
    },
    "/lights/{lightId}/brightness": {
      "post": {
        "operationId": "setLightBrightness",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "brightness": { "type": "integer", "min": 0, "max": 100 }
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "Light": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "name": { "type": "string" },
          "room": { "type": "string" }
        }
      },
      "LightState": {
        "type": "object",
        "properties": {
          "on": { "type": "boolean" },
          "brightness": { "type": "integer" },
          "colorTemp": { "type": "integer" }
        }
      }
    }
  }
}
```

**Mapping Result:**

```javascript
// device-api.js auto-generated
const deviceApi = {
  async getDevices(authToken) {
    const response = await axios.get(`${BASE_URL}/lights`, {
      headers: { 'Authorization': `Bearer ${authToken}` }
    });
    return response.data.map(light => ({
      id: light.id,
      name: light.name,
      deviceHandlerType: 'light'  // or 'colorTemperatureLight'
    }));
  },

  async getDeviceState(authToken, deviceId) {
    const response = await axios.get(`${BASE_URL}/lights/${deviceId}`, {
      headers: { 'Authorization': `Bearer ${authToken}` }
    });
    return {
      power: response.data.on,
      level: response.data.brightness,
      colorTemperature: response.data.colorTemp
    };
  },

  async setPower(authToken, deviceId, power) {
    await axios.post(`${BASE_URL}/lights/${deviceId}/power`, {
      on: power
    }, {
      headers: { 'Authorization': `Bearer ${authToken}` }
    });
  },

  async setLevel(authToken, deviceId, level) {
    await axios.post(`${BASE_URL}/lights/${deviceId}/brightness`, {
      brightness: level
    }, {
      headers: { 'Authorization': `Bearer ${authToken}` }
    });
  }
};
```

### Example 2: Text Description Based Mapping

**Input Text:**

```
Our company IoT API spec:

1. Device list query
   - GET /api/devices
   - Header: Authorization: Bearer {token}
   - Response: [{ deviceId, deviceName, deviceType, isOnline }]

2. Device state query
   - GET /api/devices/{deviceId}
   - Response: { status: "on"/"off", brightness: 0-100, currentTemp: number }

3. Device control
   - POST /api/devices/{deviceId}/control
   - Body: { action: "turnOn"|"turnOff"|"setBrightness", value: any }
```

**Mapping Analysis:**

| API Information | Mapping Result |
|----------------|----------------|
| `GET /api/devices` | discoveryHandler |
| `deviceId` → `externalDeviceId` | Discovery |
| `deviceName` → `friendlyName` | Discovery |
| `deviceType` → `deviceHandlerType` | Discovery |
| `GET /api/devices/{id}` | stateRefreshHandler |
| `status` → `st.switch.switch` | StateRefresh |
| `brightness` → `st.switchLevel.level` | StateRefresh |
| `currentTemp` → `st.temperatureMeasurement.temperature` | StateRefresh |
| `POST /api/devices/{id}/control` | commandHandler |
| `action: turnOn/turnOff` | `st.switch.on/off` |
| `action: setBrightness` | `st.switchLevel.setLevel` |

---

## Authentication Method Handling

### OAuth2 Bearer Token

```javascript
const response = await axios.get(url, {
  headers: {
    'Authorization': `Bearer ${authToken}`,
    'Content-Type': 'application/json'
  }
});
```

### API Key

```javascript
const response = await axios.get(url, {
  headers: {
    'X-API-Key': process.env.DEVICE_API_KEY,
    'Content-Type': 'application/json'
  }
});
```

### Basic Auth

```javascript
const response = await axios.get(url, {
  auth: {
    username: process.env.API_USERNAME,
    password: process.env.API_PASSWORD
  }
});
```

---

## Error Mapping

| Your API Error | SmartThings Error |
|----------------|-------------------|
| 401 Unauthorized | `AUTHORIZATION-ERROR` |
| 404 Not Found | `DEVICE-ERROR` (device not found) |
| 500 Internal Error | `INTEGRATION-ERROR` |
| 429 Rate Limited | `RATE-LIMIT-ERROR` |
| Device offline | `DEVICE-ERROR` |
| Timeout | `DEVICE-ERROR` |

---

## Checklist

Items to verify when performing API mapping:

- [ ] Device list query API confirmed
- [ ] Device state query API confirmed
- [ ] Device control API confirmed
- [ ] Authentication method confirmed (OAuth2, API Key, etc.)
- [ ] State field and Capability mapping confirmed
- [ ] Command and API action mapping confirmed
- [ ] Error handling method confirmed
- [ ] Timeout settings confirmed