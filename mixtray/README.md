# MixTray (Plasma)

A Plasma 6 system tray plasmoid that shows MixOS state in four tabs:

| Tab | Shows |
|---|---|
| COS | MixOS systemd units and their state; start, stop and restart for the units that allow it |
| BUS | services on the MixOS Bus, and the mesh peers it knows |
| MIX | the installed `mix` binary |
| SSH | hosts in `~/.ssh/hosts/`: connect, edit, test, add, trash, restore and purge |

## What it needs

It needs `mixtrayd`, the MixOS tray data service, installed at
`/opt/mixos/bin/mixtrayd` and running as a user service. The plasmoid runs
`mixtrayd` as a one-shot client and reads JSON from its stdout:

| Command | Used for |
|---|---|
| `mixtrayd snapshot` | the four tabs, on open and on Refresh |
| `mixtrayd wait REVISION` | the next snapshot after a change, kept running while the popup is open, so nothing polls |
| `mixtrayd control UNIT start\|stop\|restart` | the COS row buttons |
| `mixtrayd ssh probe\|create\|trash\|restore\|purge ...` | the SSH tab actions |

Those commands are the plasmoid's only contract with MixOS. `mixtrayd` is
built and documented in MixOS. If it is missing or not running, the popup
says so and does not retry in a loop.

Connect runs `konsole -e ssh ALIAS`, and Edit runs `xdg-open` on the host
file. Both run in the Plasma session.

## Install

From the repository root:

```
kpackagetool6 -t Plasma/Applet -i mixtray/package     # first time
kpackagetool6 -t Plasma/Applet -u mixtray/package     # upgrade
install -Dm644 icons/hicolor/scalable/apps/dev.mixos.tray.svg \
  ~/.local/share/icons/hicolor/scalable/apps/dev.mixos.tray.svg
```

Then turn on "MixTray" under System Tray Settings, Entries.

Uninstall with `kpackagetool6 -t Plasma/Applet -r dev.mixos.mixtray`.
