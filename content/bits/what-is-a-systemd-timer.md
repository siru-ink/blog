+++
title = "What is a systemd timer?"
date = "2026-03-17"
+++
As an update to cron's abilities modern linux systems can utilize systemd timers
instead. This allows the running of a cron action to be journaled via
`journalctl` making it much easier to debug if something goes wrong in job.

A systemd timer consists of two files that are activated using a few commands.
The two files are a basic systemd service file, such as `test-service.service`
with contents similar to this:

```
[Unit]
Description=Test Service
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart="do some stuff or run a script"

[Install]
```

As well as a simple timer file, such as `test-service.timer`, with contents
similar to:

```
[Unit]
Description=Test Service Timer

[Timer]
OnCalendar=*:0/5
Persistent=true
AccuracySec=1min
RandomizedDelaySec=1sec
Unit=backup.service

[Install]
WantedBy=timers.target
```

For user services, these files can reside in `~/.config/systemd/user/`, whereas
for system/root services, these files should be located in
`/etc/systemd/system/`. To activate the timer, the following command can be
used:

```bash
systemctl enable --now test-service.timer  # or systemctl --user enable --now test-service.timer
```

And to list all running timers, the following command can be used:

```bash
systemctl list-timers --all  # or systemctl --user list-timers --all
```

Lastly, a more in depth guide on systemd timers can be found
[here](https://blogs.reliablepenguin.com/2025/10/15/systemd-timers-a-practical-guide-to-replacing-cron-on-linux).
