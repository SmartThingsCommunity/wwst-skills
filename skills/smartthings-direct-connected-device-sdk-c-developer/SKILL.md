---
name: smartthings-direct-connected-device-sdk-c-developer
description: Guides SmartThings Direct Connected integration using the SmartThings Device SDK for C over MQTT. Covers when to start from the IoT core device library repository vs the SmartThings SDK Reference repository, strict porting boundaries, capability handling via capability_sample, Console project setup, project-specific onboarding_config.json replacement, device_info.json handling for prototype vs production flows, device identity registration with serial number and ed25519 public key, and WWST certification and product launch prerequisites.
metadata:
  version: "2026-05-29"
---

# Direct Connected -> SmartThings Device SDK for C

## When to use

- Use this skill when working on a **SmartThings Direct Connected** device that integrates with SmartThings using the **SmartThings Device SDK for C** over **MQTT**.
- Use it when the target device firmware is built either from the **IoT core device library** repository or from the **SmartThings SDK Reference** repository for supported chipsets.
- Use it when you need guidance on **porting**, **capability handling**, **Console preparation**, or **device identity provisioning** for a Direct Connected product.

---

## Scope

**In scope**

- Choosing the correct starting point between the IoT core device library repo and the SmartThings SDK Reference repository.
- Porting rules for unsupported chipsets.
- Recommended capability handling flow using `capability_sample`.
- Required SmartThings Console setup for a Direct Connected project.
- Required onboarding asset replacement and device identity registration.
- WWST certification and product launch prerequisites such as contract and certification testing.

**Out of scope unless the user expands**

- Hub Connected, Cloud Connected, or Mobile Connected flows.
- BSP-specific bring-up details for chipsets not already covered by the SmartThings SDK Reference repository.
- Secure element or manufacturing-line implementation details beyond the required key-registration workflow.

---

## Core repositories

