# Plasma action buttons and icon

D-Bus monitoring confirmed the bridge sent a valid alternating string array.
The missing buttons came from the one-shot sender disconnecting: Plasma
removed its actions. A managed `notify-send --wait --print-id` child keeps
the libnotify connection alive for each question. The existing monitor still
forwards the real Plasma action signal to notifyd. Questions use resident
true and timeout zero; no desktop entry or default action is needed.

Use the existing `dev.mixos.tray` themed icon. `dev.mixos` was absent.
Screenshots showed Yes/No and OK alongside the MixOS icon. Real xdotool
clicks settled notifyd records with actions `yes` and `ok`. Plain toasts
and question withdrawal also passed. Mix lint and unit verification passed.
All test children were stopped; the unit was not installed.
