+++
title = "What is cron?"
date = "2026-03-16"
+++
It is a simple linux utility that allows certain activities to happen at
specified times without requiring user input to commence the action.[^1]

## What is the structure of a cron job?

The default semantics of a cron job is a pattern of timings followed by a
command to be executed at the specified time intervals. The pattern is shown
below:

```
  * * * * * <command to execute>
# | | | | |
# | | | | day of the week (0–6) (Sunday to Saturday;
# | | | month (1–12)             7 is also Sunday on some systems)
# | | day of the month (1–31)
# | hour (0–23)
# minute (0–59)
```

___

[^1]: It is nowadays often preferred to use systemd's timer abilities instead of a direct cron job.
