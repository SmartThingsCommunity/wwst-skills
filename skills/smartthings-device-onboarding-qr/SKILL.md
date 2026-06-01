---
name: smartthings-device-onboarding-qr
description: Explains SmartThings device QR code purpose, security and onboarding role, and type-specific formats for Matter, Zigbee 3.0, Direct Connected (st-device-sdk-c), and Mobile Connected (SmartThings Find Device SDK). Covers Zigbee minimum vs recommended payloads (install code, manufacturer code, OUI), WWST partner notification, official publish docs, partner QR label template link, per-type field checklist, and product-oriented payload examples. Use when the user asks about SmartThings device QR codes, onboarding QR requirements, Zigbee 3.0 install-code QR, Matter CSA QR, Direct Connected or Mobile Connected QR specifications, Find Device SDK, WWST QR registration, or partner label print layout.
metadata:
  version: "2026-05-29"
---

# SmartThings device onboarding QR codes

## When to use this skill

Apply when answering whether a SmartThings compliant product needs a QR code and what format or fields apply, by integration type: **Matter**, **Zigbee 3.0**, **Direct Connected**, **Mobile Connected**.

**Out of scope:** Z-Wave and Cloud Connected QR requirements are not covered here.

**Product-oriented answers:** When a concrete example helps, include **copy-pastable payload strings** in the reply, using the user’s values or clear placeholders (e.g. `YOUR_INSTALL_CODE`) for secrets. For QR **bitmaps**, use a real **QR encoder** (library or trusted tool), not generative image models—artistic or “drawn” codes usually fail to scan reliably.

### Inputs by integration type

