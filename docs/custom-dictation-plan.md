# Custom dictation profiles

Goal: accurate Hebrew and English dictation and English voice-to-code using local models, without changing the installed stable app.

- [ ] First milestone: opt-in per-shortcut ASR model/language overrides; preserve default behavior, never persist overrides, reject wrong-model results, use batch for explicit overrides.
- [ ] Verify Rust build and run unit tests; Windows microphone and paste smoke test with both shortcuts.
- [ ] Add persisted per-shortcut profile UI and independent prompt selection.
- [ ] Add normal-mode bilingual model routing using existing language detector; compare mixed-language samples and avoid forcing an incorrect route.
- [ ] Measure load latency, VRAM, code corrections; improve unloading and formatting only where needed.

Use a separate Tauri identifier for local trials, disable updater, and keep original installed app and its settings intact. Never commit recordings or credentials.
