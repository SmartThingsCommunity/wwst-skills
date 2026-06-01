# WWST Certification Checklist

This document provides a checklist for SmartThings WWST (Works With SmartThings) certification.

## Table of Contents

1. [Pre-Certification Preparations](#pre-certification-preparations)
2. [Feature-by-Feature Checklist](#feature-by-feature-checklist)
3. [Regional Testing](#regional-testing)
4. [STTS Test Suite](#stts-test-suite)
5. [How to Apply for Certification](#how-to-apply-for-certification)
6. [Post-Certification Management](#post-certification-management)

---

## Pre-Certification Preparations

### Required Items

- [ ] SmartThings Developer account created
- [ ] Organization created or joined
- [ ] Brand registered
- [ ] Product registered
- [ ] Device Profile created (Optional)
- [ ] SmartThings Schema (Schema App) implemented
- [ ] SmartThings Schema (Schema App) registered
- [ ] Hosting environment set up

### Document Preparation

- [ ] Product image (transparent background, minimum 584x584px)
- [ ] Product description document
- [ ] Product number (M/N, EAN, UPC, SKU, etc.)
- [ ] Purchase link (optional)
- [ ] Supported regions list

---

## Feature-by-Feature Checklist

### 1. OAuth Authentication

- [ ] OAuth 2.0 authentication server working properly
- [ ] Authorization Code Grant supported
- [ ] Refresh Token supported
- [ ] Correct Redirect URI settings
  - [ ] US: `https://c2c-us.smartthings.com/oauth/callback`
  - [ ] EU: `https://c2c-eu.smartthings.com/oauth/callback`
  - [ ] AP: `https://c2c-ap.smartthings.com/oauth/callback`
- [ ] Re-authentication works when token expires
- [ ] Logout/integration removal works properly

### 2. Discovery (Device Discovery)

- [ ] Device list returned normally
- [ ] All device types displayed normally
- [ ] Device name (friendlyName) displayed normally
- [ ] Device icon displayed normally
- [ ] Offline devices handled appropriately
- [ ] Large number of devices (100+) can be processed

### 3. StateRefresh (State Query)

- [ ] Device state queried normally
- [ ] All Capability states returned normally
- [ ] State value formats correct (string, number, boolean)
- [ ] Offline device state handling
- [ ] Response time within 5 seconds
- [ ] Multi-device state query supported

### 4. Command (Device Control)

- [ ] Power control (on/off) works normally
- [ ] Level control (brightness, etc.) works normally
- [ ] Color control works normally (if supported)
- [ ] Mode control works normally (if supported)
- [ ] State update after command execution normal
- [ ] Error response on command failure normal
- [ ] Consecutive command processing supported

### 5. Health Check

- [ ] Device online/offline status displayed
- [ ] healthCheck Capability implemented
- [ ] Offline devices distinctly displayed
- [ ] Timeout handling appropriate

### 6. Callback (State Change Notification)

- [ ] callbackAccessHandler implemented
- [ ] Callback URL saving functionality
- [ ] Callback sent on state change
- [ ] Callback authentication working normally
- [ ] Multi-user callback supported

### 7. Integration Deleted

- [ ] integrationDeletedHandler implemented
- [ ] Data cleanup on integration removal
- [ ] Token invalidation processing

---

## Regional Testing

SmartThings is serviced in multiple regions, so testing for each region is required.

### Regional Test Items

#### US Region
- [ ] Change Samsung account to US region
- [ ] Integration works normally
- [ ] Device discovery normal
- [ ] State query normal
- [ ] Command control normal
- [ ] Callback works normally

#### EU Region
- [ ] Change Samsung account to European region (e.g., Germany)
- [ ] Integration works normally
- [ ] Device discovery normal
- [ ] State query normal
- [ ] Command control normal
- [ ] Callback works normally

#### AP Region
- [ ] Change Samsung account to Asian region (e.g., Korea)
- [ ] Integration works normally
- [ ] Device discovery normal
- [ ] State query normal
- [ ] Command control normal
- [ ] Callback works normally

### Regional Test Accounts

- You must create separate Samsung accounts for each region you wish to test.
- Changing the region of an existing Samsung account is not recommended and may cause integration issues.

---

## STTS Test Suite

The SmartThings Test Suite (STTS) is an automated certification testing tool.

### How to Run STTS

1. Access SmartThings Console
2. Select the project
3. Enter the Test Suite menu
4. Run the test

### STTS Test Items

#### Basic Tests
- [ ] Discovery Test
  - Device list query
  - Response format validation

- [ ] State Refresh Test
  - State query
  - State value validation

- [ ] Command Test
  - Basic commands (on/off)
  - State update validation

#### Capability-Specific Tests

**switch**
- [ ] on command normal
- [ ] off command normal
- [ ] State synchronization normal

**switchLevel**
- [ ] setLevel command normal
- [ ] Level range (0-100) normal

**colorControl**
- [ ] setColor command normal
- [ ] setHue command normal
- [ ] setSaturation command normal

**colorTemperature**
- [ ] setColorTemperature command normal
- [ ] Color temperature range normal

**thermostat**
- [ ] setThermostatMode command normal
- [ ] setCoolingSetpoint command normal
- [ ] setHeatingSetpoint command normal
- [ ] Temperature range normal

**lock**
- [ ] lock command normal
- [ ] unlock command normal
- [ ] Lock state synchronization normal

**Sensor Capabilities**
- [ ] temperatureMeasurement state normal
- [ ] relativeHumidityMeasurement state normal
- [ ] motionSensor state normal
- [ ] contactSensor state normal

### STTS Result Confirmation

- PASS: Test passed
- FAIL: Test failed (fix required)
- SKIP: Capability not applicable

All tests must PASS to apply for certification.

---

## How to Apply for Certification

### 1. Verify Certification Requirements

- [ ] All STTS tests passed
- [ ] All documents prepared
- [ ] 3 regional tests completed
- [ ] All feature checklist items completed

### 2. Apply for Certification

1. Access SmartThings Console
2. Select the project
3. Enter the Certification menu
4. Complete the certification application:
   - Confirm product information
   - Confirm test results
   - Enter additional information (if needed)
5. Submit application

### 3. Certification Review

- SmartThings team conducts manual review
- May request additional information
- Approval or rejection decision

### 4. Certification Complete

- Upon certification approval, the Works With SmartThings logo can be used
- Product is published in the SmartThings App
- CbS (Certification by Similarity) program becomes available

---

## Post-Certification Management

### Maintenance

- [ ] Maintain API compatibility
- [ ] Maintain server availability (99.9% target)
- [ ] Apply security updates
- [ ] Monitor errors

### Handling Changes

When there are changes to the product:

1. **Minor Change**
   - UI changes
   - Internal logic improvements
   - No separate certification required

2. **Major Change**
   - New Capability added
   - API structure changes
   - Re-certification required

3. **Breaking Change**
   - Existing devices no longer supported
   - Full re-certification required

### CbS (Certification by Similarity)

Simplified procedure when adding products similar to an already certified product:

**Application Conditions:**
- Uses the same Schema App
- Uses the same Device Profile
- Uses the same API structure
- Only brand/model name differs

**Benefits:**
- Free certification
- Simplified testing
- Faster approval

---

## Checklist Summary

### Final Pre-Certification Check

| Item | Status |
|------|--------|
| OAuth authentication normal | ☐ |
| Discovery normal | ☐ |
| StateRefresh normal | ☐ |
| Command normal | ☐ |
| Callback normal | ☐ |
| Health Check normal | ☐ |
| US region testing complete | ☐ |
| EU region testing complete | ☐ |
| AP region testing complete | ☐ |
| STTS tests passed | ☐ |
| Documents prepared | ☐ |

### Common Problems and Solutions

| Problem | Cause | Solution |
|---------|-------|----------|
| STTS Discovery failure | Response format error | Validate JSON schema |
| STTS Command failure | State not reflected | Query state after command |
| Regional failure | Callback URL mismatch | Allow callbacks for all regions |
| Timeout failure | Response delay | Keep response within 5 seconds |
| Certification rejected | Incomplete documents | Check required documents |

---

## Reference Links

- [WWST Certification Guide](https://developer.smartthings.com/docs/certification)
- [STTS Test Suite](https://developer.smartthings.com/docs/certification/test-suite)
- [Required Capabilities](https://developer.smartthings.com/docs/certification/required-capabilities)
- [CbS Program](https://developer.smartthings.com/docs/certification/certification-by-similarity)
- [SmartThings Console](https://developer.smartthings.com/console)