# XTTS Audiobook Generator (Voice-Cloned TTS)

Generate audiobook from **Text** files (`.txt` only) input, match with a **custom voice** (`.mp3/.wav`), using **Multilingual Zero-Shot Text-to-Speech Model** (ZS-TTS) with **XTTS v2** (via Coqui TTS).

> ⚠️ **Ethics & Consent**: Only use voice cloning with voices you **own** or have **explicit permission** to use. Do not impersonate real people without consent.

---

## Features

### Core
- ✅ TXT → audiobook audio (`.mp3` or `.wav`)
- ✅ Voice cloning / speaker conditioning from **reference MP3**
- ✅ Stable long-form generation via **chunking**

### Audio post-processing (FFmpeg)
- ✅ **Speed control** via `atempo` (tempo change without pitch shift; supports >2× or <0.5× by chaining)
- ✅ **Bitrate control** for MP3 export (e.g. `96k`, `128k`, `192k`, `256k`)
- ✅ **Volume normalization** via `loudnorm` (targets consistent loudness)
- ✅ **Silence trimming** via `silenceremove` (leading/trailing silence removal)

---

## Model overview (XTTS)

XTTS is a **massively multilingual, zero-shot, multi-speaker TTS** system designed to:
- Support **multiple languages** (including low/medium-resource languages)
- Perform **zero-shot speaker adaptation** using short voice prompts
- Enable **cross-lingual** speech synthesis (speaker voice retained while changing target language)

The XTTS architecture builds on a language-modeling approach (inspired by Tortoise-style pipelines) with components including:
- A **VQ-VAE** audio tokenizer/encoder
- A large **Transformer** (GPT-like) model to predict audio codes conditioned on text + speaker prompt
- A neural vocoder / decoder (HiFi-GAN style) to produce waveform audio

![Figure 1:XTTS training architecture overview.](https://arxiv.org/html/2406.04904v1/extracted/5646863/Images/XTTS.png)

Figure 1:XTTS training architecture overview.
---

## Requirements

- Python 3.9+ recommended
- **FFmpeg** installed and available on `PATH` (required for MP3 I/O + filters)
- Enough RAM/VRAM for XTTS inference (faster with NVDIA GPUs and some AMD GPUs)

---

## Installation

### 1) Create an environment (recommended)

**venv**
```bash

python -m venv .venv

# Windows:

.venv\Scripts\activate

# macOS/Linux:

source .venv/bin/activate

```

### 2) Install FFmpeg

``` bash
# Windows (Chocolatey):

choco install ffmpeg

# macOS (Homebrew):

brew install ffmpeg

# Ubuntu/Debian:

sudo apt-get update && sudo apt-get install -y ffmpeg

# Verify:

ffmpeg --version
```

### 3) Install Python deps

⚠️ Dependency pin: The Coqui TTS repo is unmaintain for 2 years (as of 2026) and no longer compatible with the latest `pytorch` and `transformers`. explicitly notes they maintain a fork available via pip install coqui-tts for better compatibility with newer stacks. Coqui TTS / XTTS may break with transformers>=4.57 due to a removed top-level export (BeamSearchScorer).
This repo pins Transformers to a compatible version.

`pip install --upgrade "TTS" "transformers<5.0.0" "pydub"`
## Quick start

### 1) Prepare files (same folder as the notebook)

Put in the notebook directory:

- `voice_sample.mp3` (reference voice)
- `book.txt` (input text)

### 2) Run the notebook

```bash
jupyter notebook voice_extract.ipynb
```

### 3) Configure inputs/outputs in the notebook

In the “Set inputs/outputs” cell, adjust:

- `VOICE_MP3` (e.g., `voice_sample.mp3`)
- `TEXT_TXT` (e.g., `book.txt`)
- `OUT_AUDIO` (e.g., `audiobook.mp3` or `audiobook.wav`)
- `LANG` (e.g. `en`, `vi`, `zh-cn`)
- Post-processing controls:
  - `SPEED`
  - `MP3_BITRATE`
  - `NORMALIZE_LOUDNESS`, `LOUDNORM_I`, `LOUDNORM_TP`, `LOUDNORM_LRA`
  - `TRIM_SILENCE`, `SILENCE_THRESHOLD_DB`, `SILENCE_MIN_SEC`

Then run all cells.

---

## Recommended input quality

### Reference voice MP3
For best speaker similarity:

- 10–30 seconds of **clean** speech
- Single speaker only
- Minimal background noise, music, and echo/reverb
- Similar microphone distance throughout

### Text TXT
- UTF-8 plain text works best
- Add punctuation where possible (chunking is sentence-based; punctuation improves prosody)
- Avoid very long unbroken paragraphs

---

## Tuning guide

### Speed (tempo-only)
- `SPEED = 1.0` normal
- `SPEED = 1.10` 10% faster
- `SPEED = 0.90` 10% slower

Implementation: FFmpeg `atempo` (tempo change without pitch shift).
For speeds outside `[0.5, 2.0]`, the notebook chains multiple `atempo` filters.

### MP3 bitrate
Only applies when `OUT_AUDIO` ends with `.mp3`.

Examples:
- `MP3_BITRATE = "96k"` smaller file
- `MP3_BITRATE = "192k"` good default
- `MP3_BITRATE = "256k"` higher quality

### Loudness normalization (`loudnorm`)
Defaults are “podcast/audiobook-ish”:
- `LOUDNORM_I = -16` LUFS
- `LOUDNORM_TP = -1.5` dBTP
- `LOUDNORM_LRA = 11`

If output is too quiet/loud:
- increase loudness: set `LOUDNORM_I` closer to `-14`
- decrease loudness: set `LOUDNORM_I` closer to `-18`

### Silence trimming (`silenceremove`)
- `SILENCE_THRESHOLD_DB`:
  - more negative (e.g., `-50`) = more aggressive trimming
  - less negative (e.g., `-35`) = less aggressive trimming
- `SILENCE_MIN_SEC`:
  - minimum duration of silence to trim at start/end (e.g., `0.2` sec)

---

## Troubleshooting

### Error: `ImportError: cannot import name 'BeamSearchScorer' from 'transformers'`
Fix by pinning transformers (then restart the notebook kernel):

```bash
pip uninstall -y transformers
pip install "transformers<4.57"
```

### FFmpeg not found / MP3 read-write fails
Make sure:
- FFmpeg is installed
- `ffmpeg -version` works in the same terminal you used to launch Jupyter

### Voice similarity is weak / output sounds robotic
Try:
- using a cleaner reference voice sample (less noise/reverb)
- using a longer reference (15–30s)
- lowering `MAX_CHARS_PER_CHUNK` (e.g., 200–240)
- keeping `SPEED` close to `1.0`

---

## Limitations & notices

- Voice cloning quality depends strongly on the reference recording quality.
- This notebook does **not** separate a voice from mixed audio. If your reference MP3 has music or multiple speakers, results will degrade.
- Respect copyright, privacy, and your local laws.

---

## Citation

If you use XTTS in academic work, cite the XTTS paper:

- https://arxiv.org/abs/2406.04904

---

## License

`Mozilla Public License Version 2.0.`