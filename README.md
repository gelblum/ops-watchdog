# ops-watchdog

Watches the watcher. [ops-watch](https://github.com/gelblum/ops-factory) is a Cloudflare Worker
that checks Morris's apps every minute and alerts when one goes down. If ops-watch itself stops,
nothing alerts. This repo covers that, from GitHub Actions, off Cloudflare.

Every 5 minutes, `.github/workflows/watchdog.yml`:

1. Reads ops-watch's `GET /heartbeat`, which answers only `{"last_run": ISO time, "age_s": seconds}`
   (it is written every 5th minute, so 0-5 minutes old when healthy). 3 tries, 10 s apart.
2. Calls it **stale** if it is unreachable or more than 10 minutes old.
3. Texts through Twilio: once when it goes stale, a reminder at most once an hour while it stays
   stale, and once when it is back. The state between runs (timestamps only) lives in the Actions
   cache.
4. Every Monday 9 AM ET (14:00 UTC, so 10 AM during daylight time), texts "Weekly check: ...
   No action needed." to prove the whole path still works. Skipped if the heartbeat is stale, since
   the stale alert already texted.
5. Once a day, re-enables itself through the API so GitHub's 60-day inactivity rule never turns the
   schedule off.

If a text can't be sent, the run fails, so GitHub's own failed-run email is the backup.

Run it by hand from Actions > watchdog > Run workflow; tick "Send one test text" to text a test.

This repo is public so its Actions minutes are free. It holds no secrets: the Twilio account
and phone numbers are Actions secrets (`TWILIO_SID`, `TWILIO_TOKEN`, `TWILIO_FROM`,
`ALERT_PHONE`) and the heartbeat address is the variable `HEARTBEAT_URL`.
