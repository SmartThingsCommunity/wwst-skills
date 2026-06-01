# Z-Wave WWST references

Use this file when the task involves Z-Wave WWST validation or Z-Wave-specific platform limits.

## Z-Wave notes

- Treat Z-Wave as a **hub-connected** flow in Console. Follow **Publish hub-connected devices** and **Test Suite**.
- Validate **hub compatibility**, **region**, and **S2 / inclusion** behavior on real hardware.
- If the device depends on manufacturer-specific or non-standard handling, treat it as a **Custom** path and confirm the supported approach through official or partner guidance.
- For any custom implementation beyond a fingerprint-only change, reuse existing Edge driver packages, profiles, and sub-drivers first. Avoid creating a standalone driver package when the device can fit into an existing package.
- Prefer extending the closest existing sub-driver rather than creating a new top-level driver package.
- First look for a manufacturer-specific sub-driver when the same manufacturer already has device handling in the target driver package.
- If there is no manufacturer-specific fit, look for a functional sub-driver that already models the same device shape or behavior.
- Examples in SmartThingsEdgeDrivers include manufacturer-oriented paths such as `zwave-switch/src/inovelli` or `zwave-switch/src/qubino-switches`, and function-oriented paths such as `zwave-button/src/zwave-multi-button` or `zwave-window-treatment/src/window-treatment-venetian`.
- Keep any implementation guidance tied to official or partner direction; this skill does not define Z-Wave codework details.
