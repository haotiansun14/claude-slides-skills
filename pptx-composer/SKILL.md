---
name: pptx-composer
description: Compose pixel-accurate editable PPTX presentations by visually reading HTML/PDF slides and recreating them with native python-pptx shapes. Use when converting web presentations to PowerPoint, or when the user wants an editable PPTX that matches an existing HTML/PDF presentation.
triggers:
  - pptx-composer
  - compose pptx
  - html to pptx
  - convert slides to pptx
  - make pptx from
  - editable powerpoint
---

# pptx-composer — Visual PPTX Composer

Compose pixel-accurate, fully editable PPTX presentations by visually reading rendered slides (PDF/screenshots) and HTML source, then recreating each slide with native python-pptx shapes.

## Core Philosophy

**See it, then build it.** Instead of automated format conversion (which loses fidelity), Claude Code reads each slide visually, understands the layout, and writes python-pptx code that recreates it with native editable shapes. This leverages Claude's multimodal understanding for much higher fidelity than any parser.

## When to Use

- Converting an HTML presentation to editable PPTX
- User has a PDF and wants an editable PowerPoint version
- User wants PPTX that closely matches a rendered web presentation
- Any case where visual fidelity + editability are both required

## Prerequisites

```bash
pip install python-pptx Pillow playwright
playwright install chromium  # only if capturing from HTML
```

## Workflow

### Phase 1: Gather Reference Material

**Required:** At least one of these:
- HTML presentation file → read source + capture screenshots
- PDF file → read pages visually
- Screenshots/images of slides

**Steps:**

1. **If HTML source exists**, read it to extract:
   - Exact text content (titles, bullets, code blocks, labels)
   - Color palette (CSS variables)
   - Font choices
   - Slide structure and ordering
   - Speaker notes / transcript data attributes

2. **Generate visual references** — For each slide, get a high-quality image:
   - If HTML: Use Playwright to capture each slide at 2x resolution
   - If PDF: Read each page with the Read tool (Claude is multimodal)
   - Save captures to a temporary directory

3. **View each slide image** using the Read tool to see exactly how it looks

### Phase 2: Analyze Design System

Before building any slides, analyze the visual design system across all slides:

```
Design System Checklist:
- [ ] Background color (usually white or dark)
- [ ] Primary colors (extract from CSS or visuals)
- [ ] Font family mapping (web font → PowerPoint font)
- [ ] Title style (size, weight, color, position)
- [ ] Subtitle style
- [ ] Body text style
- [ ] Card/box style (fill, border, corner radius, shadow)
- [ ] Code block style (background, font, border)
- [ ] Diagram style (box colors, arrow styles, label positioning)
- [ ] Chip/tag style
- [ ] Accent elements (dividers, dots, numbered circles)
- [ ] Slide dimensions (widescreen 16:9 = 13.333" × 7.5")
```

Create a constants section in Python:
```python
from pptx import Presentation
from pptx.util import Inches, Pt, Emu
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN
from pptx.enum.shapes import MSO_SHAPE, MSO_CONNECTOR_TYPE
from pptx.oxml.ns import qn
from lxml import etree

# ── Design System Constants ──────────────────────────────
SLIDE_W = Inches(13.333)
SLIDE_H = Inches(7.5)

# Colors (extract from HTML CSS variables)
BLUE   = RGBColor(0x05, 0x50, 0xAE)
GREEN  = RGBColor(0x11, 0x63, 0x29)
# ... etc

# Fonts
FONT_HEADING = "Calibri"  # Map from web font
FONT_BODY    = "Calibri"
FONT_CODE    = "Consolas"

# Standard positions
MARGIN_LEFT = Inches(0.8)
HEADER_Y    = Inches(0.3)
SUBTITLE_Y  = Inches(0.85)
CONTENT_Y   = Inches(1.5)
```

### Phase 3: Build Each Slide

For **each slide**, follow this process:

#### Step 3.1: View the Reference
Use the Read tool to view the slide image. Study:
- Overall layout (single column, two-column, centered)
- What elements exist (title, subtitle, cards, diagram, code, callout, stats)
- Relative positioning and sizing
- Colors and emphasis

#### Step 3.2: Read the HTML Source
Get the exact text content from HTML. Don't guess — copy text verbatim.

#### Step 3.3: Compose with python-pptx
Write a builder function for the slide:

