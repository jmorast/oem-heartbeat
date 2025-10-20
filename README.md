# oem-heartbeat
Dead man’s switch heartbeat for Oracle Enterprise Manager (OEM), designed for cron + external services like Healthchecks.io.

## Setup oem-heartbeat

Preq: Create dead mans alarm in healthchecks.io / cronitor / better stack

1. Copy the script to `/usr/local/bin` and make it executable:

   ```bash
   curl -sSLO https://raw.githubusercontent.com/jmorast/oem-heartbeat/v1.0.4/oem-heartbeat
   curl -sSLO https://raw.githubusercontent.com/jmorast/oem-heartbeat/v1.0.4/oem-heartbeat.sha256

   # Verify the checksum
   sha256sum -c oem-heartbeat.sha256

   sudo cp oem-heartbeat /usr/local/bin/oem-heartbeat
   sudo chmod 755 /usr/local/bin/oem-heartbeat
   ```

2. Verify that the script works as user oracle

   ```bash
   sudo su - oracle
   ORACLE_SID=YOUR_OEM_SID HC_URL="https://hc-ping.com/YOUR-UUID" /usr/local/bin/oem-heartbeat
   ```

3. Edit the oracle user’s crontab:
   ```bash
   crontab -e
   ```

4. Add a daily heartbeat job (example: run at 07:12):
  
   ```bash
   12 7 * * * ORACLE_SID=YOUR_OEM_SID HC_URL="https://hc-ping.com/YOUR-UUID" /usr/local/bin/oem-heartbeat >>/tmp/oem-heartbeat.log 2>&1
   ```

   - `ORACLE_SID=oem database sid` → replace with the SID for your EM repository database.
   - `HC_URL=...` → your unique Healthchecks.io / Cronitor / Better Stack endpoint.
   - Output is logged to /tmp/oem-heartbeat.log.

---

## Verifying oem-heartbeat

After saving your crontab, confirm it’s scheduled:

```bash
crontab -l
```

Check logs after the first run:

```bash
tail -f /tmp/oem-heartbeat.log
```

You should see entries like:

```
[2025-09-04T07:12:00Z] OMS check...
[2025-09-04T07:12:05Z] Agent check...
[2025-09-04T07:12:10Z] EMREP open check...
[2025-09-04T07:12:12Z] All good → pinging healthcheck
[2025-09-04T07:12:12Z] Ping sent
```

---

## Maintainer Notes (Release Checklist)

When updating the script, follow these steps:

1. Update checksum

   ```bash
   sha256sum oem-heartbeat > oem-heartbeat.sha256
   git add oem-heartbeat oem-heartbeat.sha256
   git commit -m "Update checksum for vX.Y.Z"
   ```

   Update version constant in the script

   - Edit the `VERSION` variable near the top of `oem-heartbeat` to the new version (for small repos we keep the version inside the script).

   - Example (zsh/mac sed):

     ```bash
     # change 1.0.4 -> 1.0.5
     sed -i '' 's/VERSION="1.0.4"/VERSION="1.0.5"/' oem-heartbeat
     git add oem-heartbeat
     git commit -m "Bump version to vX.Y.Z"
     ```

2. Tag new version

   ```bash
   git tag -a vX.Y.Z -m "Release vX.Y.Z"
   git push origin main
   git push origin vX.Y.Z
   ```

3. Update README version

   Replace the previous version numbers in the curl URLs with vX.Y.Z.

   Commit that change:

   ```bash
   git add README.md
   git commit -m "Update README to reference vX.Y.Z"
   git push origin main
   ```

4. Verify release

   ```bash
   curl -sSLO https://raw.githubusercontent.com/jmorast/oem-heartbeat/vX.Y.Z/oem-heartbeat
   curl -sSLO https://raw.githubusercontent.com/jmorast/oem-heartbeat/vX.Y.Z/oem-heartbeat.sha256
   sha256sum -c oem-heartbeat.sha256
   ```

