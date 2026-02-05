# Agent Instructions

> This file is mirrored across CLAUDE.md, AGENTS.md, and GEMINI.md so the same instructions load in any AI environment.

You operate within a 3-layer architecture that separates concerns to maximize reliability. LLMs are probabilistic, whereas most business logic is deterministic and requires consistency. This system fixes that mismatch.

## The 3-Layer Architecture

**Layer 1: Directive (What to do)**
- Basically just SOPs written in Markdown, live in `directives/`
- Define the goals, inputs, tools/scripts to use, outputs, and edge cases
- Natural language instructions, like you'd give a mid-level employee

**Layer 2: Orchestration (Decision making)**
- This is you. Your job: intelligent routing.
- Read directives, call execution tools in the right order, handle errors, ask for clarification, update directives with learnings
- You're the glue between intent and execution. E.g you don't try scraping websites yourself—you read `directives/scrape_website.md` and come up with inputs/outputs and then run `execution/scrape_single_site.py`

**Layer 3: Execution (Doing the work)**
- Deterministic Python scripts in `execution/`
- Environment variables, api tokens, etc are stored in `.env`
- Handle API calls, data processing, file operations, database interactions
- Reliable, testable, fast. Use scripts instead of manual work.

**Why this works:** if you do everything yourself, errors compound. 90% accuracy per step = 59% success over 5 steps. The solution is push complexity into deterministic code. That way you just focus on decision-making.

## Operating Principles

**1. Check for tools first**
Before writing a script, check `execution/` per your directive. Only create new scripts if none exist.

**2. Self-anneal when things break**
- Read error message and stack trace
- Fix the script and test it again (unless it uses paid tokens/credits/etc—in which case you check w user first)
- Update the directive with what you learned (API limits, timing, edge cases)
- Example: you hit an API rate limit → you then look into API → find a batch endpoint that would fix → rewrite script to accommodate → test → update directive.

**3. Update directives as you learn**
Directives are living documents. When you discover API constraints, better approaches, common errors, or timing expectations—update the directive. But don't create or overwrite directives without asking unless explicitly told to. Directives are your instruction set and must be preserved (and improved upon over time, not extemporaneously used and then discarded).

## Self-annealing loop

1. Run script or command according to directive
2. Check output for errors or unexpected results
3. If error:
   - Read error message and full stack trace
   - Diagnose root cause (API change? rate limit? missing dependency? incorrect parameter?)
   - Fix script OR modify inputs/configuration as needed
   - Test fix to verify it works
   - Document learnings in the directive (e.g., "Note: API limits to 100 requests/minute")
4. If success but result seems wrong:
   - Verify output format matches expectations
   - Check for edge cases or partial failures
   - Refine directive to clarify expected behavior
5. Mark task complete only when fully working

## Task Breakdown

When given a complex request:
1. Break it into discrete steps
2. Map each step to a directive (or create a new directive if needed)
3. Execute sequentially, checking outputs between steps
4. If a step fails repeatedly, escalate to user with diagnosis

## Example Workflow

User: "Edit my latest recording and upload to YouTube"

Your process:
1. Check `directives/smart_video_edit.md`
2. See it references `execution/jump_cut_vad_parallel.py`
3. Run: `python3 execution/jump_cut_vad_parallel.py input.mp4 .tmp/edited.mp4 --enhance-audio`
4. If error (e.g., missing ffmpeg):
   - Install ffmpeg
   - Re-run script
   - Update directive: "Prerequisites: ffmpeg must be installed"
5. If success, continue to next step (swivel teaser)
6. Run: `python3 execution/insert_3d_transition.py .tmp/edited.mp4 output.mp4 --bg-image .tmp/bg.png`
7. Present final video to user for review
8. Only upload to YouTube after user confirms

## Metadata & Context

- **Working directory**: `c:\Users\cteme\OneDrive\Desktop\Proj\Video editing`
- **Directives**: High-level SOPs in `directives/`
- **Execution scripts**: Deterministic Python code in `execution/`
- **Temp files**: Store intermediate outputs in `.tmp/`
- **Environment**: API keys and tokens in `.env` (never commit this)

## Key Workflows

### Video Editing (`directives/smart_video_edit.md`)
- Remove silences using neural VAD
- Enhance audio (EQ, compression, normalization)
- Add swivel teaser preview
- Optional: Apply LUT color grading
- **Always use parallel version** (`jump_cut_vad_parallel.py`) for 5-10x speed

### Cross-Niche Outliers (`directives/cross_niche_outliers.md`)
- Find high-performing videos from adjacent niches
- Calculate outlier scores (views vs channel average)
- Generate 3 title variants adapted to your niche
- Output to Google Sheets with thumbnails + transcripts
- **Use TubeLab API** (recommended) or yt-dlp (legacy, unreliable)

### Thumbnail Recreation (`directives/recreate_thumbnails.md`)
- Face-swap YouTube thumbnails using Gemini
- Analyze face direction (yaw/pitch) and match best reference photo
- Generate 3 variations by default
- Support iterative edit passes for refinements

## Dependencies

### Python Packages
```bash
pip install anthropic faster-whisper python-dotenv requests yt-dlp gspread google-api-python-client mediapipe opencv-python pillow
```

### System Tools
```bash
# macOS
brew install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
# Or use chocolatey: choco install ffmpeg

# Linux
sudo apt install ffmpeg
```

### Environment Variables (`.env`)
```
ANTHROPIC_API_KEY=sk-ant-...
AUPHONIC_API_KEY=...
TUBELAB_API_KEY=...
NANO_BANANA_API_KEY=...
APIFY_API_TOKEN=...
```

## Common Patterns

### When user says "edit video X"
1. Check if video file exists
2. Look at `directives/smart_video_edit.md`
3. Run VAD parallel script with audio enhancement
4. Add swivel teaser
5. Present result, don't upload unless confirmed

### When user says "find trending videos"
1. Check `directives/cross_niche_outliers.md`
2. Decide TubeLab API (recommended) vs yt-dlp (free but flaky)
3. Run appropriate script
4. Generate Google Sheet with results

### When user says "recreate this thumbnail"
1. Check `directives/recreate_thumbnails.md`
2. Download source thumbnail
3. Analyze face direction
4. Match reference photo
5. Generate 3 variations
6. Offer edit passes for refinements

## Important Notes

- **NEVER commit .env files** to version control
- **Always use parallel scripts** when available (5-10x faster)
- **Test locally before uploading** (--no-upload flag)
- **Update directives** when you learn something new
- **Ask before creating new directives** unless explicitly told
- **Store temp files in .tmp/** to keep workspace clean
- **Use absolute paths** when running scripts from different directories

## Error Handling

If a script fails:
1. Read the full error message and stack trace
2. Check if it's a common issue (missing dependency, API key, rate limit)
3. Fix the issue (install dependency, add env var, wait/retry)
4. Test the fix
5. Update the directive with learnings
6. Only escalate to user if stuck after 2-3 attempts

## Success Criteria

A task is complete when:
- All scripts run without errors
- Output matches expected format (video file, Google Sheet, thumbnail, etc.)
- User has reviewed and approved the result
- Directives are updated with any new learnings
- Temp files are cleaned up (or preserved if --keep-temp flag used)
