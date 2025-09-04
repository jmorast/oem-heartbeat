# oem-heartbeat
Dead man’s switch heartbeat for Oracle Enterprise Manager (OEM), designed for cron + external services like Healthchecks.io.

## Cron Setup

1. Copy the script to `/usr/local/bin` and make it executable:

   ```bash
   sudo cp oem-heartbeat /usr/local/bin/oem-heartbeat
   sudo chmod 755 /usr/local/bin/oem-heartbeat
   ```

2. Edit the oracle user’s crontab:
   ```bash
   crontab -e
   ```

3. Add a daily heartbeat job (example: run at 07:12):
  
   ```bash
   12 7 * * * ORACLE_SID=emrep HC_URL="https://hc-ping.com/YOUR-UUID" /usr/local/bin/oem-heartbeat >>/tmp/oem-heartbeat.log 2>&1
   ```

   - `ORACLE_SID=oemdb` → replace with the SID for your EM repository database.
   - `HC_URL=...` → your unique Healthchecks.io / Cronitor / Better Stack endpoint.
   - Output is logged to /tmp/oem-heartbeat.log.

---

## Verifying

After saving your crontab, confirm it’s scheduled:

```bash
crontab -l
```

Check logs after the first run:

```bash
tail -f /var/log/oem-healthcheck/oem-heartbeat.log
```

You should see entries like:

```
[2025-09-04T07:12:00Z] OMS check...
[2025-09-04T07:12:05Z] Agent check...
[2025-09-04T07:12:10Z] EMREP open check...
[2025-09-04T07:12:12Z] All good → pinging healthcheck
[2025-09-04T07:12:12Z] Ping sent
