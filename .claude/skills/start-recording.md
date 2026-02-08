Start a new Android QA recording session.

Derive a session name from the test scenario context (e.g., `login-flow`, `checkout-happy-path`). The name must only contain letters, numbers, hyphens, and underscores.

Run:

```bash
./start-recording <session-name> --prompt "<original user prompt>"
```

Always pass the user's original test scenario prompt via `--prompt` so it is retained in the recording metadata.

If a session is already active, it will be auto-stopped and finalized before the new one begins.