```python
def build_slide_XX(prs, notes=""):
    """[Slide title] — [brief description]."""
    slide = prs.slides.add_slide(prs.slide_layouts[6])  # Blank layout
    # White background
    slide.background.fill.solid()
    slide.background.fill.fore_color.rgb = RGBColor(0xFF, 0xFF, 0xFF)

    # Title
    _txt(slide, MARGIN_LEFT, HEADER_Y, Inches(11), Inches(0.6),
         "Slide Title", size=36, bold=True, color=TEXT)
    # Subtitle
    _txt(slide, MARGIN_LEFT, SUBTITLE_Y, Inches(11), Inches(0.4),
         "Subtitle text here", size=18, color=DIM)

    # ... content shapes ...

    # Speaker notes
    if notes:
        slide.notes_slide.notes_text_frame.text = notes
```

### Shape Composition Reference

#### Text Box
```python
def _txt(slide, left, top, w, h, text, size=18, bold=False, color=TEXT,
         align=PP_ALIGN.LEFT, font=FONT_BODY):
    box = slide.shapes.add_textbox(left, top, w, h)
    tf = box.text_frame
    tf.word_wrap = True
    p = tf.paragraphs[0]
    p.text = text
    p.font.size = Pt(size)
    p.font.bold = bold
    p.font.color.rgb = color
    p.font.name = font
    p.alignment = align
    return box
```

#### Card with Accent Border
```python
def _card(slide, left, top, w, h, title, desc, accent=BLUE):
    # Card background
    shape = slide.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, left, top, w, h)
    shape.fill.solid()
    shape.fill.fore_color.rgb = RGBColor(0xFF, 0xFF, 0xFF)
    shape.line.color.rgb = RGBColor(0xD1, 0xD9, 0xE0)
    shape.line.width = Pt(1)
    # Left accent bar
    bar = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE,
        left, top + Inches(0.05), Inches(0.06), h - Inches(0.1))
    bar.fill.solid()
    bar.fill.fore_color.rgb = accent
    bar.line.fill.background()
    # Title + description
    _txt(slide, left + Inches(0.2), top + Inches(0.08),
         w - Inches(0.3), Inches(0.35), title, size=16, bold=True, color=accent)
    _txt(slide, left + Inches(0.2), top + Inches(0.4),
         w - Inches(0.3), h - Inches(0.45), desc, size=13, color=DIM)
```

#### Code Block
```python
def _code(slide, left, top, w, h, text):
    shape = slide.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, left, top, w, h)
    shape.fill.solid()
    shape.fill.fore_color.rgb = RGBColor(0xF0, 0xF2, 0xF5)
    shape.line.color.rgb = RGBColor(0xD1, 0xD9, 0xE0)
    shape.line.width = Pt(1)
    tf = shape.text_frame
    tf.word_wrap = True
    tf.margin_left = Inches(0.15)
    tf.margin_top = Inches(0.1)
    for j, line in enumerate(text.split("\n")):
        p = tf.paragraphs[0] if j == 0 else tf.add_paragraph()
        p.text = line
        p.font.size = Pt(11)
        p.font.color.rgb = TEXT
        p.font.name = FONT_CODE
        p.space_after = Pt(1)
```

#### Connector with Arrow
```python
def _arrow(slide, x1, y1, x2, y2, color=DIM, width=2.0):
    conn = slide.shapes.add_connector(
        MSO_CONNECTOR_TYPE.STRAIGHT, x1, y1, x2, y2)
    conn.line.color.rgb = color
    conn.line.width = Pt(width)
    # Add arrowhead via OOXML
    ln = conn._element.spPr.find(qn('a:ln'))
    if ln is None:
        ln = etree.SubElement(conn._element.spPr, qn('a:ln'))
    tail = etree.SubElement(ln, qn('a:tailEnd'))
    tail.set('type', 'triangle')
    tail.set('w', 'med')
    tail.set('len', 'med')
```

#### Diagram Box (rounded rect with fill + stroke)
```python
def _diagram_box(slide, left, top, w, h, title, subtitle="",
                 fill_color=None, stroke_color=BLUE, text_color=BLUE):
    shape = slide.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, left, top, w, h)
    if fill_color:
        shape.fill.solid()
        shape.fill.fore_color.rgb = fill_color
    else:
        shape.fill.background()
    shape.line.color.rgb = stroke_color
    shape.line.width = Pt(2)
    # Title centered
    _txt(slide, left, top + Inches(0.08), w, Inches(0.3),
         title, size=16, bold=True, color=text_color, align=PP_ALIGN.CENTER)
    if subtitle:
        _txt(slide, left, top + Inches(0.35), w, Inches(0.25),
             subtitle, size=12, color=DIM, align=PP_ALIGN.CENTER)
```

