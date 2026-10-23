# plasma: agent notes

Public repo (`markc/plasma`). Plasma/KWin extras for MixOS users. The
native MixOS versions live in `markc/mixos`, and this repo is their
low-priority sibling: the MixTray plasmoid serves as the template for the
native compd + egui MixTray.

- **Public repo.** No mesh-private values: host names, addresses, keys or
  operator home paths. Use RFC 5737 addresses, `example.*` domains and
  `/home/user`.
- **Rust builds run on the CBC workers, never locally:**
  `mix ~/.mix/_bin/cbc-build.mix --repo plasma -- cargo test -p tray`.
- **Scripts are Mix, never Python.**
- `mixtrayd` (the data service) stays in `markc/mixos`. Its Bus verbs
  (`mixtrayd.snapshot`, `wait`, `control`, `ssh`) are the contract the
  plasmoid uses.
- Always commit and push.
