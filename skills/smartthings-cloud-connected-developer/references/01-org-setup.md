# Step 1: Organization (ORG) and Brand Setup Guide

This step sets up the foundational development environment (Developer Console) for integrating with the SmartThings platform.

## Goals
- Establish an **Organization**, which is the entity that will proceed with Works With SmartThings (WWST) certification.
- Set up a **Brand** under which products (devices) will belong.

## Development Guide and Instructions

> **[Console Task Notice]**
> This step is performed through the console GUI. If the user is experiencing difficulties or wants help, first ask *"Shall I open the browser and we can proceed together?"* Only enter the Login-Assist Flow if the user wants it.

1. **Verify/Create Organization**
   - Development and WWST certification must be conducted at the Organization level, not under a personal account.
   - Ask the user whether they currently have an organization set up.
   - If not set up, guide them through the creation process.
     - Guide URL: [SmartThings Console - Organization](https://developer.smartthings.com/console/organization)
     - Inform the user that they can change the organization they are currently working under via the profile in the upper right corner of the console (`USING CONSOLE AS`).
     - If an existing organization exists, inform them that they can request a team invitation (invitation email) from the organization Admin.

2. **Verify/Create Brand**
   - Devices must be registered under a specific brand.
   - Ask the user whether the brand their product belongs to is registered, and if not, instruct them to create a brand in the console.
     - Guide URL: [SmartThings Console - Brands](https://developer.smartthings.com/console/brands)

3. **Requirements Check**
   - Remind the user that this step requires their own OAuth2 authentication server. (A mandatory requirement for ST-Schema)
     - Authentication server guide: [Auth Server Guide](https://developer.smartthings.com/docs/devices/cloud-connected/auth-server)

Once this step is complete (when the user confirms completion), the agent should inform them that they will proceed to Step 2 (02-device-profile.md) to define the device's detailed specifications.