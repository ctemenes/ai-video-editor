# Quick Start Guide

This guide will get you up and running in 5 minutes.

## Step 1: Install FFmpeg (Required)

FFmpeg is the only external dependency you need to install.

### Windows (Chocolatey - Recommended)
```bash
choco install ffmpeg
```

### Windows (Manual)
1. Download from https://ffmpeg.org/download.html
2. Extract to `C:\ffmpeg`
3. Add `C:\ffmpeg\bin` to your PATH
4. Restart terminal
5. Verify: `ffmpeg -version`

## Step 2: Configure API Keys

```bash
# Copy template
copy .env.template .env

# Edit .env file
notepad .env
```

Add your API keys:
```env
ANTHROPIC_API_KEY=sk-ant-your-key-here
NANO_BANANA_API_KEY=your-key-here
```

Get keys:
- Anthropic: https://console.anthropic.com/
- Nano Banana: https://nanobanana.xyz/

## Step 3: Test Your Setup

### Test Python
```bash
python -c "import anthropic, cv2, mediapipe; print('✓ All packages installed')"
```

### Test FFmpeg
```bash
ffmpeg -version
```

### Test API Keys
```bash
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('✓ API keys loaded' if os.getenv('ANTHROPIC_API_KEY') else '✗ No API key')"
```

## Step 4: Try the Workflows

### Option A: Edit a Video
```bash
# Record a test video first, then:
python execution/jump_cut_vad_parallel.py your_video.mp4 edited.mp4 --enhance-audio
```

### Option B: Find Trending Content
```bash
python execution/scrape_cross_niche_tubelab.py
```

### Option C: Generate Thumbnails
```bash
python execution/recreate_thumbnails.py --youtube "https://youtube.com/watch?v=dQw4w9WgXcQ"
```

## Common Issues

### "ffmpeg not found"
- Restart your terminal
- Check PATH: `echo $PATH` (Mac/Linux) or `echo %PATH%` (Windows)
- Try full path: `"C:\ffmpeg\bin\ffmpeg" -version`

### "No module named X"
```bash
python -m pip install -r requirements.txt
```

### "Missing API key"
- Check `.env` file exists (not `.env.template`)
- Verify no extra quotes around keys
- Check for typos in key names

## Next Steps

1. **Read the architecture**: [CLAUDE.md](CLAUDE.md) explains how everything works
2. **Explore directives**: Check `directives/` for workflow documentation
3. **Customize workflows**: Edit Python scripts in `execution/`
4. **Add reference photos**: Drop photos in `.tmp/reference_photos/raw/` for thumbnail generation

## Getting Help

- Full setup guide: [SETUP.md](SETUP.md)
- Main documentation: [README.md](README.md)
- Workflow details: See individual files in `directives/`

## Quick Command Reference

### Video Editing
```bash
# Basic edit
python execution/jump_cut_vad_parallel.py input.mp4 output.mp4 --enhance-audio

# Add teaser
python execution/insert_3d_transition.py input.mp4 output.mp4 --bg-image .tmp/bg.png

# Full workflow
python execution/jump_cut_vad_parallel.py raw.mp4 .tmp/edited.mp4 --enhance-audio && \
python execution/insert_3d_transition.py .tmp/edited.mp4 final.mp4 --bg-image .tmp/bg.png
```

### Thumbnails
```bash
# Generate
python execution/recreate_thumbnails.py --youtube "URL"

# Edit
python execution/recreate_thumbnails.py --edit "path/to/image.png" --prompt "Change text to X"
```

### Content Research
```bash
# Default
python execution/scrape_cross_niche_tubelab.py

# Custom
python execution/scrape_cross_niche_tubelab.py --terms "business" --queries 3
```

---

That's it! You're ready to start using AI for video editing and content creation.
