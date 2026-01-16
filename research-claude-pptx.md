# Research: Claude Code's Official PowerPoint (PPTX) Skill

**Research Date:** 2026-01-15
**Status:** Complete
**Related:** [research-google-slides-api.md](research-google-slides-api.md) | [slides-interoperability.md](slides-interoperability.md) | [markdown-to-slides.md](markdown-to-slides.md)

---

## Executive Summary

Claude Code includes an official **PPTX skill** that enables programmatic PowerPoint presentation creation and editing. This skill is part of Anthropic's Agent Skills system and uses a combination of **python-pptx** (Python), **PptxGenJS** (JavaScript), and direct **OOXML manipulation** for different workflows. The skill operates within a sandboxed code execution container and supports both creating presentations from scratch and editing existing templates.

---

## 1. What is the Official PowerPoint Skill?

### Overview

The PPTX skill is one of four document-manipulation Agent Skills provided by Anthropic:
- **pptx** - PowerPoint presentations
- **docx** - Word documents
- **xlsx** - Excel spreadsheets
- **pdf** - PDF documents

These skills are "expertise packages" that Claude loads dynamically when relevant tasks are detected. They follow a **progressive disclosure** model:
1. Claude sees skill metadata at startup (name, description)
2. When a presentation task is detected, Claude loads the full SKILL.md instructions
3. Claude executes the skill's code in a sandboxed container

### Availability

| Plan | Access |
|------|--------|
| Free | Not available |
| Pro | Available (rolling out) |
| Max | Available |
| Team | Available |
| Enterprise | Available |
| API | Available (beta) |
| Claude Code CLI | Available via plugins |

---

## 2. Technical Implementation

### Libraries and Tools Used

The PPTX skill uses a **multi-library approach** depending on the workflow:

#### Python Libraries
```python
# Core PPTX manipulation
pip install python-pptx

# Content extraction from presentations
pip install "markitdown[pptx]"

# Secure XML parsing for OOXML editing
pip install defusedxml
```

#### JavaScript/Node.js Libraries
```bash
# Presentation generation (used for HTML-to-PPTX workflow)
npm install -g pptxgenjs

# HTML rendering for slide conversion
npm install -g playwright

# Image processing (SVG rasterization)
npm install -g sharp
```

#### System Dependencies
```bash
# PDF/format conversion
sudo apt-get install libreoffice

# PDF to image conversion (for thumbnails)
sudo apt-get install poppler-utils
```

### Architecture: Code Execution Container

The skill operates within Anthropic's **sandboxed code execution environment**:

```
┌─────────────────────────────────────────────┐
│           Claude Messages API               │
├─────────────────────────────────────────────┤
│     Code Execution Container (Sandbox)      │
│  ┌─────────────────────────────────────┐   │
│  │  /mnt/skills/public/pptx/           │   │
│  │  ├── SKILL.md (workflow docs)       │   │
│  │  ├── html2pptx.md (HTML guide)      │   │
│  │  ├── ooxml.md (XML editing guide)   │   │
│  │  └── scripts/                       │   │
│  │      ├── thumbnail.py               │   │
│  │      ├── rearrange.py               │   │
│  │      ├── inventory.py               │   │
│  │      └── replace.py                 │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  Pre-installed: python-pptx, pptxgenjs,    │
│                 markitdown, playwright      │
└─────────────────────────────────────────────┘
           │
           ▼
    ┌──────────────┐
    │  Files API   │  ← Download generated .pptx
    └──────────────┘
```

---

## 3. Three Core Workflows

### Workflow 1: HTML-to-PPTX (New Presentations from Scratch)

Best for: Creating visually designed presentations without templates.

**Process:**
1. Design each slide as an HTML file (720pt × 405pt for 16:9)
2. Use semantic HTML tags (`<h1>`, `<p>`, `<ul>`, etc.)
3. Rasterize any gradients/icons to PNG using Sharp
4. Convert using html2pptx library
5. Add charts/tables via PptxGenJS API
6. Validate with thumbnail grid

**Example HTML slide:**
```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body {
      width: 720pt;
      height: 405pt;
      margin: 0;
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #1a365d 0%, #2d5a87 100%);
      color: white;
    }
    h1 { font-size: 48pt; margin: 40pt; }
    .content { padding: 0 40pt; }
  </style>
</head>
<body>
  <h1>Q4 Results Overview</h1>
  <div class="content">
    <ul>
      <li>Revenue: $12.4M (+15% YoY)</li>
      <li>New customers: 847</li>
      <li>NPS Score: 72</li>
    </ul>
  </div>
</body>
</html>
```

