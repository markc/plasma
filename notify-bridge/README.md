# notify-bridge

Shows MixOS notifyd notifications as native Plasma toasts. Actions and user
dismissals settle the original notification; outcomes elsewhere close the
Plasma copy. Expiry remains notifyd's responsibility. Open notifications are
reconciled after subscribing at startup.

Requires a graphical session, `gdbus`, `notify-send` (libnotify), local notifyd and Mix with
`process_watch` (MixOS commit `97b82fdb`, pending gold). No build is needed.
The icon is `dev.mixos.tray`, the existing MixOS tray artwork in
`icons/hicolor/scalable/apps/dev.mixos.tray.svg`. Install that SVG into the
matching user or system hicolor icon directory if it is not already there.
`dev.mixos` is not the installed icon name.

## Install

Copy `notify-bridge.mix` to `~/.local/lib/mixos/notify-bridge.mix` and the
unit to `~/.config/systemd/user/notify-bridge.service`, creating those
directories first. Then run:

```text
systemctl --user daemon-reload
systemctl --user enable --now notify-bridge.service
```

For a foreground run, use `mix --serve notify-bridge.mix`. Do not run two
instances. Stop the foreground instance with `send "notify-bridge" QUIT`.
Logs go to the terminal or the user journal. The user unit follows
`graphical-session.target` and restarts on failure.

## Interfaces and behaviour

Uses only the published [notifyd contract](https://github.com/markc/mixos/blob/main/docs/notifyd/README.md):
`notify.changed`, `notify.list`, `notify.action` and `notify.dismiss`.
The local Bus service is `notify-bridge`; `notify-bridge.status` reports the
two id maps and watcher handle for diagnosis.

Uses `org.freedesktop.Notifications.Notify` through `gdbus` for plain toasts
and a generation-owned `notify-send --wait --print-id` child for questions.
Plasma removes action buttons when the sending D-Bus connection disappears:
a one-shot `gdbus call` sends a valid alternating string array but exits too
soon. `notify-send` keeps that connection alive until the toast closes.
Its printed id establishes the mapping; a pending outcome is checked again
when the id arrives. No desktop entry or synthetic `default` action is needed.
Questions set `resident=true` and `expire_timeout=0`; notifyd's published
record currently has no urgency or level field. All Plasma copies use timeout
0, with expiry controlled by notifyd (an explicit notifyd deadline can still
settle a question). `CloseNotification` uses direct `gdbus` argv.
A managed `gdbus monitor` supplies signals without
polling. Only mapped ids are handled. Close reasons 1 and 3 send no outcome.
A dead monitor is logged and restarted after 0.5 seconds. Queue overflow and
D-Bus call failures are logged. Failed Notify calls are retried on the next
reconciliation (service restart); there is no polling retry loop.

Watchers start at initial boot and in `lifecycle.commit` after reload; reload
candidates remain passive. Mappings are in memory: restarting or reloading
while toasts remain visible can show duplicate copies. Prefer stopping after
settling open notifications. A monitor outage can lose signals emitted while
it is down; notifyd remains the authority.
