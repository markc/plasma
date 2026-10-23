# notify-bridge: blocked on child stdout events

The bridge cannot yet be implemented with the published Mix 0.117.0
primitives. The brief requires stopping at this gate rather than introducing
polling or another implementation language. No bridge or user unit is installed.

The notifyd contract was read in full from `docs/notifyd/README.md` in an
external MixOS checkout. The planned integration needs only its published
`notify.changed` topic and `notify.list/get/action/dismiss` verbs.

## Evidence

Runtime: `mix 0.117.0 (fe9dfa9048c0, built 2026-10-23T00:11:14Z)`.
The live builtin catalogue and `mix man` documentation establish:

- `task_start(command, body)` starts an async handler in the same evaluator,
  using the serve runner's LocalSet. It is not a background process reader.
- `spawn(argv, {exit_event:true})` delivers `proc.exited`, but stdout routes
  are null, inherit or file. `stdout:"capture"` raises `OPTION_INVALID`:
  `spawn: stdout cannot be "capture": spawn is fire-and-forget (capture means waiting — use run_argv)`.
- `run_argv` captures stdout only as a completed process result. Its `stream`
  option tees output to the terminal; it does not deliver lines to handlers.
- `run_stream` inherits stdio, captures nothing, and blocks until child exit.
- `audio_watch` has a managed stdout reader specifically for `pactl subscribe`.
  There is no corresponding generic child stdout event primitive.

The runtime proof below passed `mix lint` with zero errors and warnings.
Run it with `mix --serve PROBE.mix --name notify-bridge-probe`:

```mix
on probe.child async
  print("child:begin")
  $r = run_argv(["sleep", "2"])
  print("child:end")
end

on probe.peer async
  sleep(0.1)
  print("peer:responsive")
end

task_start("probe.child", "{}")
task_start("probe.peer", "{}")
```

Observed stdout, with a four-second whole-process test deadline:

```text
child:begin
child:end
peer:responsive
```

Control: replace the `run_argv` assignment with `sleep(2)`. Observed:

```text
child:begin
peer:responsive
child:end
```

Both serve probes were killed by the test deadline after printing these lines.
Thus async handlers and `task_start` work, but synchronous process execution
still blocks the evaluator. An indefinitely running `gdbus monitor` cannot be
consumed this way while serving Bus events. Redirecting it to a file and
checking that file would be polling, which the brief explicitly forbids.

## Proposed Mix primitive

Add `process_watch(argv, opts) -> handle` and `process_unwatch(handle)` to
MixOS, with evaluator-owned child lifetime and asynchronous pipe reads.
Deliver `proc.line {watch, tag, stream, line}` and an ordered terminal event
with exit status. Bound line length and queue size, report overflow explicitly,
and kill/reap the process group on cancellation, shutdown or generation
retirement. Validate argv/options before starting a child; never invoke a shell.
This can follow the ownership and event delivery patterns of `audio_watch`
and managed `spawn`, while supporting arbitrary argv and stdout lines.

The bridge can then watch `gdbus monitor`, process its signal lines as events,
and keep notifyd-to-Plasma mappings in its synchronous handlers. A real
concurrency proof must show a line arriving before watcher exit while Bus
requests remain responsive, including shutdown and reload cleanup.

## Remaining work

Implement and publish that Mix primitive first. Then build the bridge, unit,
installation README and root app entry, and perform all requested real Plasma
toast, close, action and screenshot tests. None of those end-to-end tests has
been claimed as passing. No MixOS source was changed by this investigation.