- **IoT core device library:** [st-device-sdk-c](https://github.com/SmartThingsCommunity/st-device-sdk-c)
- **SmartThings SDK Reference:** [st-device-sdk-c-ref](https://github.com/SmartThingsCommunity/st-device-sdk-c-ref)

Use these repositories differently:

- For **ESP32**, **BL602**, and **BK7236**, start from the **SmartThings SDK Reference** repository when possible.
- For other chipsets, start from the **IoT core device library** repository and port it to the target platform yourself.

---

## Critical porting guardrail

When porting the IoT core device library to a new chipset:

- You may modify the **port layer** under [`src/port`](https://github.com/SmartThingsCommunity/st-device-sdk-c/tree/main/src/port).
- Do **not** modify code outside the port layer in the IoT core device library repository.

If the user proposes changing logic outside `src/port`, steer them back to a port-layer-only approach unless they explicitly have upstream guidance from SmartThings.

---

## Recommended starting path

1. Confirm whether the target BSP is supported in the **SmartThings SDK Reference** repository.
2. If yes, begin from the relevant example in the **SmartThings SDK Reference** repository and focus on implementing the application layer.
3. If not, first port the **IoT core device library** to the target BSP by implementing the required platform adaptation in `src/port`, then implement the application layer on top of it.
4. In all cases, build the device application by using the IoT core APIs exposed through `st_dev.h`.

### Porting layer and application layer

In this integration flow, the implementation is divided into two parts:

- **Porting layer:** the low-level adaptation layer used to port the **IoT core device library** to the target BSP or chipset platform.
- **Application layer:** the device application built by using the IoT core APIs exposed through `st_dev.h`, including device behavior and capability handling.

The device developer is responsible for both layers.

However, for **ESP32**, **BL602**, and **BK7236**, the port layer is already provided in the **SmartThings SDK Reference** repository, so in those cases the developer usually only needs to implement the application layer.

### Prerequisites check before application work

Before starting application implementation, first verify that the prerequisites from the **SmartThings SDK Reference** README are already installed for the target chipset.

- For ESP32, check whether the build-system prerequisites described by Espressif are installed.
- For ESP32, also check whether the ESP32 toolchain setup has been completed through `setup.py`.
- If those prerequisites are missing, guide the user to complete them before moving on to application-layer work.

---

## Capability handling recommendation

For capability handling, the preferred starting point is:

- [`apps/capability_sample`](https://github.com/SmartThingsCommunity/st-device-sdk-c-ref/tree/main/apps/capability_sample)

Recommended workflow:

1. Find the matching capability-specific `caps_*.c` and `caps_*.h` files for the required capability in `capability_sample`.
2. Copy those matching capability-specific `caps_*.c` and `caps_*.h` files into the working project.
3. Adapt them inside the product application, for example in a project similar to [`apps/esp32/switch_example`](https://github.com/SmartThingsCommunity/st-device-sdk-c-ref/tree/main/apps/esp32/switch_example).
4. Use the copied capability files as the abstraction layer for SmartThings capability handling instead of rebuilding the capability logic from scratch.

This path is usually more convenient than implementing capability message handling directly at the raw SDK layer.

### Capability schema guardrail

- Do not hardcode SmartThings capability enum strings, unit strings, or other schema-constrained constants in application code.
- Always use the matching definitions from `iot_caps_helper_*.h` when setting capability values, units, or enum-like strings.
- Before sending capability events, verify that the attribute value, unit, and allowed range come from the helper definitions for that capability.
- If a helper-defined constant exists, use it instead of writing a literal string directly.

### Capability event rate-limit guidance

- SmartThings applies device-side rate limits and guardrails to connected devices. The official guidance says devices should generally not emit more than one event per minute unless the state change was caused by user interaction.
- If a device sends events too aggressively, the connection to SmartThings Cloud may be interrupted and re-established, so periodic-reporting devices should control event volume within a range that does not harm usability.
- For SDK-based devices, rate limiting is effectively tied to how often the device sends attribute updates through `st_cap_send_attr()`.
- If several capability attributes need to be updated together, send them in a single `st_cap_send_attr()` call instead of splitting them into separate calls where possible.
- This is especially useful when the device reconnects and needs to publish its initial full state efficiently.

References:

- SmartThings rate limits: [Rate Limits and Guardrails - Devices](https://developer.smartthings.com/docs/getting-started/rate-limits#devices)
- SDK capability attribute updates: [Capability_Attribute_Update.md - Sending Capability Attribute data](https://github.com/SmartThingsCommunity/st-device-sdk-c/blob/main/doc/Capability_Attribute_Update.md#sending-capability-attribute-data)

---

## SmartThings Console preparation

Before firmware integration, the manufacturer must create a new project in **SmartThings Console**:

1. Create a new project.
2. Choose **Direct Connected** as the integration type.
3. Define the device behavior and SmartThings-visible functionality for the product.
4. Download the project-specific `onboarding_config.json`.
5. Replace the placeholder `onboarding_config.json` included in the SDK/project with the downloaded file.

Treat the Console-generated `onboarding_config.json` as product-specific configuration, not as a reusable placeholder.

Repository note:

- Example projects under paths such as `apps/<chipset>/<example>/main/` may include placeholder `onboarding_config.json` and `device_info.json` files in GitHub.
- Those checked-in files are only placeholders for repository distribution and must not be treated as ready-to-ship project assets.

---

## Testing your device

- Follow the official test flow in [Test Your Direct Connected Device](https://developer.smartthings.com/docs/devices/direct-connected/test-your-device-app).
- The recommended test sequence is:
  1. Deploy product information in **SmartThings Console**
  2. Register the test device
  3. Test the device in the **SmartThings app**
- Before testing in the SmartThings app, make sure **developer mode is enabled** in the app.
- If developer mode is not enabled, the test device will not appear in the SmartThings app.
- The SmartThings app and SmartThings Console should be signed in with the same Samsung account during testing.
- In the app, test devices are typically accessed through **Add device -> Partner devices -> My Testing Devices**.

---

## Device identity and authentication

Each device must be registered as a legitimate product instance before authentication succeeds.

Notation rule in this section:

- Use **serial number**, **public key**, and **private key** for the general concepts.
- Use `serialNumber`, `publicKey`, and `privateKey` only when referring to literal JSON field names.

Required provisioning flow:

1. Generate a device-specific **serial number**.
2. Generate an **ed25519 keypair** for the device.
3. Register the device's **serial number** and **public key** through SmartThings Console.
4. Inject the **private key** into the device through a secure manufacturing or provisioning flow.

Additional requirement:

- A **QR code** containing product-specific **serial number** information is also required.
- QR code details are out of scope for this skill and should be handled by a separate skill or guide.

Security rule:

- The **private key** must be handled securely and stored or used in a secure manner on the device.

If the user asks about onboarding/authentication failures, check first whether the correct device identity and **public key** were registered in Console and whether the device is using the matching **private key**.

### `device_info.json` for prototypes vs production

- The placeholder `device_info.json` included in example projects should be replaced with a project/device-specific file before real testing.
- For prototyping or small-scale test-device work, `tools/keygen` in [`st-device-sdk-c`](https://github.com/SmartThingsCommunity/st-device-sdk-c/tree/main/tools/keygen) can be used to generate a device-specific **serial number**, keypair, and a ready-to-use `device_info.json`.
- In that prototype flow, register the generated **serial number** and **public key** in SmartThings Console and make sure the device uses the matching **private key**.
- For a production or mass-manufacturing project, do not ship `privateKey`, `publicKey`, or `serialNumber` fields inside `device_info.json`.
- In production guidance, tell the user to remove those identity fields from `device_info.json` and provision the device identity through the manufacturing/security flow instead.

---

## WWST certification and launch notes

- Development and commercialization with this SDK require a **separate commercial charging/contract arrangement with Samsung Electronics**.
- Product launch requires completing the required **WWST certification** process.
- WWST certification for Direct Connected devices requires not only functional validation but also a **Security Assessment**.
- Security features such as **firmware update**, **secure boot**, and **secure storage** should be implemented as mandatory parts of the product security design for certification readiness.
- Before requesting certification, review [Security Requirements for Direct Connected Devices](https://developer.smartthings.com/docs/devices/direct-connected/security-requirements) and the related security material available in **SmartThings Console**, then apply the relevant security features in advance.
- As part of the WWST certification submission flow, the manufacturer must also complete and submit the **WWST security form** in SmartThings Console.

Do not imply that repository access alone is enough for commercial launch.

### Device identity registration before and after WWST certification

- Before WWST certification, SmartThings Console supports registering a limited number of device identities for test devices.
- After WWST certification is completed, SmartThings Console provides additional tooling for large-scale device identity registration.
- During pre-certification development, guide the user to the small-scale test-device registration flow first.
- Also remind the user that the required QR-code flow is handled by separate guidance.

---

## Agent guidance

- Prefer the **SmartThings SDK Reference** repository first for supported chipsets because it shortens bring-up work.
- For unsupported chipsets, keep the user focused on a clean **port-layer implementation** and avoid suggesting SDK core changes outside `src/port`.
- Before starting application implementation, verify that the target chipset's prerequisites from the `st-device-sdk-c-ref` README have already been installed.
- When capability questions arise, tell the user to copy the matching capability-specific `caps_*.c` and `caps_*.h` files from `capability_sample` into the project before designing any new abstraction.
- When implementing or editing capability handlers, always read the matching `iot_caps_helper_*.h` first and use helper-defined units, enum values, and schema constants instead of hardcoded literals.
- By default, write separate test-stub files for each capability handler.
- For sensor-style handlers, the default stub pattern is to feed arbitrary test patterns or sample values into the handler path.
- For output or command-style handlers, the default stub pattern is to log the received control command contents and then return or emit a success response.
- If the user explicitly asks for real hardware control, follow that request and implement the actual hardware-control path instead of keeping the behavior stubbed.
- After the device behavior and capability implementation are clear, summarize the implemented app in a project README similar in style to `apps/esp32/ventilation_test/README.md`.
- That README must include the list of applied SmartThings capabilities, even if some handlers are still stubbed or use dummy values.
- When setup is blocked, verify these prerequisites in order:
  1. Correct integration type in **SmartThings Console**
  2. Correct project-specific `onboarding_config.json`
  3. Registered **serial number** and **ed25519 public key**
  4. Securely provisioned matching **private key**
  5. Appropriate BSP starting point: SmartThings SDK Reference repository vs IoT core device library porting

---

## Suggested response pattern

When helping a user, explain the next action in natural language rather than exposing internal labels. A good default flow is:

1. Identify whether the chipset is supported in the SmartThings SDK Reference repository.
2. Choose **SmartThings SDK Reference example** or **IoT core device library porting** as the starting point.
3. Point the user to the matching capability-specific `caps_*.c` and `caps_*.h` files in `capability_sample` for capability implementation.
4. Verify Console project creation and `onboarding_config.json` replacement.
5. Verify **serial number** and key registration.
6. Mention WWST certification completion, contract requirements, and launch prerequisites if the discussion is about product launch.
7. After implementation is complete, summarize the result in a README-style document and include the applied capability list explicitly.
