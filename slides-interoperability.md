# Slides Interoperability

**Cross-format compatibility between PowerPoint (.pptx) and Google Slides**

**Last Updated**: 2026-01-15
**Related**: [research-claude-pptx.md](research-claude-pptx.md) | [research-google-slides-api.md](research-google-slides-api.md) | [markdown-to-slides.md](markdown-to-slides.md)

---

## Executive Summary

PPTX ↔ Google Slides conversion works well for **basic content** (text, images, shapes, tables) but has significant gaps for **advanced features** (animations, transitions, audio, custom fonts). For SlidesForAll, the safest path is to generate content that works in both formats by avoiding platform-specific features.

---

## Conversion Directions

### PPTX → Google Slides (Import)

When you upload a .pptx file to Google Drive and open with Google Slides:

| Feature | Conversion Quality | Notes |
|---------|-------------------|-------|
| **Text content** | Excellent | Preserves text, basic formatting |
| **Bold/italic/underline** | Excellent | Full preservation |
| **Font family** | Good | Substitutes non-Google fonts with similar alternatives |
| **Font size** | Excellent | Exact preservation |
| **Text colors** | Excellent | RGB colors preserved |
| **Shapes** | Good | Basic shapes convert well, complex shapes may simplify |
| **Images** | Excellent | PNG, JPEG, GIF all preserved |
| **Tables** | Good | Structure preserved, some cell formatting may vary |
| **Charts** | Fair | Static image conversion, no link to source data |
| **Hyperlinks** | Excellent | Internal and external links preserved |
| **Speaker notes** | Excellent | Text fully preserved |
| **Slide layouts** | Good | May map to closest Google layout |
| **Master slides** | Fair | Basic structure preserved, some theme elements lost |
| **Animations** | Poor | Most animations lost or converted to basic fade |
| **Transitions** | Poor | Often lost entirely |
| **Audio** | None | Audio files not supported |
| **Video (embedded)** | Poor | May need re-linking |
| **Video (YouTube)** | Good | Link preserved if recognizable |
| **SmartArt** | Fair | Converted to grouped shapes |
| **3D effects** | None | Flattened or removed |
| **Custom fonts** | Fair | Replaced with similar Google Fonts |
| **Macros/VBA** | None | Stripped entirely |

### Google Slides → PPTX (Export)

When you export a Google Slides presentation to .pptx:

| Feature | Conversion Quality | Notes |
|---------|-------------------|-------|
| **Text content** | Excellent | Full preservation |
| **Text formatting** | Excellent | Bold, italic, underline, strikethrough preserved |
| **Font family** | Good | Google Fonts → closest system font |
| **Shapes** | Good | Basic shapes convert cleanly |
| **Images** | Excellent | Full quality preserved |
| **Tables** | Good | Structure and basic formatting preserved |
| **Linked Sheets charts** | Fair | Converted to static image |
| **Hyperlinks** | Excellent | Preserved |
| **Speaker notes** | Excellent | Preserved |
| **Theme colors** | Fair | May shift slightly |
| **Layouts** | Good | Maps to PPTX layouts |
| **Animations** | N/A | Google Slides doesn't support via API |
| **Transitions** | N/A | Google Slides doesn't support via API |
| **Comments** | Good | Usually preserved |
| **Revision history** | None | Not exportable |

---

## Safe Content (Works in Both)

Content that survives round-trip conversion well:

### Text
- Plain text with basic formatting (bold, italic, underline)
- Standard fonts: Arial, Helvetica, Times New Roman, Georgia, Verdana
- Font sizes (points)
- Text colors (RGB hex)
- Bullet points (standard disc/circle/square)
- Numbered lists (decimal, alphabetic)
- Hyperlinks

### Visual Elements
- Rectangle, ellipse, rounded rectangle shapes
- Lines and arrows
- Images (PNG, JPEG preferred; < 25 megapixels)
- Simple tables (avoid merged cells if possible)
- Solid fill colors
- Basic borders/outlines

### Layout
- Title + body layouts
- Two-column layouts
- Blank slides with positioned elements
- Speaker notes (text only)

---

## Risky Content (May Not Convert Well)

### Avoid When Targeting Both Platforms

| Feature | Risk Level | Alternative |
|---------|------------|-------------|
| **Animations** | High | Use static layouts, emphasize content |
| **Transitions** | High | Accept simple cuts between slides |
| **Custom fonts** | Medium | Stick to web-safe fonts |
| **Embedded audio** | High | Link to external audio |
| **Complex SmartArt** | Medium | Use simple shapes instead |
| **Gradient fills** | Medium | Use solid colors or pre-rendered images |
| **Shadows** | Medium | May render differently |
| **3D effects** | High | Avoid entirely |
| **Merged table cells** | Medium | Test carefully |
| **Linked charts** | Medium | Accept static snapshots |
| **Theme-dependent colors** | Medium | Use explicit RGB values |

