Stop the active Android QA recording session and finalize the recording.

Run:

```bash
./stop-recording
```

Report the finalized recording location and command count shown in the output.

If performance tracking was enabled, also report the performance metrics file path and key metrics (total frames, janky frames, memory usage) from the output.

If Perfetto tracing was enabled, report the trace file path and size, and remind the user to view at https://ui.perfetto.dev.
