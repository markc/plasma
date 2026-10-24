# launchers

Plasma menu entries for the MixOS apps installed in `/opt/mixos`, so you can
start Email, Prefs, Term and the others from KDE's launcher, KRunner or a
panel.

```
mix launchers/launchers.mix install   # write a launcher and icon for every MixOS app
mix launchers/launchers.mix check     # ok / missing / stale per app; exit 1 if any are off
mix launchers/launchers.mix remove    # delete everything this script wrote
```

Run `install` again after a MixOS update. It adds launchers for new apps and
removes the ones for apps MixOS no longer ships.

## How the launchers work

- **Source:** MixOS's own entries in `/opt/mixos/share/applications`
  (`MIXOS_PREFIX` overrides `/opt/mixos`). Nothing is copied into this
  repo, so the names, comments, categories, MIME types and actions
  (Screenshot's *region* and *screen*) stay MixOS's.
- **Output:** `~/.local/share/applications/dev.mixos.<app>-plasma.desktop`
  and `~/.local/share/icons/hicolor/scalable/apps/dev.mixos.<app>-plasma.svg`.
  `XDG_DATA_HOME` is honoured.
- **Exec paths:** Plasma's `PATH` doesn't include `/opt/mixos/bin`, so
  `Exec` and `TryExec` become absolute.
- **Plasma only:** each entry has its own desktop id and `OnlyShowIn=KDE;`.
  A MixOS session that shares the same home never lists these entries,
  and they never shadow MixOS's own entry for the same app.
- **Icons:** the MixOS icons are one-colour line art. Each copy gets KDE's
  colour-scheme stylesheet, so it follows Breeze light or dark.
- **Task manager:** `StartupWMClass` is kept, so a running app's window
  shows under its launcher.
- **What `remove` deletes:** only files carrying the
  `X-MixOS-Plasma-Launcher=true` marker.
