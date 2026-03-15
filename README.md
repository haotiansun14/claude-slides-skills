# Claude Slides Skills

Custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for creating presentations.

## Skills

### frontend-slides

Create stunning, animation-rich HTML presentations from scratch or by converting PowerPoint files. Zero-dependency single HTML files that run entirely in the browser.

**Features:**
- Zero dependencies — single HTML files with inline CSS/JS
- 12 curated style presets (dark, light, and specialty themes)
- Viewport-fitting guaranteed — every slide fits exactly in one screen
- Responsive across desktop, tablet, and mobile
- Show-don't-tell aesthetic discovery workflow

**Usage:** `/frontend-slides`

### pptx-composer

Compose pixel-accurate, fully editable PPTX presentations by visually reading HTML/PDF slides and recreating them with native `python-pptx` shapes.

**Features:**
- Visual-first approach — Claude reads each slide as an image, then recreates it
- Fully editable output — native PowerPoint shapes, not embedded images
- High fidelity color and layout matching
- Speaker notes preserved
- Reusable shape composition helpers (cards, code blocks, diagrams, chips, etc.)

**Prerequisites:** `pip install python-pptx Pillow playwright`

**Usage:** `/pptx-composer`

## Installation

Copy the skill directories into your Claude Code skills folder:

```bash
# Clone this repo
git clone https://github.com/haotiansun14/claude-slides-skills.git

# Copy skills to your Claude Code config
cp -r claude-slides-skills/frontend-slides ~/.claude/skills/
cp -r claude-slides-skills/pptx-composer ~/.claude/skills/
```

## License

MIT
