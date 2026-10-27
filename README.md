# Legenda Studio — English edition

Caption plugin for **Adobe Premiere Pro 2026 on Windows**, packaged as a signed
CEP/ZXP extension with a local transcription and rendering engine.

**Development status:** package integrity and automated checks are verified.
This CEP edition still needs functional testing inside Premiere. Its certificate
is self-signed for development; it is not Adobe Marketplace approval.

## Features

- Local Whisper large-v3 transcription on an NVIDIA GPU.
- Multiple spoken languages and automatic language detection.
- Optional local translation with M2M100, with editable translated captions.
- One to twelve words per caption, with regrouping after text edits.
- Fonts, text and outline color palettes, dark backgrounds, and bloom/glow.
- Entrance and exit presets, including adjustable Pop Out intensity.
- Decimal caption durations such as `0.5` and `1.5` seconds.
- Per-caption timing edits, frame nudges, and a global synchronization offset.
- Automatic panel preview and timeline updates after local rendering completes.
- Imported caption media grouped in a dedicated project bin.

This English edition has a separate extension ID, settings folder, and caption
bin. Installing it does not replace the Portuguese UXP or CEP editions. Use one
panel at a time when editing the same sequence.

## Install the panel

1. Download **Legenda-Studio-0.4.0-CEP-EN.zxp** from this repository.
2. Install it with a CEP/ZXP installer that supports self-signed packages.
3. Restart Premiere. Open **Window > Extensions > Legenda Studio CEP (English)**.
   Your Premiere installation may call this menu **Extensions (Legacy)**.

No third-party installer is bundled. The package does not change Adobe debug
settings. Opening the panel does not issue timeline-editing commands.

## Prepare the local engine

The ZXP contains `motor-local.zip`. In the panel, click **Save local engine**,
then extract the ZIP to a folder where you can write files. A ZIP reader can also
extract it directly from the ZXP.

Requirements: 64-bit Python 3.10+, an NVIDIA GPU compatible with CUDA 12,
a compatible driver, and enough available GPU memory for Whisper large-v3.
The engine checks GPU availability. Models and dependencies are downloaded
only when you explicitly run the setup commands.

Open PowerShell in the extracted engine folder:

```powershell
# powershell · setup.ps1 · local transcription engine · Windows · PowerShell 5.1+
.\setup.ps1 -Install -InstallGpu -DownloadModel
.\start-engine.ps1
```

If Windows blocks local scripts, run them with a process-scoped override:

```powershell
# powershell · engine setup · Windows · PowerShell 5.1+
powershell -NoProfile -ExecutionPolicy Bypass -File .\setup.ps1 -Install -InstallGpu -DownloadModel
powershell -NoProfile -ExecutionPolicy Bypass -File .\start-engine.ps1
```

For optional local translation:

```powershell
# powershell · setup.ps1 · optional M2M100 translation · Windows · PowerShell 5.1+
.\setup.ps1 -InstallTranslation -DownloadTranslationModel
```

Keep the engine running. In the panel, click **Connect** and choose
`runtime/connection.json` from the engine folder. That file contains a local
connection secret: do not upload it. Models, media, cache, and connection keys
are not included in this repository's ZXP.

An existing compatible Legenda Studio engine can also be used by selecting its
`connection.json`. Do not run two engines on port 47831 simultaneously. For
English engine messages, use this edition's bundled engine. The panel also
localizes known messages from the existing Portuguese engine.

## Create and style captions

1. Select a direct media clip with speech on the timeline. Linked audio and
   video are supported. Nested sequences and speed changes are not supported
   by this CEP edition; export audio for those cases.
2. Choose the spoken language or automatic detection, then click
   **Transcribe selection**.
3. Review the text and timing. Enable optional translation if needed.
4. On **Style**, select the word limit, font, colors, glow, and animations.
5. Click **Apply to timeline**. With automatic updates enabled, subsequent valid
   style and duration changes refresh the preview and then update the track
   after rendering finishes.

By default, a caption disappears when its speech ends. **Custom duration** can
extend its display time, capped at the next caption. Minimum duration accepts
decimal points and commas. **Review > Timing** controls the global offset;
individual Start/End fields adjust a single caption.

## Timeline behavior and limitations

Timeline output consists of transparent MOV graphics generated locally and
reused from a render cache. These are not native editable Premiere text clips,
and timeline updates wait for rendering. Imported graphics go into the
**Legenda Studio CEP EN** bin.

Updates are prepared on a muted reserve track. The visible version switches
only after clip count and timing checks pass. The previous track becomes the
reserve. Do not enable or edit a reserve track during an update. If the plugin
detects manual changes to an owned track, it stops to preserve those changes.

Automatic track creation uses Premiere's undocumented QE API. If it is not
available in your version, create an empty video track above the others and
retry. Host commands are serialized: if Premiere has not responded, the next
command waits for that response.

## Integrity

The `.zxp.sha256` file contains the package's SHA-256 checksum. The ZXP signature
is verified with Adobe's official ZXPSignCmd 4.1.3 using `-verify -certInfo
-skipOnlineRevocationChecks`. A self-signed certificate verifies package
integrity without providing a public certificate trust chain.

The signing certificate, signing password, runtime state, models, generated
media, and private diagnostics are excluded from the publication files.
