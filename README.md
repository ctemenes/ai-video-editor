# AI Video Editing & Content Workflow

An agentic workflow system for automating video editing, YouTube thumbnail generation, and content research. Built using Claude Code and the 3-layer architecture (Directives, Orchestration, Execution).

## Overview

This project implements three main workflows:

1. **AI Video Editor** - Automatically edits talking-head videos by removing silences, enhancing audio, and adding transitions
2. **AI Thumbnail Generator** - Face-swaps YouTube thumbnails using AI image generation
3. **AI Outlier Detector** - Finds high-performing videos from adjacent niches for content inspiration

All workflows are powered by Claude Code using a structured approach that separates high-level instructions (directives), decision-making (orchestration), and execution (Python scripts).

## Quick Start

### 1. Prerequisites
- Python 3.10+ (✓ Python 3.14.2 detected)
- FFmpeg ([installation guide](SETUP.md#2-ffmpeg-required))
- Node.js 18+ (optional, for 3D transitions)

### 2. Installation

```bash
# Install Python dependencies
python -m pip install -r requirements.txt

# Install Node dependencies (optional, for 3D video effects)
cd execution/video_effects
npm install
cd ../..

# Set up environment variables
copy .env.template .env
# Edit .env and add your API keys
```

### 3. Configure API Keys

Required:
- **ANTHROPIC_API_KEY** - Get from https://console.anthropic.com/
- **NANO_BANANA_API_KEY** - Get from https://nanobanana.xyz/

Optional:
- **AUPHONIC_API_KEY** - For YouTube uploads
- **TUBELAB_API_KEY** - For outlier detection
- **APIFY_API_TOKEN** - For transcript fallback

See [SETUP.md](SETUP.md) for detailed instructions.

## Features

### 🎬 Smart Video Editor

Automatically edit talking-head videos with:
- Neural voice activity detection (Silero VAD)
- Silence removal (configurable threshold)
- Audio enhancement (EQ, compression, normalization)
- Optional color grading (LUT support)
- 3D swivel teaser previews
- Hardware-accelerated encoding

**Example:**
```bash
python execution/jump_cut_vad_parallel.py input.mp4 output.mp4 --enhance-audio
```

See [directives/smart_video_edit.md](directives/smart_video_edit.md) for full documentation.

### 🖼️ AI Thumbnail Recreation

Face-swap YouTube thumbnails with pose-aware matching:
- Analyzes face direction (yaw/pitch)
- Matches best reference photo by pose
- Generates 3 variations per run
- Supports iterative edits

**Example:**
```bash
python execution/recreate_thumbnails.py --youtube "https://youtube.com/watch?v=VIDEO_ID"
```

See [directives/recreate_thumbnails.md](directives/recreate_thumbnails.md) for full documentation.

### 📊 Cross-Niche Outlier Detection

Find high-performing videos from adjacent niches:
- Calculates outlier scores (views vs channel average)
- Generates 3 title variants adapted to your niche
- Outputs to Google Sheets with thumbnails + transcripts
- Uses TubeLab API or yt-dlp

**Example:**
```bash
python execution/scrape_cross_niche_tubelab.py
```

See [directives/cross_niche_outliers.md](directives/cross_niche_outliers.md) for full documentation.

## Architecture

This project uses a **3-layer architecture** to maximize reliability:

### Layer 1: Directive (What to do)
- SOPs written in Markdown, stored in `directives/`
- Define goals, inputs, tools, outputs, and edge cases
- Natural language instructions

### Layer 2: Orchestration (Decision making)
- Claude Code handles intelligent routing
- Reads directives, calls execution scripts in correct order
- Handles errors, asks for clarification, updates directives

### Layer 3: Execution (Doing the work)
- Deterministic Python scripts in `execution/`
- Handle API calls, data processing, file operations
- Reliable, testable, fast

**Why this works:** By pushing complexity into deterministic code, errors don't compound. The AI focuses on decision-making, not implementation.

## Project Structure

```
.
├── CLAUDE.md                 # Agent instructions (3-layer architecture)
├── SETUP.md                  # Detailed setup guide
├── README.md                 # This file
├── requirements.txt          # Python dependencies
├── .env.template             # Environment variable template
│
├── directives/               # High-level SOPs (Layer 1)
│   ├── smart_video_edit.md
│   ├── recreate_thumbnails.md
│   ├── cross_niche_outliers.md
│   ├── jump_cut_vad.md
│   └── pan_3d_transition.md
│
├── execution/                # Python scripts (Layer 3)
│   ├── jump_cut_vad_parallel.py
│   ├── insert_3d_transition.py
│   ├── recreate_thumbnails.py
│   ├── analyze_face_directions.py
│   ├── scrape_cross_niche_tubelab.py
│   ├── scrape_cross_niche_outliers.py
│   ├── simple_video_edit.py
│   └── video_effects/        # Remotion project for 3D transitions
│       ├── package.json
│       └── src/
│
└── .tmp/                     # Temp files (created during setup)
    ├── thumbnails/
    └── reference_photos/
```

## Usage Examples

### Edit a Video

```bash
# Basic editing (silence removal + audio enhancement)
python execution/jump_cut_vad_parallel.py recording.mp4 edited.mp4 --enhance-audio

# Add swivel teaser preview
python execution/insert_3d_transition.py edited.mp4 final.mp4 --bg-image .tmp/bg.png

# One-liner workflow
python execution/jump_cut_vad_parallel.py input.mp4 .tmp/edited.mp4 --enhance-audio && \
python execution/insert_3d_transition.py .tmp/edited.mp4 output.mp4 --bg-image .tmp/bg.png
```

### Generate Thumbnails

```bash
# From YouTube video
python execution/recreate_thumbnails.py --youtube "https://youtube.com/watch?v=VIDEO_ID"

# From local image
python execution/recreate_thumbnails.py --source "thumbnail.jpg"

# Edit pass (refine a generated thumbnail)
python execution/recreate_thumbnails.py --edit ".tmp/thumbnails/generated.png" \
  --prompt "Change text to 'NEW TITLE'. Update colors to teal."
```

### Find Content Ideas

```bash
# Default run (finds ~20 outliers from last 30 days)
python execution/scrape_cross_niche_tubelab.py

# Custom search
python execution/scrape_cross_niche_tubelab.py --terms "business strategy" --queries 3

# Skip transcripts (faster)
python execution/scrape_cross_niche_tubelab.py --skip_transcripts
```

## Workflows

### Video Editing Workflow
1. Record video in OBS or similar
2. Run VAD parallel script to remove silences
3. Add swivel teaser preview
4. Review and adjust if needed
5. Optionally upload to YouTube via Auphonic

### Thumbnail Creation Workflow
1. Run outlier detection to find high-performing videos
2. Pick a video with good thumbnail
3. Run thumbnail recreation with face-swap
4. Generate 3 variations
5. Select best variant and run edit passes
6. Use final thumbnail for your video

### Content Research Workflow
1. Run cross-niche outliers weekly
2. Review results in Google Sheet (sorted by date)
3. Filter by cross-niche score for transferability
4. Study title variants, thumbnails, and transcripts
5. Adapt patterns to your niche

## Customization

### Update Your Channel Niche
Edit `execution/scrape_cross_niche_tubelab.py`:
```python
USER_CHANNEL_NICHE = "AI agents, automation, LangGraph, CrewAI, agentic workflows"
```

### Add Custom Directives
1. Create new markdown file in `directives/`
2. Follow existing directive format
3. Reference it in Claude Code conversations

### Modify Video Settings
Edit script parameters:
```python
# In jump_cut_vad_parallel.py
MIN_SILENCE_DURATION = 0.5  # Minimum gap to cut
PADDING_MS = 50              # Padding around speech
```

## Troubleshooting

### FFmpeg not found
```bash
# Windows (Chocolatey)
choco install ffmpeg

# Manual installation
# See SETUP.md for instructions
```

### Missing API keys
```bash
# Check .env file
cat .env

# Verify keys are set
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print(os.getenv('ANTHROPIC_API_KEY')[:10])"
```

### Permission errors
```bash
# Use python -m pip instead
python -m pip install -r requirements.txt

# Or install to user directory
pip install --user -r requirements.txt
```

See [SETUP.md](SETUP.md) for more troubleshooting tips.

## Requirements

### System Dependencies
- Python 3.10+
- FFmpeg
- Node.js 18+ (optional, for 3D transitions)

### Python Packages
- anthropic
- opencv-python
- mediapipe
- google-generativeai
- gspread
- yt-dlp
- youtube-transcript-api
- See [requirements.txt](requirements.txt) for full list

### Optional Dependencies
- faster-whisper (requires Rust compiler)
- remotion (Node.js, for 3D transitions)

## API Costs

Approximate costs per operation:

| Operation | Model | Cost |
|-----------|-------|------|
| Video editing | - | Free (local processing) |
| Thumbnail generation | Gemini Pro | $0.14-0.24 per image |
| Outlier detection | Claude Sonnet | $0.15-0.25 per outlier |
| Title generation | Claude Sonnet | $0.05-0.10 per title set |

## Contributing

This is a personal workflow system, but feel free to:
- Fork and adapt for your needs
- Submit issues for bugs
- Share improvements and optimizations

## License

MIT License - See individual scripts for specific attributions.

## Credits

Built using:
- Claude Code by Anthropic
- Silero VAD for voice activity detection
- MediaPipe for face pose estimation
- FFmpeg for video processing
- Remotion for 3D transitions
- TubeLab API for YouTube data

## Related Resources

- [Claude Code Documentation](https://docs.anthropic.com/claude/docs)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
- [Remotion Documentation](https://www.remotion.dev/)
- [Original Video Tutorial](transcipt.txt) (if available)

## Support

For questions or issues:
1. Check [SETUP.md](SETUP.md) for setup help
2. Review directive files for workflow details
3. Examine execution scripts for advanced options
4. Open an issue on GitHub

---

**Note:** This project was built using Claude Code and demonstrates the power of agentic workflows for content creation automation.
