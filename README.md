# Machines Dashboard

This project is now a static GitHub Pages dashboard backed by a single Google Apps Script web app. The frontend is already wired to this deployed Apps Script endpoint:

`https://script.google.com/macros/s/AKfycbwtCs6khyngFpoq7UwuSZpVcz6Y600uZIHjnUzXVF7uzdE6z1wok9fsvquzDm_U5Wen/exec`

## Final Deployment Files

GitHub Pages:
- `index.html`
- `dashboard.css`
- `dashboard.js`

Google Apps Script:
- `apps-script/Code.gs`

## What Still Needs To Be Configured

Only the Apps Script project configuration is still manual.

Set these Script Properties in the Apps Script project:
- `BIOT_BASE_URL=https://api.dev.igin.biot-med.com`
- `BIOT_USERNAME=your BIOT username`
- `BIOT_PASSWORD=your BIOT password`

If those values are already configured in the deployed Apps Script project, nothing else is needed from the frontend side.

## Apps Script Setup

1. Create or open the Apps Script project.
2. Copy `apps-script/Code.gs` into the project.
3. Set the Script Properties listed above.
4. Deploy as a Web App.
5. Use access settings that allow the GitHub Pages site to call the deployed `/exec` URL.

## Frontend Setup

Publish these files to GitHub Pages:
- `index.html`
- `dashboard.css`
- `dashboard.js`

The Apps Script URL is already set in `index.html`.

## BIOT Behavior In This Version

- BIOT credentials stay only in Apps Script.
- The browser never calls BIOT directly.
- Organization users are locked to their own organization.
- Manufacturer users get an organization selector and can view all accessible organizations.
- Glove data is fetched from Generic Entity V3 `device_event` with pagination.

## Old Backend Files No Longer Used

These files are no longer part of the final deployment path:
- `app.py`
- `biot_client.py`
- `dashboard_service.py`
- `requirements.txt`
- `.env.example`
