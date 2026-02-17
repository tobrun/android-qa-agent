Start a new Android QA recording session.

Derive a session name from the test scenario context (e.g., `login-flow`, `checkout-happy-path`). The name must only contain letters, numbers, hyphens, and underscores.

## Device Selection

Before starting the recording, detect connected devices:

1. Run `adb devices` and parse the output (skip the header line; each device line is `<serial>\tdevice`).
2. **0 devices** → Tell the user: "No Android devices connected. Please connect a device and try again." Do not proceed.
3. **1 device** → Auto-select it. Pass `--device <serial>` to `./start-recording`.
4. **Multiple devices** → Use **AskUserQuestion** to let the user pick which device to target. Show each serial as an option, marking the first as "(Recommended)". Pass the chosen serial via `--device <serial>`.

## Performance Tracking

Check the user's prompt for any of these keywords (case-insensitive): "track performance", "capture metrics", "measure performance", "frame rate", "jank", "memory usage", "performance metrics", "fps", "frame drops", "rendering".

If any keyword matches, add `--perf` to the `./start-recording` command below.

## Perfetto Tracing

Check the user's prompt for any of these keywords (case-insensitive): "enable tracing", "systrace", "perfetto", "system trace", "capture trace", "trace the app", "tracing".

If any keyword matches, add `--trace` to the `./start-recording` command below.

## No-Clear Detection

Check the user's prompt for any of these keywords (case-insensitive): "don't clear", "no clear", "preserve state", "keep data", "warm start", "without clearing".

If any keyword matches, add `--no-clear` to the `./start-recording` command below.

## Package Resolution

Before starting the session, resolve the app package name:

1. Extract a keyword from the user's prompt that identifies the app (e.g., "clock", "maps", "contacts", "navigation").
2. Run `adb -s <serial> shell pm list packages | grep <keyword>` to find matching packages.
3. **0 matches** → Use **AskUserQuestion** to ask the user for the full package name.
4. **1 match** → Use it (strip the `package:` prefix).
5. **Multiple matches** → Use **AskUserQuestion** to let the user pick from the matches. Show each package as an option.
6. Pass `--package <pkg>` to `./start-recording`.

## Start the Session

```bash
./start-recording <session-name> --prompt "<original user prompt>" --device <serial> --package <pkg>
```

Add `--perf` if performance tracking keywords were detected above.
Add `--trace` if Perfetto tracing keywords were detected above.
Add `--no-clear` if no-clear keywords were detected above.

Always pass the user's original test scenario prompt via `--prompt` so it is retained in the recording metadata.

If a session is already active, it will be auto-stopped and finalized before the new one begins.

The `--package` flag causes `start-recording` to automatically force-stop and clear the app's data before the test begins (unless `--no-clear` is also passed).

## After Launching the App

If `--perf` was used, after the first `am start -n <pkg>/<activity>` command, reset the gfxinfo counters so the test captures fresh frame data:

```bash
./android-qa shell dumpsys gfxinfo <pkg> reset
```

Replace `<pkg>` with the package name from the `am start` component (the part before `/`). This command goes through `./android-qa` so it gets recorded and replays correctly.

If `--trace` was used, after launching the app (and after the gfxinfo reset if `--perf` was also used), start the Perfetto trace in the background:

```bash
./android-qa shell perfetto -o /data/misc/perfetto-traces/trace.perfetto-trace --no-guardrails -d gfx view input am wm dalvik sched freq
```

The `-d` flag detaches immediately and `--no-guardrails` disables duration and size limits so the trace runs for the entire test session. The command is recorded in the session. The trace runs until `./stop-recording` stops it cleanly via SIGTERM.
