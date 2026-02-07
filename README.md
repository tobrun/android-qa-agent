# Android QA Agent

Record-and-replay QA tool for Android. Claude Code drives an Android emulator via natural language, and all ADB commands are transparently recorded for later replay.

## Prerequisites

- Python 3.8+
- `adb` on your `PATH` (or set `ANDROID_HOME`)
- A running Android emulator or connected device
- [Claude Code](https://claude.com/claude-code)

## Record

Open Claude Code in this directory. The `CLAUDE.md` system prompt configures Claude as a QA engineer to use `android-qa` for all device interaction.
Prompt a test scenario in natural language:

```
> open the settings app and toggle dark mode
```

Claude will:

1. Start a recording session
2. Take UI dumps to find accurate tap coordinates
3. Execute the steps via `android-qa`
4. Stop the recording when done

## Replay

```bash
./android-qa-replay .android-qa/recordings/my-session.json --speed 5
```

Replay preserves relative timing between commands, compressed by the speed factor. It aborts on the first command failure.

## Recording Format

Finalized recordings are stored at `.android-qa/recordings/<session>.json`:

```json
{
  "metadata": {
    "name": "login-flow",
    "started_at": "2025-01-15T10:32:00.000Z",
    "completed_at": "2025-01-15T10:33:45.000Z",
    "command_count": 27
  },
  "commands": [
    {
      "issued_at": "2025-01-15T10:32:01.123Z",
      "args": ["shell", "input", "tap", "540", "960"],
      "exit_code": 0,
      "duration_ms": 245
    }
  ]
}
```

## Project Structure

```
android-qa-agent/
├── android-qa              # ADB wrapper (Python, executable)
├── android-qa-replay       # Replay tool (Python, executable)
├── start-recording         # Session start script
├── stop-recording          # Session stop script
├── CLAUDE.md               # Claude Code system prompt
├── .claude/
│   ├── settings.json       # Stop hook for auto-finalizing sessions
│   └── skills/             # Claude Code skills
└── .android-qa/            # Created at runtime
    ├── active-session.json # Lock file (only during recording)
    └── recordings/         # Finalized recordings
```

## Limitations

- Streaming ADB commands (`logcat`, `shell top`, etc.) are not supported. 
- Single-threaded: one `android-qa` invocation at a time.
- No stdout/stderr capture — only the command and its metadata are recorded.
