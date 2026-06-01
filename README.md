# SmartThings Developer Skills

> **📢 Notice**
> Features, documentation, and skill behaviors may change. Your feedback via GitHub Issues is highly appreciated!

A collection of AI skills that accelerates SmartThings device integration. It supports the full lifecycle, from initial code generation to WWST certification.

## 📖 Overview

This project provides AI skills for developers who want to integrate devices into the SmartThings ecosystem. Each skill improves development productivity with step-by-step guidance, best practices, code generation, and debugging support.

### SmartThings Device Integration Types

SmartThings supports three integration types depending on the device characteristics and the manufacturer's infrastructure:

| Integration Type | Description | Best For |
|------------------|-------------|----------|
| **Cloud Connected** | A Device Integration in which devices communicate to the SmartThings cloud via your own cloud | Your devices connect to SmartThings through your own cloud backend |
| **Hub Connected** | A Device Integration in which devices connect to SmartThings via a SmartThings hub using Matter, Zigbee, or Z-Wave | Your devices integrate into the ecosystem through a SmartThings Hub |
| **Direct Connected** | A Device Integration in which devices communicate directly to the SmartThings cloud via MQTT / SmartThings SDK | Your IP devices connect to SmartThings without a separate cloud backend |

---

## 🚀 Installation and Usage

### Prerequisites

- SmartThings Developer account ([Sign up](https://developer.smartthings.com/))
- SmartThings CLI (optional, recommended)
- A coding assistant environment such as Claude Code, Codex, or Antigravity

### How To Use These Skills

These skills can be used with AI assistants that support MCP (Model Context Protocol), such as Cline, Cursor, and Claude Desktop.

1. **Installation**
   Install the full skill directory under `skills/` that matches your AI development environment.

   You can instruct your AI assistant to install the skills directly by pasting the following prompt:

   ```text
   Install Agent Skills from https://github.com/SmartThingsCommunity/wwst-skills.
   ```

   > **💡 Installation Tip**: If your AI Coding Assistant does not support automatic skill installation, refer to the [Open Agent Skills specification](https://agentskills.io/client-implementation/adding-skills-support#step-1-discover-skills) or your AI Coding Assistant's Skills documentation for manual setup instructions.

   **Uninstall**
   - Delete the copied skill directory, or remove the reference from your tool's configuration.

2. **Invoking a skill**

   When you ask in natural language, the AI assistant can automatically activate the appropriate skill:

   - **Cloud Connected (With API Spec)**
     ```text
     Help me build a SmartThings Cloud Connected device integration. I have my own REST API specification ready in cloud_spec.md.
     ```
   - **Hub Connected (With Device Spec Details)**
     ```text
     I want to integrate this Zigbee device with SmartThings. It uses a Zigbee endpoint with Basic, Identify, On/Off, Level Control, and Electrical Measurement clusters.
     ```
   - **Direct Connected / MQTT (With Product URL)**
     ```text
     How can I integrate my smart fan into SmartThings using MQTT? It has speed control, oscillation, and a timer. Here is the product description page URL: https://example.com/product
     ```

---

## 📚 Skill List

This English README currently documents the following skills under `skills/`.

### 1. Cloud Connected (Schema App)

> **Skill**: `smartthings-cloud-connected-developer`

This skill guides developers through the entire lifecycle of creating a SmartThings Cloud Connected (Schema App) integration, from organization setup to code generation, registration, and final WWST certification.

**Key features**
- Step-by-step progression from org setup through certification (5-step workflow)
- Device Profiles and Capabilities mapping
- Schema App code generation
- OAuth configuration, hosting, and Schema App registration guidance
- Debugging support for common integration issues (OAuth, discovery, state sync)

**Best fit**
- Teams with an existing cloud backend who want Cloud Connected integration
- Developers building a Schema App using Node.js
- Teams preparing WWST certification for Cloud Connected devices

### 2. Hub Connected

> **Skill**: `smartthings-hub-connected-developer`

This skill guides WWST certification for hub-connected products that use Zigbee, Matter, or Z-Wave through SmartThings Hub.

**Key features**
- Standard vs Custom path guidance
- Developer Console and Test Suite workflow
- Zigbee Edge driver PR worked example
- Capability mapping and certification guidance

**Best fit**
- Hub-connected products using Zigbee, Matter, or Z-Wave
- Teams preparing WWST certification or driver contribution work

### 3. Direct Connected Device SDK for C

> **Skill**: `smartthings-direct-connected-device-sdk-c-developer`

This skill guides SmartThings Direct Connected integration using the SmartThings Device SDK for C over MQTT.

**Key features**
- IoT core device library vs SDK Reference starting guidance
- Porting boundary and application layer guidance
- Console setup and onboarding asset guidance
- Device identity registration and WWST prerequisites

**Best fit**
- Direct Connected products using the SmartThings Device SDK for C
- Teams working on porting, provisioning, and certification readiness

### 4. Device Onboarding QR Codes

> **Skill**: `smartthings-device-onboarding-qr`

This skill explains SmartThings device onboarding QR requirements across Matter, Zigbee 3.0, Direct Connected, and Mobile Connected products.

**Key features**
- Integration-type-specific QR payload guidance
- Zigbee minimum vs recommended payload coverage
- Direct Connected and Mobile Connected QR field guidance
- Product-oriented payload examples and partner publication notes

**Best fit**
- Teams defining device labels and onboarding assets
- Developers validating QR payload fields before publication
- Partners handling Matter, Zigbee 3.0, Direct Connected, or Mobile Connected onboarding flows

---

## ⚠️ Limitations and Notes

These skills provide development guidance and code generation support for SmartThings device integration. Before applying their output, verify the relevant content with the official SmartThings documentation, Developer Console, SmartThings CLI, and real-device testing.

---

## 🤝 Contributing

If you find a bug, have a suggestion for improvement, or encounter issues while using the skills, please **create an Issue** in this repository. Your feedback is highly appreciated and will help us improve the skills.

---

## 📁 Project Structure

```text
wwst-skills/
├── README.md                                        # README
├── skills/                                              # Skills
│   ├── smartthings-cloud-connected-developer/
│   ├── smartthings-direct-connected-device-sdk-c-developer/
│   ├── smartthings-device-onboarding-qr/
│   ├── smartthings-hub-connected-developer/
│   └── ...
```

---

## 🔗 Useful Links

### Official Documentation
- [SmartThings Developer Center](https://developer.smartthings.com/)
- [Device Profiles Guide](https://developer.smartthings.com/docs/devices/device-profiles)
- [Capabilities Reference](https://developer.smartthings.com/docs/devices/capabilities/capabilities-reference)

### Developer Tools
- [SmartThings CLI](https://github.com/SmartThingsCommunity/smartthings-cli)
- [Schema App SDK (Node.js)](https://github.com/SmartThingsCommunity/st-schema-nodejs)

### Certification and Release
- [WWST Certification Requirements](https://developer.smartthings.com/docs/certification/required-capabilities)
- [Works With SmartThings](https://www.smartthings.com/works-with-smartthings)
