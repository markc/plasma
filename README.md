# plasma

KDE Plasma (KWin) add-ons for [MixOS](https://mixos.dev) users who run
Plasma beside, or instead of, the MixOS desktop.

These are low-priority extras. They depend on MixOS services installed
under `/opt/mixos`, and they get dropped if keeping them current gets
awkward. The native MixOS desktop (compd + egui) versions live in
[markc/mixos](https://github.com/markc/mixos).

| Path | What |
|---|---|
| [`mixtray/`](mixtray/) | MixTray: a Plasma 6 plasmoid with four tabs (COS, BUS, MIX, SSH), fed by `mixtrayd` |
| [`tray/`](tray/) | `tray`: a StatusNotifierItem in Rust that shows MixOS unit state in Plasma's tray |

Both moved here from `markc/mixos` (`apps/mixtray`, `apps/tray`) on
2026-10-23, with their history.

## MixTray

It needs `mixtrayd` from MixOS at `/opt/mixos/bin/mixtrayd`, running as a
user service. Install or upgrade the plasmoid with:

```
kpackagetool6 -t Plasma/Applet -i mixtray/package
kpackagetool6 -t Plasma/Applet -u mixtray/package
```

Then add "MixTray" to the system tray (System Tray Settings, Entries).

## tray

See [`tray/README.md`](tray/README.md). It is a Cargo workspace member:

```
cargo build --release -p tray
```

## Licence

MIT OR Apache-2.0, at your option. See `LICENSE-MIT` and `LICENSE-APACHE`.