| Type | Collect or reference |
|------|----------------------|
| **Matter** | Commissioning flow; **discriminator**, **setup passcode**, **Vendor ID (VID)**, **Product ID (PID)**; any additional fields **CSA Matter** and your **Matter stack / silicon vendor** docs require for the setup or QR payload. |
| **Zigbee 3.0** | **MAC** (64-bit as used in QR), **Install Code**; for recommended QR: **Model Number**, **Manufacturer Code** (or confirm OUI-only omission per [IEEE OUI](https://standards-oui.ieee.org/)). |
| **Direct Connected** | Identifiers and segments named in [Publish a Direct Connected Device](https://developer.smartthings.com/docs/devices/direct-connected/publish) for the partner QR (align with Developer Workspace / doc field names). |
| **Mobile Connected** | Fields required by [Developing your SmartThings Find device](https://developer.smartthings.com/docs/devices/mobile-connected/developing-your-find-device) and **Find Device SDK** / partner materials. |

## Why QR codes matter

- **Security:** Help establish protected communication (e.g. encrypted channel setup where the standard requires it).
- **Proof of possession:** Tie onboarding to a physical device the user has.
- **Onboarding UX:** User can start pairing by scanning a QR code.

Payload **shape and content differ by integration type**; do not assume one QR format fits all.

---

## Matter

- Follow **CSA Matter** standard QR code rules. **No SmartThings-specific QR format** beyond that standard.
- The **QR payload format and encoding rules** are defined by **CSA Matter**. Use your **Matter stack** documentation only when implementation-specific details are needed.

---

## Zigbee 3.0

Official requirements are cited by **document number** (e.g. **18-01000** *QR Code Requirements*, **05-3874** *Manufacturer Code Database*). Obtain current specifications through **Connectivity Standards Alliance** (Zigbee) documentation channels.

### Minimum (required pattern)

- Satisfy **dotdot** QR requirements per **Zigbee Document 18-01000** (*QR Code Requirements*).
- QR must include **MAC address** and **Install Code**.

Example (minimum-style payload illustration):

```text
Z:0123456789ABCDEF$I:023047432043AF3456FEB23452423423621A
```

### Recommended (beyond minimum)

When possible, include both **manufacturer identity** and **model identity** in the QR. This lets SmartThings identify **which product is being onboarded** more accurately, so the user does not have to pick from a long model list and can be guided with more **product-specific onboarding instructions**.

**On SmartThings:**

- **With model + manufacturer identity (Manufacturer Code and/or OUI as allowed by the Zigbee rules):** After the user scans the QR, **SmartThings** can **determine which product** is being onboarded **from the QR payload**, without making the user pick the exact model from a long in-app list. Onboarding copy and flows that the manufacturer **preregistered in SmartThings Console** can then **bind to that product** and display **accurately**.
- **Minimum-only QR (MAC + Install Code, no usable model/manufacturer signal):** The platform often **cannot pinpoint a specific SKU**, so it can show **only generic** guidance—not **product-specific** Console content tuned for that device.

That **simplifies the user journey**: **unbox the device → scan the QR → follow product-optimized in-app content** for the remaining steps (instead of manual catalog matching).

### Ways to express manufacturer identity

Explain manufacturer identity in this order:

1. **Use a CSA-registered Manufacturer Code after `%M`**
   Treat `%M` as the field marker used to carry the manufacturer code. The value after `%M` should be the manufacturer's unique code listed in **Zigbee Document 05-3874** (*Manufacturer Code Database*). If the manufacturer does not have its own code, this method is not available.
2. **Use the MAC address OUI**
   If the Zigbee rules allow omitting `%M`, and the first 3 bytes of the MAC (**OUI / MA-L**) reliably identify the commercial manufacturer of the product, the OUI can be used instead. This is not a good fit for **OEM / multi-brand** situations where the OUI does not uniquely represent the sellable product identity.
3. **Use a Samsung-issued manufacturer identifier (MNID) through a SmartThings partner procedure**
   If the standard manufacturer-identity approaches above are not enough for the intended onboarding UX, a **Samsung-issued manufacturer identifier (MNID)** may be handled through a separate **SmartThings partner procedure**. This is not the same thing as a standard Zigbee QR field, so direct partners to **partner@smartthings.com** for the exact process.

### How to express model identity

Explain model identity as **Model Number** carried in **`$M`** under the **Global Parameter (`G`)** section.

**Uniqueness (OEM / multi-brand):** The **manufacturer + product identity** encoded in the QR (Manufacturer Code, OUI, or SmartThings partner-managed manufacturer identity, plus **Model Number**) should map to **one distinct commercial product**. If the **same underlying device** is sold under **different brands or model names** with **identical** manufacturer/model identifiers in the QR, SmartThings **cannot** tell those SKUs apart from the payload alone. In that case, each sellable variant needs its own distinguishing metadata.

Example:

```text
G$M:BOX%Z$A:0123456789ABCDEF$I:023047432043AF3456FEB23452423423621A%M:10E1
```

Interpretation:

| Field | In example | Notes |
|--------|------------|--------|
| Model Number | `BOX` | Carried under **Global Parameter (G)** as **$M** (Model Number). |
| Manufacturer Code | `10E1` | Manufacturer code placed after **`%M`**; must exist in **Zigbee Document 05-3874** (*Manufacturer Code Database*). Use the **device OEM’s** assigned code; **do not** use the chipset vendor’s code as a substitute. |

### Omitting Manufacturer Code

If the **first 3 bytes of the MAC** (OUI / MA-L) are already the manufacturer’s **standard** assignment, **Manufacturer Code may be omitted**.

Example (no `%M:` when OUI is sufficient):

```text
G$M:BOX%Z$A:0123456789ABCDEF$I:023047432043AF3456FEB23452423423621A
```

Here `012345` in `0123456789ABCDEF` is the OUI/MA-L portion; verify assignments at [IEEE OUI public listing](https://standards-oui.ieee.org/).

### After WWST (recommended QR)

If the product follows the **recommended** Zigbee QR pattern, or also needs an **MNID-based SmartThings partner procedure**, after **WWST** completion send the details to **partner@smartthings.com** so SmartThings can **store** the metadata and use it for that product’s **onboarding UX**.

---

## Direct Connected and Mobile Connected

Both paths require a **partner QR code** for publishing; format and fields follow the **SmartThings developer documentation** for that path (Direct: publish guide; Mobile: Find / Mobile Connected guides). The **product technology** differs.

**Partner label (print / packaging):** For sticker or box artwork, SmartThings publishes **Illustrator-oriented** assets and layout notes under **[QR code design template](https://developer.smartthings.com/docs/devices/direct-connected/publish#qr-code-design-template)** on the Direct Connected publish page (e.g. last four serial digits above the QR, SmartThings branding below—per guidance on that page). Use that **documentation anchor** for links.

### Direct Connected

- Typical products use **[SmartThings SDK for Direct Connected Devices (C)](https://github.com/SmartThingsCommunity/st-device-sdk-c)** — cloud-oriented device firmware (e.g. customized MQTT, onboarding APIs) integrated with the SmartThings Cloud.
- **QR code is mandatory.** Follow the **SmartThings partner device QR specification** for publishing.
- Source of truth: [Publish a Direct Connected Device](https://developer.smartthings.com/docs/devices/direct-connected/publish).

### Mobile Connected

- Typical products use the **SmartThings Find Device SDK** (Find / BLE-UWB-style mobile integration; partner access and materials are managed through the Find partnership program).
- **QR code is mandatory** under the same **partner publishing** expectations; use SmartThings **Mobile Connected** / Find device documentation for your product’s QR and workflow.
- Orientation: [Developing your SmartThings Find device](https://developer.smartthings.com/docs/devices/mobile-connected/developing-your-find-device); partner program overview: [SmartThingsFindPartnershipProgram](https://github.com/SmartThingsCommunity/SmartThingsFindPartnershipProgram) (access is partnership-gated — contact **partners@smartthings.com** per that program).
- **Label artwork** uses the same **[QR code design template](https://developer.smartthings.com/docs/devices/direct-connected/publish#qr-code-design-template)** section as Direct Connected for recommended partner layout.

---

## Quick reference

| Integration | QR required? | Primary rule |
|-------------|--------------|--------------|
| Matter | Per CSA Matter | CSA + stack/vendor docs only; no extra SmartThings QR spec |
| Zigbee 3.0 | Yes (minimum: MAC + Install Code per 18-01000) | Recommended: + model + manufacturer code (05-3874); optional `%M` if OUI proves OEM |
| Direct Connected | **Yes** | **st-device-sdk-c** class products; [publish / QR spec](https://developer.smartthings.com/docs/devices/direct-connected/publish); print: [QR code design template](https://developer.smartthings.com/docs/devices/direct-connected/publish#qr-code-design-template) |
| Mobile Connected | **Yes** | **Find Device SDK** class products; Mobile Connected / Find docs + partner program; print: [QR code design template](https://developer.smartthings.com/docs/devices/direct-connected/publish#qr-code-design-template) |
| Z-Wave / Cloud | — | Not covered in this skill |
