# CivicFix Smart Infrastructure Monitoring

This bundle adds live notifications on the engineer dashboard when the ESP32 telemetry API creates automatic complaints.

## Files
- `app.py`: Flask backend and ESP32 telemetry API
- `templates/`: web pages, including live automatic-complaint notification on the engineer dashboard
- `static/`: CSS/assets
- `firmware/Smart_Infrastructure_Monitor.ino`: ESP32 firmware example
- `requirements.txt`, `render.yaml`, `Procfile`, `runtime.txt`: Render deployment configuration

## Deploy the website update
1. Upload/replace the project files in the same GitHub repository connected to Render.
2. Ensure `templates/engineer_dashboard.html` and `app.py` are both updated.
3. Commit the changes and wait for Render to redeploy.
4. In Render Environment, set `ESP32_DEVICE_TOKEN` to a new secret token (the previous token was shared in chat, so rotate it).
5. Put that same new token in the firmware constant `CIVICFIX_DEVICE_TOKEN`.
6. Confirm the firmware API URL ends with `/api/esp32/telemetry`, configure the actual site address/coordinates, then upload the sketch to ESP32.
7. Log in as an Engineer and keep the **All Complaints** dashboard open. It polls every 5 seconds and shows a notification when a new ESP32 complaint is created, then reloads to display it in the table.

## Trigger conditions in the current firmware
- Street light: detected as damaged only at night when the digital LDR does not detect light.
- Manhole: automatic complaint alert at fill level >= 50% (half-filled).
- Drainage: sensor output LOW means full.
- Pipeline leakage: flow difference must exceed `LEAK_DIFF_THRESHOLD` and at least one flow must be above 0.3 L/min.

## Security and data
- Never commit `.env`, Wi-Fi passwords, or device tokens.
- `complaints.db` is not included. Render's ephemeral filesystem may not retain SQLite data across restarts/redeploys; use a persistent disk or PostgreSQL for durable storage.


## Important: keep accounts and complaints after restarts

The default local SQLite file is `complaints.db` beside `app.py`. On Render, the service filesystem can be ephemeral. For durable data, attach a **Render Persistent Disk** to the web service and mount it at `/var/data` (Render may require a paid service plan for persistent disks). Then open **Render Dashboard → your `civicfix-complaint` service → Environment** and set:

- `DATABASE_PATH` = `/var/data/complaints.db`
- `UPLOAD_FOLDER` = `/var/data/uploads`

Save changes and redeploy/restart the service. The app uses the persistent database path from then on. If the old `complaints.db` file is still present beside `app.py` at that deployment, the app copies it to `/var/data/complaints.db` once, only if the persistent file does not already exist. It never overwrites an existing persistent database.

**Important data-preservation warning:** This code cannot recover records that have already been erased from Render's temporary filesystem. If the old data is still visible before changing settings, export/backup it first and avoid deleting/recreating the service. After attaching the disk, verify that the old accounts and complaints appear before doing any further redeploy. Keep a separate backup of the database. If the existing database is already gone, restore it from a backup; the new persistence setup only protects future data.

A laptop restart alone should not erase data stored on Render. If it disappears exactly when the laptop restarts, confirm that the website is still being accessed at the same live Render URL and that you are not accidentally running a local copy.

## Citizen-friendly interface update

- Added a refreshed responsive visual theme and a language selector for English, Hindi and Marathi.
- Added a floating CivicFix Help chat panel with quick actions and multilingual guidance for submitting complaints, tracking status, using location, and account access.
- The help assistant is a lightweight in-browser FAQ assistant; it does not send messages to an external AI service and does not access or change complaint records.
- Language preference is saved in the visitor's browser. Some complaint/user-generated text and less-common labels may remain in the language in which they were entered; this is a prototype translation layer, not a full professional translation system.