**Conversion script:**
```javascript
const { html2pptx } = require('html2pptx');

async function createPresentation() {
  const slides = ['slide1.html', 'slide2.html', 'slide3.html'];
  await html2pptx(slides, 'output.pptx');
}

// Run with: NODE_PATH=$(npm root -g) node script.js
```

### Workflow 2: Template-Based Creation

Best for: Corporate templates, brand consistency, bulk generation.

**Process:**
1. Extract template content: `python -m markitdown template.pptx`
2. Generate thumbnail grid: `python scripts/thumbnail.py template.pptx`
3. Create inventory mapping (which slides serve which purpose)
4. Rearrange slides by index: `python scripts/rearrange.py template.pptx working.pptx 0,5,5,12,3`
5. Extract text inventory: `python scripts/inventory.py working.pptx inventory.json`
6. Create replacement JSON
7. Apply replacements: `python scripts/replace.py working.pptx replacements.json output.pptx`

**Replacement JSON structure:**
```json
{
  "slides": [
    {
      "slide_index": 0,
      "shapes": [
        {
          "shape_id": "Title 1",
          "paragraphs": [
            {
              "text": "Q4 2025 Results",
              "alignment": "CENTER",
              "bold": true,
              "font_size": 44
            }
          ]
        }
      ]
    }
  ]
}
```

### Workflow 3: OOXML Direct Editing

Best for: Precise control, complex modifications, preserving exact formatting.

**Process:**
1. Unpack: `python ooxml/scripts/unpack.py presentation.pptx unpacked/`
2. Edit XML files directly (e.g., `ppt/slides/slide1.xml`)
3. Validate: `python ooxml/scripts/validate.py unpacked/ --original presentation.pptx`
4. Repack: `python ooxml/scripts/pack.py unpacked/ output.pptx`

**Key OOXML files:**
```
unpacked/
├── [Content_Types].xml
├── _rels/
├── docProps/
│   ├── app.xml
│   └── core.xml
└── ppt/
    ├── presentation.xml      # Main metadata, slide references
    ├── slides/
    │   ├── slide1.xml        # Individual slide content
    │   └── slide2.xml
    ├── slideLayouts/         # Layout templates
    ├── slideMasters/         # Master slides
    ├── theme/
    │   └── theme1.xml        # Colors, fonts, effects
    └── notesSlides/          # Speaker notes
```

---

## 4. How to Invoke the Skill

### Via Claude.ai / Desktop App

Simply describe what you need:
```
"Create a 5-slide presentation about renewable energy"
"Update the attached template with Q4 sales data from this Excel file"
"Convert this PDF report into a PowerPoint presentation"
```

Claude automatically detects presentation-related tasks and loads the PPTX skill.

### Via API

```python
import anthropic

client = anthropic.Anthropic()

response = client.beta.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    betas=["code-execution-2025-08-25", "skills-2025-10-02"],
    container={
        "skills": [
            {
                "type": "anthropic",
                "skill_id": "pptx",
                "version": "latest"
            }
        ]
    },
    messages=[{
        "role": "user",
        "content": "Create a presentation about AI in education with 5 slides"
    }],
    tools=[{
        "type": "code_execution_20250825",
        "name": "code_execution"
    }]
)

# Extract file_id from response and download
for block in response.content:
    if hasattr(block, 'file_id'):
        file_content = client.beta.files.download(
            file_id=block.file_id,
            betas=["files-api-2025-04-14"]
        )
        with open("presentation.pptx", "wb") as f:
            file_content.write_to_file(f.name)
```

### Via Claude Code CLI

**Option 1: Install from Anthropic's skills repository**
```bash
/plugin marketplace add anthropics/skills
/plugin install document-skills@anthropic-agent-skills
```

**Option 2: Use third-party claude-office-skills**
```bash
git clone https://github.com/tfriedel/claude-office-skills
cd claude-office-skills
# Follow installation instructions
```

**Option 3: Natural language request**
```
> Create a quarterly sales presentation with 5 slides
> Create a powerpoint presentation based on @input/notes.txt
```

---

## 5. Capabilities

### Content Types Supported

| Feature | Support | Notes |
|---------|---------|-------|
| Text (titles, body, bullets) | Full | With formatting control |
| Images | Full | PNG, JPG, SVG (rasterized) |
| Tables | Full | With cell formatting |
| Charts | Full | Bar, line, pie, scatter |
| Shapes | Full | Rectangles, lines, arrows |
| SmartArt | Partial | Basic shapes only |
| Animations | Limited | Via OOXML only |
| Transitions | Limited | Via OOXML only |
| Speaker notes | Full | Read and write |
| Comments | Full | Via OOXML |
| Embedded video/audio | Limited | URL references only |
| Hyperlinks | Full | Internal and external |

### Styling Control

