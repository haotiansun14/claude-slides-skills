# Claude Slides Skills

A complete toolkit for creating presentations with [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — from HTML slides to editable PowerPoint files.

## Skills Included (Custom)

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

## Recommended Official Plugins

For the full end-to-end slides workflow, install these official Anthropic plugins alongside the custom skills above:

| Plugin | Skills | What it adds |
|--------|--------|--------------|
| **document-skills** | `pptx`, `frontend-design`, `web-artifacts-builder`, `theme-factory` | PPTX creation/editing, frontend UI design, React artifact bundling, 10 professional themes |
| **frontend-design** | `frontend-design` | Standalone production-grade frontend interface design |

### What each official skill provides

- **`document-skills:pptx`** — Create, read, edit, and convert `.pptx` files using `pptxgenjs` and `python-pptx`. Includes scripts for thumbnail generation, XML unpacking, and LibreOffice conversion.
- **`document-skills:frontend-design`** — Distinctive, production-grade frontend interfaces with bold aesthetic direction. Avoids generic "AI slop" aesthetics.
- **`document-skills:web-artifacts-builder`** — Multi-component React + Tailwind + shadcn/ui artifacts bundled into single HTML files. Full build pipeline with init and bundle scripts.
- **`document-skills:theme-factory`** — 10 curated color/font themes (Ocean Depths, Sunset Boulevard, Forest Canopy, etc.) that can be applied to any artifact — slides, docs, HTML pages.

## Installation

### Step 1: Install custom skills (this repo)

```bash
# Clone this repo
git clone https://github.com/haotiansun14/claude-slides-skills.git

# Copy skills to your Claude Code config
cp -r claude-slides-skills/frontend-slides ~/.claude/skills/
cp -r claude-slides-skills/pptx-composer ~/.claude/skills/
```

### Step 2: Install official plugins

In Claude Code, install the official plugins:

```
/install-plugin document-skills
/install-plugin frontend-design
```

Or add them manually in your Claude Code settings under Plugins.

### Step 3: Verify

Start a Claude Code session and check that all skills are available:
- `/frontend-slides` — custom HTML presentation builder
- `/pptx-composer` — custom visual PPTX composer
- `/pptx` — official PPTX skill
- `/frontend-design` — official frontend design skill
- `/theme-factory` — official theme toolkit

## Typical Workflows

### HTML presentation from scratch
1. `/frontend-slides` → pick a style preset → get a polished HTML deck

### HTML to editable PowerPoint
1. `/frontend-slides` → create HTML deck
2. `/pptx-composer` → convert HTML to pixel-accurate editable PPTX

### PowerPoint from scratch with themes
1. `/pptx` → create slides
2. `/theme-factory` → apply a professional theme

### React-based interactive presentation
1. `/web-artifacts-builder` → scaffold React + shadcn/ui project
2. `/frontend-design` → apply distinctive styling

## License

Custom skills in this repo are released under the MIT License.

Official Anthropic plugins (`document-skills`, `frontend-design`) are proprietary — see their respective LICENSE.txt files.
