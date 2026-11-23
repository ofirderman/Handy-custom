# Custom dictation profiles

Goal: accurate Hebrew and English dictation and English voice-to-code using local models, without changing the installed stable app.

- [x] First milestone (implementation only): opt-in per-shortcut ASR model/language overrides; preserve default behavior, never persist overrides, reject wrong-model results, use batch for explicit overrides.
- [ ] Verify Rust build and run unit tests; Windows microphone and paste smoke test with both shortcuts.
- [ ] Add persisted per-shortcut profile UI and independent prompt selection.
- [ ] Add normal-mode bilingual model routing using existing language detector; compare mixed-language samples and avoid forcing an incorrect route.
- [ ] Measure load latency, VRAM, code corrections; improve unloading and formatting only where needed.

Use a separate Tauri identifier for local trials, disable updater, and keep original installed app and its settings intact. Never commit recordings or credentials.

## Windows first trial (not yet built or runtime tested)

Follow BUILD.md for Rust, Bun, Microsoft C++ build tools, CMake and Vulkan SDK. Quit the official Handy from its tray first to avoid conflicting shortcuts; do not uninstall it. From the root of this branch:

```powershell
bun install
New-Item -ItemType Directory -Force src-tauri/resources/models | Out-Null
curl.exe -fL "https://huggingface.co/handy-computer/silero-vad-v6.2-gguf/resolve/main/silero-vad-v6.2-F32.gguf" -o src-tauri/resources/models/silero-vad-v6.2-F32.gguf
if ($LASTEXITCODE -ne 0) { throw "VAD download failed" }
$src = Join-Path $env:APPDATA "com.pais.handy\models"
$dst = Join-Path $env:APPDATA "com.pais.handy.custom\models"
New-Item -ItemType Directory -Force $dst | Out-Null
foreach ($name in @("ivrit-large-v3-turbo.bin", "ggml-large-v3-q5_0.bin")) {
    $file = Join-Path $src $name
    if ((Test-Path $file) -and !(Test-Path (Join-Path $dst $name))) { Copy-Item $file $dst }
}
$env:HANDY_CODE_ASR_MODEL = "large"
$env:HANDY_CODE_ASR_LANGUAGE = "en"
$env:HANDY_DISABLE_UPDATER = "1"
bun run tauri dev --config src-tauri/tauri.custom.conf.json
```

Use the Custom app's model UI to download Whisper Large if the file was not present, and select the ivrit.ai model for normal dictation. Configure normal spoken languages and choose Hebrew or Auto for normal dictation as appropriate. Configure the existing post-processing shortcut and local Ollama endpoint, model, and code prompt in the Custom app. Code dictation uses Whisper Large (ID `large`) and English regardless of the normal language preference. Model overrides are opt-in and process-local; remove both HANDY_CODE_ASR_* variables to use stock behavior. Only one ASR engine is loaded at a time, so switching models has a load cost. This first milestone does NOT implement automatic model routing for normal dictation or a profile settings UI.

Smoke tests:
- Normal Hebrew, normal English, mixed language normal speech; report raw transcript for each.
- Code shortcut: Java, C#, HTML and a final verbal correction. Compare raw and processed history.
- Switch between the shortcuts repeatedly; test a missing code model (must not decode with the normal model) and cancel a recording.
- Quit the custom app and reopen official Handy; verify original settings are unchanged.

Development checks: `cargo fmt --manifest-path src-tauri/Cargo.toml -- --check`, `cargo test --manifest-path src-tauri/Cargo.toml blank_profile_overrides_are_ignored`, plus the upstream test suite. No tests have run in ChatGPT's environment.