#### Chip / Tag
```python
def _chip(slide, left, top, label, color=BLUE, width=Inches(1.55)):
    shape = slide.shapes.add_shape(
        MSO_SHAPE.ROUNDED_RECTANGLE, left, top, width, Inches(0.38))
    shape.fill.solid()
    shape.fill.fore_color.rgb = RGBColor(0xF6, 0xF8, 0xFA)
    shape.line.color.rgb = color
    shape.line.width = Pt(1)
    p = shape.text_frame.paragraphs[0]
    p.text = label
    p.font.size = Pt(12)
    p.font.bold = True
    p.font.color.rgb = color
    p.font.name = FONT_BODY
    p.alignment = PP_ALIGN.CENTER
```

#### Numbered Circle
```python
def _circle(slide, left, top, num, color, size=Inches(0.45)):
    shape = slide.shapes.add_shape(MSO_SHAPE.OVAL, left, top, size, size)
    shape.fill.solid()
    shape.fill.fore_color.rgb = color
    shape.line.fill.background()
    p = shape.text_frame.paragraphs[0]
    p.text = str(num)
    p.font.size = Pt(18)
    p.font.bold = True
    p.font.color.rgb = RGBColor(0xFF, 0xFF, 0xFF)
    p.alignment = PP_ALIGN.CENTER
```

#### Dashed Border Rectangle (extensibility hints)
```python
def _dashed_box(slide, left, top, w, h, stroke_color=BLUE):
    shape = slide.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, left, top, w, h)
    shape.fill.background()
    shape.line.color.rgb = stroke_color
    shape.line.width = Pt(1.2)
    # Set dash style via OOXML
    ln = shape._element.spPr.find(qn('a:ln'))
    if ln is not None:
        etree.SubElement(ln, qn('a:prstDash')).set('val', 'dash')
```

### Phase 4: Compose Diagrams

Diagrams are the hardest part. For each diagram:

1. **View the reference image closely** — note every box, arrow, and label
2. **Plan the layout on a grid** — estimate positions in Inches
3. **Build bottom-up:**
   - Background containers first (dashed outlines, section boxes)
   - Main boxes (diagram nodes)
   - Connectors and arrows
   - Text labels last (on top)
4. **Use consistent spacing** — align to a grid (e.g., 0.25" increments)

**Positioning strategy for diagrams:**
- Place diagrams within a content area (typically Inches(0.8) to Inches(12.5) width)
- Use the SVG viewBox aspect ratio to calculate height
- Map SVG coordinates proportionally: `pptx_x = left + (svg_x / svg_width) * pptx_width`

### Phase 5: Assemble and Verify

```python
def main():
    prs = Presentation()
    prs.slide_width = SLIDE_W
    prs.slide_height = SLIDE_H

    # Build all slides in order
    build_slide_01(prs, notes="...")
    build_slide_02(prs, notes="...")
    # ... etc

    prs.save("output.pptx")
    print(f"Saved: output.pptx ({os.path.getsize('output.pptx') / 1024:.0f} KB)")
```

After generating:
1. Open the PPTX: `open output.pptx`
2. View each slide in PowerPoint
3. Compare side-by-side with the original HTML/PDF
4. Adjust positions, sizes, colors as needed
5. Re-run until satisfied

## Quality Checklist

For each slide, verify:
- [ ] All text matches the original exactly (no typos, no missing content)
- [ ] Colors match the HTML color palette
- [ ] Layout proportions are close to the original
- [ ] Cards, boxes, and shapes have correct fills and borders
- [ ] Diagrams have all boxes, arrows, and labels
- [ ] Code blocks use monospace font with correct background
- [ ] Speaker notes are included
- [ ] Everything is editable (no embedded images for text/diagrams)

## Tips for High Fidelity

1. **Read the PDF page before coding each slide** — don't code from memory
2. **Copy text verbatim from HTML** — never retype or paraphrase
3. **Use the HTML color values exactly** — don't approximate
4. **Position elements relative to each other** — not absolute guesses
5. **Test frequently** — generate and open after every 2-3 slides
6. **Iterate on diagram slides** — these need the most visual tuning

## Limitations

- **Font rendering** differs between PowerPoint and browsers
- **Gradient text** (CSS background-clip: text) has no python-pptx equivalent → use solid color
- **Semi-transparent fills** require OOXML manipulation or can be composited to solid colors
- **Exact corner radii** may not match (PowerPoint default differs from CSS border-radius)
- **SVG icons/illustrations** cannot be converted → use closest MSO_SHAPE or describe with shapes
- **CSS grid/flexbox** layouts must be manually translated to absolute positions

## Example Invocation

```
/pptx-composer doc/slides/slides.html --pdf doc/slides/slides.pdf --output doc/slides/slides.pptx
```

The skill reads both sources, views each slide visually, and generates a complete
python-pptx script that produces an editable PPTX matching the original as closely as possible.
