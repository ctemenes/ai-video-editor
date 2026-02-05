# Setup Guide

This guide will help you set up the video editing agentic workflow system.

## Prerequisites

### 1. Python (✓ Installed)
- Version: Python 3.14.2 detected

### 2. FFmpeg (⚠ Required)
FFmpeg is required for video processing. Install it using one of these methods:

#### Option A: Chocolatey (Recommended for Windows)
```bash
choco install ffmpeg
```

#### Option B: Manual Installation
1. Download from https://ffmpeg.org/download.html
2. Extract to a folder (e.g., `C:\ffmpeg`)
3. Add to PATH:
   - Open System Properties → Environment Variables
   - Edit PATH variable
   - Add `C:\ffmpeg\bin`
4. Restart your terminal
5. Verify: `ffmpeg -version`

### 3. Node.js (Optional - for 3D video transitions)
Only required if using Remotion for 3D video effects.

Download from: https://nodejs.org/

## Installation Steps

### 1. Install Python Dependencies

Most dependencies are already installed. To install any missing ones:

```bash
python -m pip install -r requirements.txt
```

**Note:** `faster-whisper` is commented out due to compilation requirements. If you need transcription:
- On Windows: Install Visual Studio Build Tools or use WSL
- Alternatively: Use cloud-based transcription APIs

### 2. Configure Environment Variables

```bash
# Copy the template
copy .env.template .env

# Edit .env and add your API keys
notepad .env
```

Required API keys:
- **ANTHROPIC_API_KEY**: Get from https://console.anthropic.com/
- **NANO_BANANA_API_KEY**: Get from https://nanobanana.xyz/ (for thumbnail generation)

Optional API keys:
- **AUPHONIC_API_KEY**: For automated YouTube uploads
- **TUBELAB_API_KEY**: For cross-niche outlier detection
- **APIFY_API_TOKEN**: Fallback for transcript fetching

### 3. Create Necessary Directories

```bash
mkdir .tmp
mkdir .tmp\thumbnails
mkdir .tmp\reference_photos
mkdir .tmp\reference_photos\raw
```

### 4. Google Sheets Setup (Optional)

If using cross-niche outliers with Google Sheets:

1. Go to https://console.cloud.google.com/
2. Create a new project
3. Enable Google Sheets API
4. Create OAuth 2.0 credentials
5. Download credentials as `credentials.json`
6. Place in project root
7. First run will prompt for authentication and create `token.json`

### 5. Install Node Dependencies (Optional)

Only if using 3D video transitions:

```bash
cd execution\video_effects
npm install
```

## Verify Installation

### Test FFmpeg
```bash
ffmpeg -version
```

### Test Python Dependencies
```bash
python -c "import anthropic, cv2, mediapipe; print('All imports successful')"
```

### Test API Keys
```bash
# Create test.py
python test.py
```

test.py:
```python
from dotenv import load_dotenv
import os

load_dotenv()

keys = {
    "ANTHROPIC_API_KEY": os.getenv("ANTHROPIC_API_KEY"),
    "NANO_BANANA_API_KEY": os.getenv("NANO_BANANA_API_KEY"),
}

for key, value in keys.items():
    if value:
        print(f"✓ {key}: {'*' * 8}{value[-4:]}")
    else:
        print(f"✗ {key}: Not set")
```

## Quick Start

Once setup is complete:

### 1. Edit a Video
```bash
python execution/jump_cut_vad_parallel.py input.mp4 output.mp4 --enhance-audio
```

### 2. Find Trending Videos
```bash
python execution/scrape_cross_niche_tubelab.py
```

### 3. Recreate Thumbnails
```bash
python execution/recreate_thumbnails.py --youtube "https://youtube.com/watch?v=VIDEO_ID"
```

## Troubleshooting

### "ffmpeg not found"
- Restart terminal after installation
- Check PATH variable includes ffmpeg/bin
- Run `where ffmpeg` to verify

### "No module named 'faster_whisper'"
- This is expected if you skipped faster-whisper installation
- Use cloud transcription APIs instead
- Or install Visual Studio Build Tools and rerun pip install

### "Missing API key"
- Check .env file exists
- Verify keys are not surrounded by quotes
- Check for extra spaces or newlines

### Permission Errors
- Run terminal as Administrator
- Use `python -m pip` instead of `pip`
- Install to user directory with `--user` flag

## Next Steps

1. Read [CLAUDE.md](CLAUDE.md) for agent architecture overview
2. Check `directives/` folder for workflow documentation
3. Try the example workflows above
4. Customize directives for your specific needs

## Getting Help

- GitHub Issues: https://github.com/yourusername/yourrepo/issues
- Check individual directive files for detailed documentation
- Review execution scripts for advanced options
