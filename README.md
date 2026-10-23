# plasma

KDE Plasma (KWin) compatibility apps for MixOS: small tools that bring MixOS
state and controls to people who run Plasma instead of, or beside, the MixOS
desktop.

This is a separate project from [MixOS](https://github.com/markc/mixos),
with its own releases and pace. It uses MixOS services from outside, through
their command-line and Bus interfaces, and is not part of the MixOS tree or
its site. The native MixOS desktop has its own versions of these tools.

## Apps

| Path | What |
|---|---|
| [`mixtray/`](mixtray/) | MixTray: a Plasma 6 tray plasmoid with four tabs (COS, BUS, MIX, SSH), fed by MixOS's `mixtrayd` |
| [`tray/`](tray/) | `tray`: a StatusNotifierItem in Rust that shows MixOS unit state in Plasma's tray |
| [`icons/`](icons/) | Shared icons, in the hicolor layout (`dev.mixos.tray`) |

More Plasma apps may be added over time. Each one gets its own top-level
directory and README.

## Build

The Rust apps are members of one Cargo workspace:

```
cargo build --release
cargo test
```

The QML plasmoids need no build step. Each one's README covers installing
it.

## History

`mixtray` and `tray` started in `markc/mixos` as `apps/mixtray` and
`apps/tray`. They moved here on 2026-10-23, with their history.

## Licence

MIT OR Apache-2.0, at your option. See `LICENSE-MIT` and `LICENSE-APACHE`.
