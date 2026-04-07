# AI Note Taker

Professional Windows audio recorder with an intuitive GUI that captures mic + system audio simultaneously, then:
1. **Transcribes** using local `faster-whisper` models (no API calls)
2. **Generates AI meeting minutes** using Claude API (optional)

Records as stereo WAV, transcripts saved as plain text, and structured meeting minutes exported as professionally-formatted Word documents using a standard template.

## Quick Start

### Download Installer
Download the latest stub installer from the [GitHub Releases page](https://github.com/AYJ-Systems/ai_note_taker/releases).

Run `AI-Note-Taker-Setup.exe` — it's a small (~15 MB) installer that downloads and installs the full app on first run. No Python or manual setup needed.

On first run, the GUI will prompt you to enter your Claude API key (optional, for AI meeting minutes). The app saves your key securely for future use. Alternatively, you can go to Settings and input your API key there directly.

On first run, the program will download the whisper model which can take a few minutes depending on the model. "Medium" whisper model is default and recommended.

### Main Features in GUI

**Recording Panel:**
- **Select Microphone** — Choose from available mics, or use default
- **Select Speaker** — Choose system audio source
- **Record Button** — Start/stop recording with elapsed time display
- **Files are saved to:**
  - Audio: `Documents/ai_note_taker/recordings/meeting_TIMESTAMP.wav`
  - Transcript: `Documents/ai_note_taker/transcripts/raw/meeting_TIMESTAMP.txt` (if enabled)
  - Minutes: `Documents/ai_note_taker/meeting_minutes/meeting_TIMESTAMP_minutes.docx` + `.txt` (if enabled)

**Recording Library:**
- View all past recordings with timestamps
- Select recordings to transcribe or replay
- Delete recordings to save space
- Configure max recordings to keep

**Transcription:**
- Real-time transcription status and progress
- Choose model size: tiny, base, small, medium, large-v3, turbo
- Auto/GPU/CPU device selection with status indicator
- Automatic output saved to `Documents/ai_note_taker/transcripts/raw/`

**Meeting Minutes (Optional):**
- Auto-generate formatted Word documents from transcripts
- Requires Claude API key (configured on first run)
- Formatted with meeting info, agenda items, action items, and color-coded owners

**Attendee Management:**
- Manage list of possible attendee names
- Claude uses these names to correct transcription errors
- Add/remove names directly from settings panel

**Settings Tab:**
- Theme toggle (light/dark mode)
- Device selection
- Model size configuration
- Recording output directory
- Transcription settings
- Summarization enable/disable

## Output

After recording, files are saved to your Documents folder:
- **Audio**: `Documents/ai_note_taker/recordings/meeting_20260312_222558.wav` (stereo, 48kHz)
- **Transcript**: `Documents/ai_note_taker/transcripts/raw/meeting_20260312_222558.txt` (plain text)
- **Meeting Minutes** (if enabled):
  - `Documents/ai_note_taker/meeting_minutes/meeting_20260312_222558_minutes.txt` (markdown)
  - `Documents/ai_note_taker/meeting_minutes/meeting_20260312_222558_minutes.docx` (formatted Word doc)

### Meeting Minutes Format

The Word document is formatted according to your template and includes:
- **Header Info**: Project, Date, Time, Location, Attendees, Apologies, Objective
- **Agenda Items**: Detailed table with Issues, Decisions, Action Items for each item
- **Questions**: Open questions raised during the meeting
- **Next Meeting**: Scheduled date/time if mentioned
- **Action Items Summary**: Color-coded by owner (🟢 Internal, 🟡 External, 🔵 Cross-party)

## Attendee Name Recognition

When generating meeting minutes, the app passes a list of possible attendee names to Claude so it can correct spelling errors from the Whisper transcription.

Names are stored in `assets/possible_names.csv` and managed via **Names** in the main menu.

Names in the list are treated as *possible* attendees — Claude only uses names that appear in the transcript and does not add unlisted people as attendees.

## faster-whisper Models

| Model    | Size  | Speed | Accuracy |
|----------|-------|-------|----------|
| tiny     | 39M   | Fast  | Lower    |
| base     | 140M  | Fast  | Good     |
| small    | 244M  | ~1x   | Better   |
| medium   | 769M  | ~2x   | Very Good|
| large-v3 | ~1.5G | ~4x   | Best     |
| turbo    | ~809M | Fast  | Very Good|

**First run downloads the model** to the local Hugging Face cache.

**Cache location:**
- Windows: `%USERPROFILE%\.cache\huggingface\hub\`
- Models are cached after first download (one-time only)
- Safe to delete cache to free space, will re-download on next use

## GPU Acceleration

By default, transcription runs on CPU. For NVIDIA GPU acceleration (3–5x faster), follow these steps:

### Enable GPU Transcription

**Step 1: Install CUDA Toolkit 12.x**

Download from [NVIDIA's CUDA Toolkit archive](https://developer.nvidia.com/cuda-12-9-1-download-archive):
- Select **Windows**, **x86_64**, **12.x** (latest 12.x version), **exe (local)**
- Run the installer with default settings

**Step 2: Verify CUDA is installed**
```bash
nvcc --version
```
Should show version 12.x

### Performance

GPU transcription is typically **3–5x faster** than CPU on medium/large models:
- **CPU (int8)**: ~1x speed
- **GPU (float16)**: ~3–5x speed

**CPU is fine for:** Quick notes, small models (tiny/base)
**GPU is recommended for:** Large models (medium/large-v3), long recordings

## Limitations

- Speech-only: Music and non-speech audio are filtered out
- Accuracy depends on audio quality (noise, clarity, volume)

## Troubleshooting

### Recording Issues

**No audio captured**
- Open the GUI and check the **Select Microphone** and **Select Speaker** dropdowns
- Verify all available devices are showing in the dropdowns
- Verify mic and speaker are working in Windows Settings

**Audio is quiet or distorted**
- Check Windows volume levels for mic and system audio
- Try adjusting gain in device settings
- Try a different microphone from the dropdown

### Transcription Issues

**Transcription not running**
- Verify `transcription.enabled: true` in `config.yaml`
- Check `faster-whisper` is installed: `pip install faster-whisper`
- Verify you have enough disk space for model cache (~1-5GB)

**Transcription is very slow**
- First run downloads the model (~1-5GB) — wait for completion
- Subsequent runs use cached model
- Try smaller model: change `model_size: tiny` or `base` in `config.yaml`
- **Enable GPU**: Install CUDA Toolkit 12.x for 3–5x speedup (see [GPU Acceleration](#gpu-acceleration) section)

**GPU errors (CUDA not found, cublas DLL missing, etc.)**
- Install **CUDA Toolkit 12.x** from [NVIDIA's archive](https://developer.nvidia.com/cuda-downloads-archive)
- Verify CUDA is available: 
  ```bash
  nvcc --version
  ```
  Should show version 12.x
- Test ctranslate2 support:
  ```bash
  python -c "import ctranslate2; print(ctranslate2.get_supported_compute_types('cuda'))"
  ```
  Should print: `['float32', 'float16', 'int8_float16', 'int8']`
- If still failing, fall back to CPU: change `device: cpu` in `config.yaml`

**Transcription fails on audio file**
- For non-WAV formats (MP3, M4A, FLAC): install ffmpeg from https://ffmpeg.org/download.html
- Add ffmpeg to Windows PATH or place in `utils/` folder

### Meeting Minutes Issues

**"No API key provided" error**
- Click the API Key field in Settings and enter your key from https://console.anthropic.com
- Key should start with `sk-ant-`
- Click "Save API Key" to store it securely

**Invalid/expired API key**
- Get a new key from https://console.anthropic.com
- Clear the key field in Settings and enter the new one
- Click "Save API Key" and restart the app

**Word document formatting issues**
- Ensure `assets/meeting_minutes_TEMPLATE.docx` exists in the app directory
- Ensure `assets/note_taker_prompt.txt` exists (meeting minutes prompt template)
- Try re-transcribing the recording to regenerate minutes

## Project Structure

```
ai_note_taker/
├── gui.py               # Main GUI application (PySide6) — primary entry point
├── main.py              # Core recording functionality (used by GUI)
├── transcribe.py        # Transcription module (also runnable standalone)
├── summarize.py         # Meeting minutes generation (Claude API)
├── docx_generator.py    # Word document formatter
├── config_loader.py     # Centralized config & .env loader
├── auth_manager.py      # Authentication for API key management
├── download_helper.py   # Update/version checking
├── build_exe.py         # Build script for creating .exe
├── build_stub_installer.py  # Create Windows installer
├── config.yaml          # Configuration file
├── .env                 # API credentials (auto-created on first run, in .gitignore)
├── .env.example         # Template for .env file
├── requirements.txt     # Python dependencies
├── README.md            # This file
├── assets/              # Templates and prompts
│   ├── note_taker_prompt.txt           # Meeting minutes prompt
│   ├── meeting_minutes_TEMPLATE.docx   # Word doc template
│   └── possible_names.csv              # Attendee names for AI name recognition
├── recordings/          # Audio output directory (auto-created)
├── transcripts/         # Transcript output directory (auto-created)
├── meeting_minutes/     # Meeting minutes output (auto-created)
├── utils/               # Utilities (ffmpeg, etc.)
└── dist/                # Build output (created by build_exe.py)
    └── AI Note Taker/   # Distribute this entire folder (zip it)
```

## License

This project is licensed under the **Creative Commons Attribution–NonCommercial 4.0
International License (CC BY-NC 4.0)**.

You are free to:
- Use the software for personal and educational purposes
- Modify and experiment with the source code
- Share and redistribute the software

Under the following terms:
- **NonCommercial** — You may not use this project for commercial purposes,
  including selling the software or incorporating it into commercial products
  or services.
- **Attribution** — You must give appropriate credit when redistributing.

See [LICENSE](LICENSE) file for full details.
