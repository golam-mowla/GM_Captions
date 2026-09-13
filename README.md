# GM Captions (v1 scaffold)

CEP panel for Premiere Pro: generate subtitles/captions with local Whisper
(offline, whisper.cpp) or your existing manual Colab/Kaggle Whisper workflow,
then chunk them using the same preferences as Premiere's native "Create
Captions" dialog (max chars, min duration, gap, single/double line).

## What's built vs what needs your testing

**Solid / standard:**
- Manifest, panel UI, slider/box sync, mode toggle, SRT parsing + chunking logic
- Node child_process spawn wiring for whisper-cli.exe

**Needs validation on your machine (flag back what happens, same as GM Fx's
sequence-detection bug did):**
- `hostExportAudioRange` — uses `exportAsMediaDirect`, requires a real 16kHz
  mono WAV `.epr` preset. You'll need to create one once in Premiere's export
  dialog (Format: Waveform Audio, 16000 Hz, Mono) and save it as
  `host_bin/audio_16k_mono.epr`.
- `hostImportCaptions` — attaching the SRT as a **live, native Captions
  track** (not just a project-panel file) is the riskiest ExtendScript call
  here. Premiere's own UI does this via its internal captions engine, which
  isn't a stable public API. First test will tell us whether
  `importFiles()` auto-attaches or whether we need a manual drag/attach step
  scripted in.

## Setup steps

1. **Get the whisper.cpp binary + models**: push this repo to GitHub, rename
   `.github_workflow_whisper-build.yml` to `.github/workflows/build-whisper.yml`
   via the GitHub web UI (New File button — drag-drop skips `.github/`),
   run the workflow manually (Actions tab → Run workflow), download the
   `gm-captions-host-bin` artifact, and drop `whisper-cli.exe` + `models/`
   into this project's `host_bin/` folder.

2. **Create the WAV export preset** in Premiere (Export → Audio only →
   Waveform, 16kHz, Mono) → save preset as `audio_16k_mono.epr` into
   `host_bin/`.

3. **Load the panel**: same as your other CEP panels —
   `PlayerDebugMode = 1` under `CSXS.9` (or whichever CSXS version matches
   your Premiere build) in the registry, then copy this whole folder to
   `C:\Program Files (x86)\Common Files\Adobe\CEP\extensions\GM_Captions\`.

4. **Cloud mode**: paste your Colab/Kaggle notebook URL into the panel once —
   it's remembered per-session. Flow: Export audio → Open notebook → run it
   like today → download the `.srt` it produces → pick it back in the panel.

## Known constraint (be realistic about this)

`large-v3` quantized on your 8GB CPU-only laptop will still be slow — expect
roughly 2-5x the clip's own length in transcription time. If a clip is taking
too long, drop to `base.en` or `small.en` in the model picker, or switch to
Cloud mode for that one.
