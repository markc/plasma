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

Copy `notify-bridge.mix` and `signals.mix` to `~/.local/lib/mixos/` and the
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
`notify.changed`, `notify.list`, `notify.get`, `notify.present`, `notify.action`
and `notify.dismiss`.
The local Bus service is `notify-bridge`; `notify-bridge.status` reports the
two id maps, child watches and both monitor handles for diagnosis.

Uses `org.freedesktop.Notifications.Notify` through `gdbus` for plain toasts
and a generation-owned `notify-send --wait --print-id` child for questions.
Plasma removes action buttons when the sending D-Bus connection disappears:
a one-shot `gdbus call` sends a valid alternating string array but exits too
soon. `notify-send` keeps that connection alive until the toast closes.
Each watch is mapped to its notifyd id immediately, independently of its
printed Plasma id. The child's trailing stdout action settles that record
even if a click precedes receipt of the Plasma id. A pending outcome is
checked again when the id arrives. A child exit always removes its tracking;
without an action or a reason-2 close signal, no human choice is inferred.
At most 20 question children run concurrently. Further questions remain
recorded and appear as plain toasts, with a logged warning. They can still
be answered through notifyd's other surfaces.
For regression tests, `NOTIFY_BRIDGE_MAX_CHILDREN` lowers the cap: set it to
`3` and send five questions to get three action children and two plain toasts.
Unset or empty, it defaults to `20` and accepts only integers from `0` to `20`; invalid values
refuse startup. Leave it unset in normal operation.
No desktop entry or synthetic `default` action is needed.
Urgency maps `low`, `normal` and `critical` to the D-Bus byte hint 0, 1 and 2.
Critical toasts and questions with action children set `resident=true` and `expire_timeout=0`.
All Plasma copies use timeout 0, with expiry controlled by notifyd (an explicit
notifyd deadline can still settle a question). The sender's non-empty `app`,
then `source`, then `MixOS` supplies `app_name`. The origin hint stays `MixOS`:
Plasma renders these as, for example, `Ced · MixOS`.
`CloseNotification` uses direct `gdbus` argv.
A managed `gdbus monitor` supplies signals without
polling. Only mapped ids are handled. Every close reason removes the id
mapping; only reason 2 sends `notify.dismiss`. GVariant action strings support
single and double quotes and escaped characters.
A second managed monitor watches the notification server's D-Bus owner.
An owner change drops the dead mappings and retires question children;
an acquired owner gets fresh presentations without closing old references.
Dead monitors restart with independent 0.5-second exponential backoff capped
at 30 seconds, reset after a healthy minute. Queue overflow and
D-Bus call failures are logged. Failed Notify calls are retried on the next
reconciliation (service restart); there is no polling retry loop.

Watchers start at initial boot and in `lifecycle.commit` after reload; reload
candidates remain passive. Each shown Plasma id is saved through
`notify.present {id, surface:"plasma", ref}`. A failed save closes the new
toast. At startup and committed reload, `notify.list` supplies durable refs.
The notification interface has no query for an individual toast, so the bridge
closes all saved refs before presenting any replacements, so reused ids are
safe. Failed closes are logged and do not stop reconciliation. This also restores
action buttons after the old sender connection retires. Refs disappear when
notifyd settles the record. Only one bridge instance may run at a time.
A monitor outage can lose signals emitted while it is down; notifyd remains
the authority.

Before a child prints its id, a runtime-directory pending-watch journal
prevents a reload from retiring an unreferenced toast and presenting it again.
Such a reload reverts with `PRESENT_PENDING`; retry once the id arrives.
Initial startup replaces stale pending state. Run only one instance.

## Regression checks

Run `mix notify-bridge/signals_test.mix` for GVariant parsing and backoff
boundaries. For safe owner-change testing, set `NOTIFY_BRIDGE_OWNER_FIXTURE`
to the absolute path of `owner_fixture.mix`, and `NOTIFY_BRIDGE_OWNER_INPUT`
to a test input file. Write a `NameOwnerChanged`-shaped line to that file;
the generation-owned fixture emits it through the real `process_watch` path.
Leave these variables unset in normal operation. Never restart Plasma to
exercise this case.
