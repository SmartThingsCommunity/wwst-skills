---
name: smartthings-hub-connected-developer
description: Guides commercial SmartThings Hub Connected product integrations for manufacturers or partners pursuing Works With SmartThings (WWST) certification for Zigbee, Matter, or Z-Wave devices. Use for hub-side WWST registration, validation, Test Suite, Integration Details, Standard vs Custom path decisions, and Zigbee Edge driver PR guidance when manufacturer-side work is required for certification. Do not use for hobby-only driver experiments, Cloud Connected, Direct Connected, or Mobile Connected integrations.
metadata:
  version: "2026-05-29"
---

# Hub-connected SmartThings Developer (Zigbee, Matter, Z-Wave)

## When to use

- Use this skill for **commercial SmartThings Hub Connected products** whose manufacturer or partner is preparing **WWST certification** and product launch.
- Use this skill only for **SmartThings Hub Connected** device integrations.
- Apply it when the product joins SmartThings through a **hub-side protocol path** such as **Zigbee**, **Matter**, or **Z-Wave**, and the task is about **WWST registration**, **validation**, **Test Suite**, **Integration Details**, **device configuration**, or **hub-side implementation fit**.
- Use it when you need to decide whether the device can stay on the **Standard** path or requires the **Custom** path.
- For detailed codework, this skill expands only the **Zigbee + SmartThingsEdgeDrivers** worked example.
- Do not use this skill for **hobby-only driver experiments**, **Cloud Connected (ST-Schema / Cloud Connector)**, **Direct Connected**, or **Mobile Connected** integrations.

---

## Scope

This skill owns:

- Commercial certification readiness for manufacturer or partner products, not hobby-only driver experiments.
- Hub-connected WWST flow in **SmartThings Console**, including **Test Suite** and **Integration Details**.
- Path selection between **Standard** and **Custom** for **Zigbee**, **Matter**, and **Z-Wave** hub-connected products.
- Profile and device-configuration checks when standard onboarding does not expose all required functionality.
- Detailed worked-example guidance only for **Zigbee + SmartThingsEdgeDrivers** when manufacturer-side implementation is required.

This skill does not own:

- **Cloud Connected** integration design such as **Cloud Connector / ST-Schema**, OAuth, hosting, or cloud registration.
- **Direct Connected** integration work such as **SmartThings Device SDK for C**, MQTT-based firmware, onboarding assets, or device identity provisioning.
- **Mobile Connected** device flows.
- Detailed custom implementation procedures for **Matter** or **Z-Wave**.

---

## Core assumptions

This skill is intended to help an IoT device manufacturer connect **Zigbee, Matter, or Z-Wave products** to **SmartThings** and complete **WWST certification**.

- In **most standard cases**, the flow finishes through **SmartThings Console** registration, hub validation, Test Suite, and certification submission without additional codework.
- A **custom implementation case** is one where the manufacturer's device implementation is not fully described by the protocol **standard model**, so SmartThings **standard Capability mapping** alone may not be enough to produce the expected device experience, and extra manufacturer-side implementation or submission artifacts may be needed.
- This skill explains the **detailed implementation workflow using Zigbee as the worked example**, but it does **not** assume that only Zigbee can require custom handling.

## Assumption for the Zigbee worked example

