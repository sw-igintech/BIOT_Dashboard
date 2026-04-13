# Machines Dashboard

This repository now targets a static GitHub Pages frontend plus a single Google Apps Script backend. The frontend keeps the current BIOT/Grafana-inspired layout and charts, but all BIOT API calls now happen only inside Apps Script.

## Final Deployment Path

- GitHub Pages serves:
  - `index.html`
  - `dashboard.css`
  - `dashboard.js`
- Google Apps Script serves:
  - `apps-script/Code.gs`

There is no Flask, FastAPI, Express, or other runtime in the final deployment model.

## Files To Publish

### GitHub Pages

Publish these frontend files:

- `index.html`
- `dashboard.css`
- `dashboard.js`

### Google Apps Script

Copy this exact file into Apps Script:

- `apps-script/Code.gs`

## Apps Script Configuration

Set these Script Properties in the Apps Script project:

- `BIOT_BASE_URL`
  - Use: `https://api.dev.igin.biot-med.com`
- `BIOT_USERNAME`
  - BIOT username for the server-side service account
- `BIOT_PASSWORD`
  - BIOT password for the server-side service account

The Apps Script code reads those values from `PropertiesService.getScriptProperties()`.

## How BIOT Auth Works In This Version

1. GitHub Pages calls the Apps Script Web App only.
2. Apps Script reads BIOT credentials from Script Properties.
3. Apps Script logs into BIOT with `POST /ums/v2/users/login`.
4. Apps Script calls `GET /ums/v2/users/self` to determine role and scope.
5. Apps Script fetches devices and glove events, aggregates them, and returns dashboard JSON.
6. The browser never stores BIOT credentials and never calls BIOT directly.

## Role Handling

### Organization account

- Role is inferred from BIOT self data.
- Scope is locked to the BIOT `ownerOrganizationId`.
- Organization selector is hidden.

### Manufacturer account

- If any BIOT group contains `Manufacturer`, the backend treats the account as manufacturer scope.
- The frontend shows an organization selector.
- Default scope is `All organizations`.
- Organizations are derived from the accessible device owners plus BIOT self payload organization references.

## Why The Frontend Uses JSONP

Google Apps Script Web Apps do not provide normal custom CORS header control for cross-origin GitHub Pages `fetch()` requests. The frontend therefore uses JSONP to call the Apps Script Web App securely without exposing BIOT credentials.

This still keeps the architecture valid:

- GitHub Pages stays fully static
- Apps Script is the only backend
- BIOT communication stays server-side only

## Deploy Apps Script

1. Create a new standalone Google Apps Script project.
2. Open `apps-script/Code.gs` from this repository.
3. Copy the full file contents into the Apps Script editor, replacing the default file contents.
4. In Apps Script, open `Project Settings`.
5. Add Script Properties:
   - `BIOT_BASE_URL`
   - `BIOT_USERNAME`
   - `BIOT_PASSWORD`
6. Deploy the project:
   - `Deploy` -> `New deployment`
   - Type: `Web app`
   - Execute as: `Me`
   - Who has access: `Anyone`
7. Copy the deployed `/exec` Web App URL.

## Wire The Frontend To Apps Script

1. Open `index.html`.
2. Find `window.DASHBOARD_CONFIG.appsScriptUrl`.
3. Paste the deployed Apps Script `/exec` URL there.
4. Publish the frontend on GitHub Pages.

## Returned Dashboard Data

Apps Script returns dashboard-ready JSON including:

- `viewer`
- `scope`
- `organizations`
- `connection`
- `offlineDevices`
- `gloves`
- `sanitizer`
- `meta`

The frontend renders:

1. Device Connection Status
2. Offline Devices
3. Glove Consumption
4. Sanitizer Status
5. Sanitizer device list
6. Filters bar

## BIOT Mappings Used

- Device connection status
  - `GET /device/v2/devices`
  - `_status._connection._connected`
- Offline devices
  - `GET /device/v2/devices`
  - `_id`, `_status._connection._lastConnectedTime`, `_status._connection._connected`
- Sanitizer status
  - `GET /device/v2/devices`
  - `_status.septol_availability1`
- Glove consumption
  - `GET /generic-entity/v3/generic-entities/device_event?searchRequest=...`
  - `event_code = GLOVE_TAKEN`
  - `event_cartridge_size`
  - `_creationTime`
  - `_ownerOrganization.id`

## Limitations Compared To The Previous Python Backend

- This version uses one BIOT account configured in Apps Script, not per-browser BIOT login.
- Manufacturer glove aggregation can be slower because Apps Script loops through organizations and paginates event pages.
- Google Apps Script execution time and quota limits apply.
- Apps Script JSONP is used for cross-origin GitHub Pages calls, so the frontend uses GET requests to the web app.
- BIOT token refresh is not implemented because a refresh endpoint contract was not confirmed.

## Old Backend Files No Longer In The Final Deployment Path

These files are no longer used by the final GitHub Pages + Apps Script deployment:

- `app.py`
- `biot_client.py`
- `dashboard_service.py`
- `requirements.txt`
- `.env.example`
