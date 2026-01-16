# SlidesForAll

**Research hub for programmatic slide generation** - comparing Claude Code's PPTX skill with Google Apps Script Slides API.

## Goal

Take content (Markdown) + style encoding and generate slides programmatically, with Google Slides as the primary target.

---

## Project Documentation

| Document | Description |
|----------|-------------|
| [[CLAUDE.md]] | **AI Assistant Context** - Architecture, decisions, development guidelines |
| [[agents.md]] | **Multi-Agent Workflows** - Coordination guide for AI assistants |

## Research Documents

| Document | Description |
|----------|-------------|
| [[research-claude-pptx]] | Claude Code's official PowerPoint skill - capabilities, workflows, limitations |
| [[research-google-slides-api]] | Google Apps Script Slides API - full capabilities, code examples, best practices |
| [[slides-interoperability]] | Cross-format compatibility - what survives PPTX ↔ Google Slides conversion |
| [[markdown-to-slides]] | Strategies for parsing Markdown into slide structures |

---

## Quick Comparison

| Feature | Claude PPTX | Google Slides API |
|---------|-------------|-------------------|
| **Animations** | Via OOXML (manual) | Not supported |
| **Transitions** | Via OOXML (manual) | Not supported |
| **Custom fonts** | Web-safe only | Google Fonts only |
| **Page size** | Any dimension | Fixed 16:9 |
| **Real-time collab** | No | Native |
| **Speaker notes** | Full | Full |
| **Charts** | Via PptxGenJS | Via Sheets integration |
| **Cost** | API credits | Free |
| **Offline** | Full | Requires connectivity |

---

## Recommended Workflow

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Markdown   │ ──► │  Parse into  │ ──► │ Generate via:   │
│  Content    │     │  Slide Data  │     │ • Google Slides │
└─────────────┘     └──────────────┘     │ • Claude PPTX   │
                                         └─────────────────┘
```

### When to Use Which

| Use Case | Recommended |
|----------|-------------|
| K-12 classroom, collaboration needed | Google Slides API |
| Animation-heavy presentations | Claude PPTX (OOXML workflow) |
| Template-based reports | Either (both excel here) |
| Custom page sizes (A4, 4:3, etc.) | Claude PPTX |
| Offline-first workflows | Claude PPTX |
| Live data dashboards | Google Slides + Sheets |

---

## Key Findings

### Claude PPTX Strengths
- Three workflow options: HTML-to-PPTX, template-based, direct OOXML
- Full animation/transition support via OOXML editing
- Any page dimension supported
- Works offline

### Google Slides API Strengths
- Free tier with generous quotas (600 writes/min)
- Native collaboration and version history
- Excellent Sheets chart integration
- Template mail-merge is very clean (`replaceAllText`, `replaceAllShapesWithImage`)

### Critical Limitations
- **Google Slides**: No animation/transition API (open feature request)
- **Claude PPTX**: 30MB file limit, web-safe fonts only
- **Both**: Complex styling requires careful handling during conversion

---

## File Structure

```
SlidesForAll/
├── README.md                      # This file (project index)
├── CLAUDE.md                      # AI assistant context & guidelines
├── agents.md                      # Multi-agent workflow guide
├── notes.md                       # Original project notes
├── .gitignore                     # Git ignore rules
│
├── research-claude-pptx.md        # Claude PPTX skill research (656 lines)
├── research-google-slides-api.md  # Google Apps Script research (1518 lines)
├── slides-interoperability.md     # Format conversion details (287 lines)
└── markdown-to-slides.md          # Markdown parsing strategies (380 lines)
```

**For AI assistants**: Read `CLAUDE.md` first for project context and architecture.

---

## Next Steps

1. **Define content model** - What structure represents a "slide" from Markdown?
2. **Build prototype** - Start with Google Slides API (primary target)
3. **Test interoperability** - Verify PPTX → Google Slides conversion quality
4. **Add Claude PPTX path** - For animation/custom-size use cases

---

*Last updated: 2026-01-15*