For the Zigbee implementation example in this skill, the device is expected to work correctly within the **Lua Edge driver** environment used by the [SmartThingsEdgeDrivers](https://github.com/SmartThingsCommunity/SmartThingsEdgeDrivers) repository. The certification and release flow in this worked example follows that assumption.

---

## Agent-only: internal path evaluation

Describe the user's situation and next action in natural language rather than exposing internal branch labels.

| Radio | Internal check |
|--------|----------------|
| **Zigbee** | Do **Manufacturer Specific Clusters** or similar behavior require extra mapping into SmartThings **standard Capabilities**? Does onboarding alone satisfy all app and routine requirements? |
| **Matter** | Does the device stay within the normal **Matter standard onboarding** path, and is SmartThings **standard Capability mapping** enough to deliver the needed UX? |
| **Z-Wave** | On supported hubs and regions, are normal inclusion and **CC** combinations enough under the existing SmartThings **Capability mapping**? Does manufacturer-specific behavior require additional work? |

**Internal mapping (for agent reasoning):**

- **Standard path:** the protocol **standard model** and SmartThings **standard Capability mapping** are enough for the required behavior -> validation, Test Suite, **Integration Standard**.
- If the device uses the protocol **standard model** but the **Standard path** does not expose all required functionality, first use **SmartThings Console -> Integration Details -> Find an Edge Driver** to select the certification-target device among the devices onboarded to your account, then use **Modify Device Configuration** to check whether applying a different profile exposes the missing behavior.
- If changing the profile through **Modify Device Configuration** still does not expose all required functionality, move to the **Custom path** and prepare an Edge driver implementation and **Pull Request** for the device.
- **Custom path:** the protocol **standard model** alone is not enough, or SmartThings **standard Capability mapping** alone is not enough -> evaluate **Integration Custom**, and prepare PR URLs or partner-coordination artifacts if needed.
- **Current Zigbee worked example:** the codework example in this skill is only expanded using **Zigbee + SmartThingsEdgeDrivers**.
- **Reference routing:** read [reference/common.md](reference/common.md) for shared WWST flow, then open only the protocol-specific reference that matches the task: [reference/zigbee.md](reference/zigbee.md), [reference/matter.md](reference/matter.md), or [reference/zwave.md](reference/zwave.md).

If the situation is unclear, ask a clarifying question.

---

## Common path - when no extra codework is needed

**Applies when:** the protocol **standard model** plus SmartThings **standard Capability mapping** is enough for the device to behave correctly in **Zigbee**, **Matter**, or **Z-Wave**.

- **Core flow:** register the product, brand, and device information in **SmartThings Console**, connect the device to a supported hub, validate **Device UI** and **routines**, run **Test Suite**, and submit for certification.
- If the device uses the protocol **standard model** but the initially selected profile does not expose all required functionality, first use **Integration Details -> Find an Edge Driver -> Modify Device Configuration** to verify whether a different profile exposes the missing behavior while staying on the **Standard** path.
- **Certification:** submit through [SmartThings Console - Test Suite](https://developer.smartthings.com/console/test) and choose **Standard** in **Integration Details** where appropriate ([Publish hub-connected devices](https://developer.smartthings.com/docs/devices/hub-connected/certify-your-device/)).

---

## Common path - when manufacturer-side implementation is required

- If the device implementation is not fully captured by the protocol **standard model**, or if the protocol **standard model** is used but the **Standard** path still does not expose all required functionality after profile validation in **Modify Device Configuration**, first evaluate whether it can still be mapped into SmartThings [Capabilities](https://developer.smartthings.com/docs/devices/capabilities/) using a **Production** or **Proposed Capability**.
- **WWST** must not include **Custom Capabilities**, so user-facing functionality should be expressed through SmartThings **Standard Capabilities**.
- For any Zigbee or Z-Wave custom implementation beyond a fingerprint-only change, prefer the smallest reuse of existing Edge driver package, profile, and sub-driver structure; do not guide the user toward a standalone driver package for a commercial WWST submission when an existing package can be extended.
- In these cases, certification often needs the **Integration Custom** path, and the manufacturer may need to prepare additional supporting materials such as PR URLs or partner-coordination artifacts.
- The exact codework differs by protocol. This skill continues with a **Zigbee worked example**.

---

## Zigbee worked example: reflecting device support in SmartThingsEdgeDrivers

Use this path only when the hub-connected **Custom** flow needs Zigbee manufacturer-side implementation.

- Keep the decision at the WWST level in this file: can the device stay on standard onboarding, or does it need Zigbee custom implementation?
- Before implementation, verify that the required behavior can be expressed through SmartThings **Production** or **Proposed Capabilities**. Do not guide the user toward **Custom Capabilities** for WWST.
- If implementation is needed, steer toward minimal reuse of an existing manufacturer or functional sub-driver before considering new structure.
- For Zigbee Edge driver implementation details such as **`lua_libs`**, **`test/`**, **fingerprints**, **profiles**, **`sub_driver`** structure, and PR preparation, open [reference/zigbee.md](reference/zigbee.md).

---

## Matter

**Standard path:** if the device can stay on the normal **Matter standard onboarding** path and SmartThings can map the exposed behavior into its **standard Capabilities** with the required UX and automation behavior, follow the normal Console-centered WWST path.

- **Certification:** follow [Certification with CSA (Matter)](https://developer.smartthings.com/docs/certification/certification-with-csa).
- **Onboarding:** complete **Matter commissioning** in the app and hub, then validate the expected device behavior and scenarios. In this skill, the key question is whether the resulting SmartThings behavior fits the **standard path**, not whether the partner is responsible for low-level Matter standard modeling details.
- **Profile selection:** when the device stays on the **standard path** and the target Matter base driver already supports a **modular profile**, choose that modular profile rather than falling back to a non-modular alternative. For example, if the base driver supports `matter-switch/profiles/fan-modular.yml`, prefer that modular profile for the matching standard device shape.
- If the standard Matter implementation still does not expose all required functionality, use **Integration Details -> Find an Edge Driver -> Modify Device Configuration** to verify whether a different supported profile resolves the gap before moving to the **Custom** path.
- **Manufacturer-side implementation:** if the device depends on **custom clusters** or **vendor-specific extensions**, evaluate the **Custom** path instead. This skill does not expand Matter-specific codework details; use CSA, silicon-vendor, and partner guidance.

---

## Z-Wave

**Standard path:** if the device can be included on supported hubs and regions and the existing SmartThings **Capability mapping** exposes the required functionality, follow the normal Console-centered WWST path.

- **Validation:** verify inclusion, re-inclusion, **S2**, and app/routine behavior.
- If the device uses standard **CC** behavior but the selected profile does not expose all required functionality, use **Integration Details -> Find an Edge Driver -> Modify Device Configuration** to test whether another supported profile resolves the gap before moving to the **Custom** path.
- **Certification:** use [Publish hub-connected devices](https://developer.smartthings.com/docs/devices/hub-connected/certify-your-device/) and **Test Suite**. Some categories may involve an **Authorized Test Provider**; rely on the latest official guidance.
- **Manufacturer-side implementation:** if additional handling is needed because of **Z-Wave LR**, **SmartStart**, proprietary **CCs**, or manufacturer-specific behavior, evaluate the **Custom** path. This skill does not expand Z-Wave-specific codework details.

---

## Common - Device Preferences

- Always review [Device Preferences](https://developer.smartthings.com/docs/devices/preferences) for device-card settings, and keep them aligned with WWST scope, Console configuration, and policy expectations.

---

## Roles and artifacts

| Artifact | Purpose |
|----------|---------|
| **Organization + brand** | Console product setup and certification ownership |
| **Console registration** | Core registration, validation, and certification steps for the standard WWST path |
| **Integration Details** | Submission choice between **Standard** and **Custom** |
| **Test Suite** | Validation before certification submission |
| **Protocol-specific implementation** | Additional mapping, driver work, PR work, and partner coordination prepared by the manufacturer when needed |
| **Edge driver / integration implementation** | Core implementation that connects protocol behavior to SmartThings **Standard Capabilities**. In this skill, the detailed execution example is Zigbee-based |
| **Fingerprint / device matching** | Information used to identify the device, such as manufacturer and model, and associate the joined device with the correct implementation |
| **Developer tooling + test channel** | Tools and deployment channels used to validate unpublished implementations on a real hub |
| **Dev env + `test/`** | Development environment and test assets required before opening a PR |

---

## Phase 0 - Preparation

1. Review [Certification overview](https://developer.smartthings.com/docs/certification/overview) and confirm Console **Organization / Brand** setup.
2. Use official documentation for **Product Policies** and category-specific UX requirements.
3. Apply the internal evaluation above, and explain the current situation and next action in natural language.

---

## Phase 1 - Technical design notes (for the Custom path)

1. First verify whether the protocol **standard model** plus SmartThings **standard Capability mapping** is sufficient.
2. If the device uses the protocol **standard model** but the initially selected profile does not expose all required functionality, verify through **Integration Details -> Find an Edge Driver -> Modify Device Configuration** whether another supported profile resolves the gap while staying on the **Standard** path.
3. If the behavior is still not sufficient, organize what is needed for **Integration Custom**, including PR URLs or partner-coordination artifacts.
4. Follow the protocol-specific reference file before giving implementation guidance.
5. In this skill, only **Zigbee** is expanded as a worked example, and its implementation details live in [reference/zigbee.md](reference/zigbee.md).
6. Do not invent Matter- or Z-Wave-specific custom implementation procedures in this skill.

---

## Phase 2 - Self Test (before certification)

- **When standard onboarding and standard Capability mapping are sufficient:** validate full hub, app, and routine scenarios, then run Test Suite.
- **For the Custom path:** validate the manufacturer-side implementation on real hub and app scenarios, and prepare the supporting materials needed for submission.
- **Zigbee worked example:** use [reference/zigbee.md](reference/zigbee.md) for Edge driver validation details before Console **Test Suite**.

---

## Phase 3 - Certification submission (WWST)

- Follow [Publish hub-connected devices](https://developer.smartthings.com/docs/devices/hub-connected/certify-your-device/).
- For **Matter**, also verify alignment with [Certification with CSA (Matter)](https://developer.smartthings.com/docs/certification/certification-with-csa).
- In **Integration Details**, use **Standard** when onboarding alone is sufficient; use the **Custom** path when additional manufacturer-side implementation or submission artifacts are required.
- In the **Zigbee worked example**, if there is a SmartThingsEdgeDrivers PR, submit with **Custom** + **PR URL**.

---

## Phase 4 - Publishing and maintenance

- Follow the latest Samsung guidance for WWST logo and catalog usage.
- Re-validate when firmware, fingerprints, radio stack behavior, packaging, onboarding materials, or labeling changes.

---

## Agent behavior summary

- **Common:** first decide whether the normal **Console-centered path** is sufficient; only guide the user into the **Custom** path when needed.
- **Zigbee:** internally evaluate whether MSS-like handling or extra mapping is needed, but explain the result to the user as current situation plus next action.
- **WWST guardrail:** never propose **Custom Capabilities** for certified product behavior; map custom or manufacturer-specific behavior to SmartThings **Production** or **Proposed Capabilities** when possible.
- For **Matter** on the **standard path**, prefer an existing base-driver **modular profile** when one is supported for the target device shape.
- For **Zigbee** custom implementation details, use [reference/zigbee.md](reference/zigbee.md) instead of restating Edge Driver workflow in this file.
- For **Z-Wave** custom implementation, keep guidance at the WWST/partner-coordination level and use [reference/zwave.md](reference/zwave.md); preserve the reuse-first rule and avoid standalone driver packages when an existing package can be extended.
- Do **not** describe custom-implementation possibilities as if they were limited to Zigbee. Only the detailed codework example in this skill is Zigbee-specific.
- For **Matter** and **Z-Wave**, do not invent codework details in this skill. Send the user to official or partner guidance for protocol-specific implementation details.
- Always review **Device Preferences**.

## Additional material

- If a companion [reference.md](reference.md) is available for this skill, consult it for supporting reference material.
- Do not read all reference material by default; open only the reference file that matches the current protocol and task.
