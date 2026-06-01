# Zigbee WWST references

Use this file when the task involves Zigbee Edge driver work, Zigbee custom implementation, or the Zigbee worked example in this skill.

## Custom implementation path - Zigbee worked example

1. [SmartThingsEdgeDrivers Releases](https://github.com/SmartThingsCommunity/SmartThingsEdgeDrivers/releases) - download the latest **`lua_libs` `tar.gz`** and extract it
2. [Set up your development environment](https://developer.smartthings.com/docs/devices/hub-connected/set-up-dev-env)
3. [Test your Edge Driver](https://developer.smartthings.com/docs/devices/hub-connected/test-your-driver) - write or update **`test/`** integration tests and run them locally until all pass
4. [SmartThingsEdgeDrivers repo](https://github.com/SmartThingsCommunity/SmartThingsEdgeDrivers)
5. [Code formatting and submission criteria](https://developer.smartthings.com/docs/devices/hub-connected/code-formatting-criteria)
6. Validate on a custom channel and submit through [SmartThings Console - Test Suite](https://developer.smartthings.com/console/test) with **Integration Custom** + PR URL when applicable

## WWST implementation guardrails

- Except for fingerprint-only changes, reuse existing SmartThingsEdgeDrivers packages, profiles, and sub-drivers first. Avoid creating a standalone driver package when the device can fit into an existing package.
- Prefer extending the closest existing sub-driver rather than creating a new top-level driver package.
- First look for a manufacturer-specific sub-driver when the same manufacturer already has device handling in the target driver package.
- If there is no manufacturer-specific fit, look for a functional sub-driver that already models the same device shape or behavior.
- Examples in SmartThingsEdgeDrivers include manufacturer-oriented paths such as `zigbee-switch/src/aqara` or `zigbee-switch/src/frient`, and function-oriented paths such as `zigbee-button/src/zigbee-multi-button` or `zigbee-switch/src/multi-switch-no-master`.
- Keep the PR small: add only the needed fingerprints, profile reuse or minimal profile changes, handlers, and tests for the certification-target model.

## Edge drivers and Zigbee

- [Driver components and structure](https://developer.smartthings.com/docs/devices/hub-connected/driver-components-and-structure/) (fingerprints, profiles, `zigbeeManufacturer`)
- [Edge Device Drivers documentation](https://developer.smartthings.com/docs/edge-device-drivers/)
- [Edge Device Driver reference](https://developer.smartthings.com/docs/edge-device-drivers/reference/index.html)
- [SmartThingsEdgeDrivers (GitHub)](https://github.com/SmartThingsCommunity/SmartThingsEdgeDrivers)
