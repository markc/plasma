# plasma: agent notes

Public repo (`markc/plasma`): KDE Plasma/KWin compatibility apps for MixOS.
It is its own project with its own versions, docs and pace, and not part of
`markc/mixos` or mixos.dev. Over time it may hold more Plasma apps besides
the first one, MixTray.

- **MixOS is an outside dependency.** These apps only use MixOS's published
  command-line and Bus interfaces, such as `mixtrayd snapshot|wait|control|ssh`.
  Never vendor MixOS code here, and never add docs about these apps to the
  MixOS tree or site. If an interface changes in MixOS, update the app here.
- **Layout:** one top-level directory per app, each with a README. Shared
  icons go in `icons/hicolor/...`. Rust apps are members of the root Cargo
  workspace.
- **Docs** live in this repo: app READMEs, plus `_doc/` and `_journal/`
  when they're needed (`YYYY-MM-DD-lower-case-title.md`). `docs/` is
  reserved for GitHub Pages.
- **Public repo.** No private values: host names, addresses, keys or
  personal home paths. Use RFC 5737 addresses, `example.*` domains and
  `/home/user`.
- **Rust builds run on the CBC workers, never locally:**
  `mix ~/.mix/_bin/cbc-build.mix --repo plasma -- cargo test`.
- **Scripts are Mix, never Python.**
- Always commit and push.