**Typography:**
- Web-safe fonts only: Arial, Helvetica, Times New Roman, Georgia, Courier New, Verdana, Tahoma, Trebuchet MS, Impact
- Font size, bold, italic, underline
- Text alignment (left, center, right, justify)
- Line spacing, paragraph spacing
- Bullet styles and indentation levels

**Colors:**
- RGB hex values
- Theme color references
- Background fills (solid, gradient via HTML workflow)
- Shape fills and borders

**Layouts:**
- Custom positioning (EMUs or points)
- Multi-column layouts
- Header/footer areas
- Slide backgrounds

### Design Guidelines from Skill

The skill includes 17+ predefined color palettes:
- Classic Blue: `#1a365d`, `#2d5a87`, `#4a90c2`
- Teal & Coral: `#0d9488`, `#f97316`
- Deep Purple & Emerald: `#7c3aed`, `#10b981`
- Black & Gold: `#1a1a1a`, `#d4af37`
- And more...

---

## 6. Limitations

### File Size and Context Limits

| Limit | Value |
|-------|-------|
| Maximum file size | 30 MB (upload + download combined) |
| Context window | 200K tokens (500K for Enterprise Sonnet 4.5) |
| Practical slide count | ~50-100 slides for typical content |

**Note:** There's no official guarantee for slide counts. The actual limit depends on content complexity and token usage, not just file size.

### Technical Limitations

1. **Font embedding:** Web-safe fonts only; custom fonts may render differently on recipient's system
2. **Animation complexity:** Advanced animations require direct OOXML editing
3. **Master slide editing:** Limited support for modifying slide masters
4. **Copy/paste slides:** python-pptx doesn't natively support copying slides between presentations
5. **Embedded objects:** Limited support for Excel charts linked to source data
6. **Video playback:** Can embed references but not actual video files
7. **Accessibility:** No built-in alt-text generation for images

### Workflow-Specific Limitations

**HTML-to-PPTX:**
- Gradients must be rasterized as PNG first
- Complex CSS may not translate perfectly
- SVGs must be converted to PNG

**Template-based:**
- Only replaces text in identified shapes
- Complex structural changes require OOXML workflow
- May lose some formatting on heavily customized templates

**OOXML editing:**
- Steep learning curve
- Easy to create invalid XML
- Requires validation after each edit

### Platform Limitations

- **Claude.ai:** No centralized admin management of custom Skills
- **Claude Code:** Sandboxing not fully implemented (security consideration)
- **API:** Requires downloading files via separate Files API call

---

## 7. Comparison: Claude PPTX vs Alternatives

### vs Direct python-pptx

| Aspect | Claude PPTX Skill | Direct python-pptx |
|--------|-------------------|-------------------|
| Learning curve | Natural language | Python API |
| Design assistance | AI-guided | Manual |
| Template handling | Guided workflow | Full control |
| Chart creation | Automated | Manual |
| Debugging | Abstract | Direct |
| Customization | Constrained | Unlimited |

### vs Google Slides API

See [research-google-slides-api.md](research-google-slides-api.md) for full comparison.

| Aspect | Claude PPTX | Google Slides API |
|--------|-------------|-------------------|
| Output format | .pptx | Google Slides (+ export) |
| Authentication | API key | OAuth 2.0 |
| Real-time collab | No | Yes |
| Template gallery | Manual upload | Built-in |
| Offline support | Full | Limited |
| Version control | Manual | Automatic |

### vs Marp / Slidev / reveal.js

See [markdown-to-slides.md](markdown-to-slides.md) for Markdown parsing approaches.

| Aspect | Claude PPTX | Markdown-to-Slides |
|--------|-------------|-------------------|
| Input format | Natural language | Markdown |
| Export formats | .pptx | HTML, PDF, PPTX |
| Version control | Manual | Git-friendly |
| Presenter mode | PowerPoint | Web-based |
| Design control | AI-assisted | Theme-based |
| Reproducibility | Variable | Deterministic |

---

## 8. Best Practices

### Prompt Engineering

**Good prompts:**
```
"Create a 5-slide presentation about renewable energy. Use a blue/green
color scheme. Include: title slide, overview of solar power, wind energy
statistics, comparison chart, and conclusion with call to action."
```

**Better prompts (with structure):**
```
"Create a presentation using the attached template.pptx.

Content:
- Slide 1: Title "Q4 2025 Results" with subtitle "Finance Team Update"
- Slide 2: Revenue highlights (use attached revenue.xlsx for chart)
- Slide 3: Key metrics table
- Slide 4: Next quarter priorities (3 bullet points)
- Slide 5: Thank you with contact info

Preserve all template formatting, fonts, and colors."
```

### Template Design Tips

1. **Structure-first templates:** Design with fixed metric locations for stable replacements
2. **Use placeholder text:** `{{ TITLE }}`, `{{ SUBTITLE }}` for clear replacement targets
3. **Limit layout complexity:** Stick to 3-4 reusable layouts
4. **Consistent typography:** Define font hierarchy in slide master