---

## Conversion Strategies

### Strategy 1: Lowest Common Denominator

Design for features that work in both platforms:

```
Content Checklist:
[x] Web-safe fonts only (Arial, Helvetica, Times New Roman)
[x] Solid fill colors (no gradients)
[x] Standard shapes (rectangles, ellipses, lines)
[x] PNG/JPEG images under 25 megapixels
[x] Simple bullet lists
[x] No animations or transitions
[x] Speaker notes as plain text
```

**Best for**: Educational content, reports, templates meant to work anywhere

### Strategy 2: Platform-Specific Generation

Generate different outputs for different targets:

```
┌────────────────┐
│ Content Model  │
└───────┬────────┘
        │
   ┌────┴────┐
   ▼         ▼
┌──────┐  ┌──────────────┐
│ PPTX │  │ Google Slides │
│      │  │              │
│ +anim│  │ +collab      │
│ +trans│ │ +live charts │
└──────┘  └──────────────┘
```

**Best for**: When you need platform-specific features

### Strategy 3: Primary + Export

Choose a primary platform and export as fallback:

**Option A: Google Slides Primary**
- Create in Google Slides
- Export PPTX for offline use
- Accept: no animations, linked charts become static

**Option B: PPTX Primary (via Claude)**
- Generate PPTX with full features
- Import to Google Slides for collaboration
- Accept: animations lost, fonts substituted

---

## Testing Protocol

### Quick Conversion Test

1. Create test presentation with:
   - Title slide
   - Text with various formatting
   - Bulleted list
   - Table (3x3)
   - Image
   - Shape with fill and border
   - Speaker notes

2. Convert in both directions:
   - Upload PPTX → Open in Google Slides → Check fidelity
   - Export Google Slides → Open in PowerPoint → Check fidelity

3. Document any discrepancies

### Automated Validation

```javascript
// Google Apps Script - compare original to round-tripped version
function validateConversion(originalId, importedId) {
  const original = SlidesApp.openById(originalId);
  const imported = SlidesApp.openById(importedId);

  const report = {
    slideCount: {
      original: original.getSlides().length,
      imported: imported.getSlides().length,
      match: original.getSlides().length === imported.getSlides().length
    },
    // Add more checks...
  };

  return report;
}
```

---

## Known Issues by Platform

### PowerPoint → Google Slides Issues

1. **Font substitution**: Custom fonts → Arial or similar
2. **Animation loss**: Complex animations → simple fade or removed
3. **Audio stripping**: Embedded audio files are removed
4. **SmartArt decomposition**: Converted to grouped shapes
5. **Equation rendering**: May convert to images

### Google Slides → PowerPoint Issues

1. **Chart unlinking**: Sheets charts become static images
2. **Font mapping**: Google Fonts → closest system font
3. **Theme drift**: Colors may shift slightly
4. **Web video**: YouTube links may need re-verification

---

## Recommendations for SlidesForAll

### Content Generation Guidelines

1. **Use explicit styling** - Don't rely on themes; specify RGB colors, font names, sizes
2. **Test fonts early** - Verify your font choices work in both platforms
3. **Images over effects** - For complex visuals, pre-render as PNG rather than using native effects
4. **Document limitations** - Tell users what won't convert when exporting

### Architecture Suggestion

```
┌─────────────────────────────────────────────────────────┐
│                   Content Model                         │
│  (Platform-agnostic slide representation)               │
├─────────────────────────────────────────────────────────┤
│  - slides: [{title, body, images, shapes, notes}]       │
│  - styles: {fonts, colors} (web-safe values)            │
│  - metadata: {title, author}                            │
└───────────────────────┬─────────────────────────────────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │  Google  │  │  PPTX    │  │  HTML    │
    │  Slides  │  │  (Claude)│  │  (Marp)  │
    └──────────┘  └──────────┘  └──────────┘
```

### Feature Flags for Output

```json
{
  "targetPlatform": "google-slides",
  "features": {
    "animations": false,
    "transitions": false,
    "linkedCharts": true,
    "speakerNotes": true
  },
  "fonts": {
    "primary": "Arial",
    "secondary": "Georgia"
  }
}
```

---

## Related Documents

- [research-claude-pptx.md](research-claude-pptx.md) - Claude's PPTX generation capabilities
- [research-google-slides-api.md](research-google-slides-api.md) - Google Slides API deep dive
- [markdown-to-slides.md](markdown-to-slides.md) - Parsing markdown into slide content

---

*Last updated: 2026-01-15*