### Quality Assurance

1. **Always generate thumbnails:** `python scripts/thumbnail.py output.pptx`
2. **Check for text cutoff:** Review thumbnail grid for overflow
3. **Verify numbers:** Human review required for financial data
4. **Test on target platform:** Open in PowerPoint, Keynote, and Google Slides

---

## 9. Code Examples

### Complete API Example (Python)

```python
import anthropic
import json

def create_presentation(topic: str, num_slides: int = 5) -> str:
    """Create a PowerPoint presentation using Claude's PPTX skill."""

    client = anthropic.Anthropic()

    response = client.beta.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=8192,
        betas=["code-execution-2025-08-25", "skills-2025-10-02"],
        container={
            "skills": [{
                "type": "anthropic",
                "skill_id": "pptx",
                "version": "latest"
            }]
        },
        messages=[{
            "role": "user",
            "content": f"""Create a professional {num_slides}-slide presentation about {topic}.

            Requirements:
            - Use a clean, modern design
            - Include data visualizations where appropriate
            - Add speaker notes for each slide
            - Use a consistent color scheme
            """
        }],
        tools=[{
            "type": "code_execution_20250825",
            "name": "code_execution"
        }]
    )

    # Extract and download the file
    file_id = None
    for block in response.content:
        if hasattr(block, 'file_id'):
            file_id = block.file_id
            break

    if file_id:
        file_content = client.beta.files.download(
            file_id=file_id,
            betas=["files-api-2025-04-14"]
        )
        output_path = f"{topic.replace(' ', '_')}.pptx"
        with open(output_path, "wb") as f:
            file_content.write_to_file(f.name)
        return output_path

    return None

# Usage
presentation_path = create_presentation("Sustainable Energy in K-12 Education")
print(f"Created: {presentation_path}")
```

### Template Update Example

```python
# Script to update a corporate template with new data

import anthropic

client = anthropic.Anthropic()

# Upload the template
template_upload = client.beta.files.create(
    file=open("corporate_template.pptx", "rb"),
    purpose="user_file",
    betas=["files-api-2025-04-14"]
)

# Upload data file
data_upload = client.beta.files.create(
    file=open("q4_data.xlsx", "rb"),
    purpose="user_file",
    betas=["files-api-2025-04-14"]
)

response = client.beta.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=8192,
    betas=["code-execution-2025-08-25", "skills-2025-10-02"],
    container={
        "skills": [{
            "type": "anthropic",
            "skill_id": "pptx",
            "version": "latest"
        }],
        "files": [
            {"type": "uploaded", "file_id": template_upload.id},
            {"type": "uploaded", "file_id": data_upload.id}
        ]
    },
    messages=[{
        "role": "user",
        "content": """Update corporate_template.pptx with data from q4_data.xlsx.

        - Keep all formatting, colors, and fonts from the template
        - Update the title slide with "Q4 2025 Results"
        - Replace chart data with values from the Excel file
        - Update all date references to Q4 2025
        """
    }],
    tools=[{
        "type": "code_execution_20250825",
        "name": "code_execution"
    }]
)
```

---

## 10. Resources

### Official Documentation
- [Agent Skills Quickstart](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart)
- [Using Skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [Create and Edit Files with Claude](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude)
- [Anthropic Skills Repository](https://github.com/anthropics/skills)

### Third-Party Tools
- [claude-office-skills](https://github.com/tfriedel/claude-office-skills) - CLI package for Claude Code
- [python-pptx Documentation](https://python-pptx.readthedocs.io/)
- [PptxGenJS Documentation](https://gitbrent.github.io/PptxGenJS/)

### Related Research
- [research-google-slides-api.md](research-google-slides-api.md) - Google Slides API capabilities
- [slides-interoperability.md](slides-interoperability.md) - Format conversion strategies
- [markdown-to-slides.md](markdown-to-slides.md) - Marp, Slidev, reveal.js comparison
- [python-pptx Documentation](https://python-pptx.readthedocs.io/) - Direct python-pptx usage patterns

---

## Appendix: Skill File Structure

```
/mnt/skills/public/pptx/
├── SKILL.md                 # Main workflow documentation
├── html2pptx.md             # HTML-to-PPTX conversion guide
├── ooxml.md                 # OOXML editing reference
├── html2pptx.tgz            # HTML conversion library
└── scripts/
    ├── thumbnail.py         # Generate thumbnail grids
    ├── rearrange.py         # Reorder/duplicate slides
    ├── inventory.py         # Extract text inventory as JSON
    └── replace.py           # Apply text replacements
```

---

*Last updated: 2026-01-15*
