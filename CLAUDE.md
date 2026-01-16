# CLAUDE.md - SlidesForAll Project Context

**AI Assistant Instructions for SlidesForAll**

This file provides context for AI coding assistants (Claude Code, Cursor, GitHub Copilot, etc.) working on this project.

---

## Project Overview

**SlidesForAll** is a research and development project exploring programmatic slide generation from Markdown content with style encoding, targeting Google Slides as the primary platform.

### Goals

1. **Parse Markdown** → structured slide content
2. **Apply style encoding** via API parameters
3. **Generate Google Slides** presentations programmatically
4. **Support PPTX export** where interoperable

### Non-Goals (Current Phase)

- Building a full-featured presentation editor
- Supporting all PowerPoint features (animations, etc.)
- Creating a hosted web service

---

## Current State

### Phase: Research & Documentation

This project is in **research phase**. We have:

- [x] Comprehensive API research (Claude PPTX, Google Slides API)
- [x] Interoperability analysis (PPTX ↔ Google Slides)
- [x] Markdown parsing strategy documentation
- [ ] Content model implementation
- [ ] Parser implementation
- [ ] Generator implementation
- [ ] CLI tool
- [ ] Example templates

### File Structure

```
SlidesForAll/
├── CLAUDE.md                      # This file (AI assistant context)
├── agents.md                      # Multi-agent workflow guidance
├── README.md                      # Project index and quick reference
├── notes.md                       # Original project notes
│
├── research-claude-pptx.md        # Claude Code PPTX skill research (656 lines)
├── research-google-slides-api.md  # Google Apps Script API research (1518 lines)
├── slides-interoperability.md     # Format conversion compatibility (287 lines)
├── markdown-to-slides.md          # Parsing strategies (380 lines)
│
└── .claude/                       # Claude Code settings
    └── settings.local.json
```

---

## Technical Architecture (Planned)

### Content Model (Platform-Agnostic IR)

The core abstraction is a platform-agnostic intermediate representation:

```typescript
interface Presentation {
  metadata: {
    title: string;
    author?: string;
    date?: string;
  };
  theme?: ThemeConfig;
  slides: Slide[];
}

interface Slide {
  id: string;
  layout: 'title' | 'titleAndBody' | 'twoColumn' | 'blank' | 'sectionHeader';
  title?: string;
  subtitle?: string;
  elements: SlideElement[];
  notes?: string;
  background?: BackgroundConfig;
}

type SlideElement =
  | TextElement
  | BulletListElement
  | NumberedListElement
  | ImageElement
  | TableElement
  | CodeElement;
```

See `markdown-to-slides.md` for full type definitions.

### Pipeline Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Markdown   │ ──► │  Parse into  │ ──► │ Generate via:   │
│  + YAML     │     │  Content     │     │ • Google Slides │
│  metadata   │     │  Model       │     │ • Claude PPTX   │
└─────────────┘     └──────────────┘     │ • HTML (Marp)   │
                                         └─────────────────┘
```

**Three-stage pipeline:**

1. **Parser** - Markdown → Content Model
   - Use `unified` + `remark-parse` for AST
   - Support `---` and `# heading` as slide breaks
   - Extract speaker notes from `<!-- notes: ... -->` comments
   - Handle images, tables, lists, code blocks

2. **Content Model** - Platform-agnostic IR
   - Represents slides, elements, styling
   - No platform-specific concepts (no "batch requests", no "EMU units")
   - Easy to serialize/deserialize (JSON)

3. **Generators** - Content Model → Platform API calls
   - Google Slides: Use Apps Script batch updates
   - Claude PPTX: Use template-based or HTML-to-PPTX workflow
   - HTML: Render to Marp/reveal.js format

### Technology Stack (When Implementation Begins)

- **Language**: TypeScript (Node.js)
- **Markdown Parsing**: `unified` + `remark-parse`
- **Testing**: Vitest or Jest
- **CLI**: Commander.js
- **Google API**: `googleapis` npm package + Apps Script
- **Build**: Vite or tsup
- **Package Manager**: pnpm (per parent repo standard)

---

## Key Design Decisions

### 1. Google Slides Primary, PPTX Secondary

**Rationale**:
- Free tier, generous quotas
- Native collaboration features
- Target use case: K-12 education (Google Workspace common)
- PPTX export via Drive API for offline use

**Limitation Accepted**: No animations/transitions via API (must create manually in UI)

### 2. Lowest Common Denominator Content

**Rationale**: Content should work in both Google Slides and PowerPoint

**Constraints**:
- Web-safe fonts only (Arial, Helvetica, Times New Roman, Georgia, Verdana)
- Solid fill colors (no gradients)
- Standard shapes (rectangles, ellipses, lines)
- PNG/JPEG images < 25 megapixels
- No animations or transitions
- Speaker notes as plain text

See `slides-interoperability.md` for full compatibility matrix.

### 3. Markdown as Content Source

**Rationale**:
- Familiar format for technical users
- Version control friendly
- Separates content from presentation
- Supports speaker notes, metadata

**Delimiter**: Use `---` (horizontal rule) to separate slides (Marp convention)

### 4. Style Encoding via Theme Config

**Approach**: YAML frontmatter or separate config file

```yaml
theme:
  primaryColor: '#1a365d'
  secondaryColor: '#4a90c2'
  fontFamily: 'Arial'
  fontSize: 24
  layout: 'default'
```

---

## Development Guidelines

### Code Style

- **TypeScript**: Strict mode enabled
- **Naming**: camelCase for variables/functions, PascalCase for types/interfaces
- **Formatting**: Prettier with 2-space indent
- **Comments**: JSDoc for public APIs only

### File Organization (When Implementing)

```
src/
├── parser/
│   ├── markdown.ts      # Markdown → AST
│   ├── slides.ts        # AST → Slide content model
│   └── notes.ts         # Extract speaker notes
├── model/
│   └── types.ts         # TypeScript types for content model
├── generators/
│   ├── google-slides.ts # Content model → Google Slides API
│   ├── pptx.ts          # Content model → PPTX (via Claude)
│   └── html.ts          # Content model → Marp/reveal.js
├── cli/
│   └── index.ts         # CLI entry point
└── index.ts             # Library entry point
```

### Testing Approach

1. **Unit Tests**: Parser logic, content model transformations
2. **Integration Tests**: Full pipeline with fixture markdown files
3. **Manual Tests**: Generate actual presentations and open in Google Slides/PowerPoint

### Git Workflow

- **Commit early, commit often** - This project should be version controlled
- **Descriptive commits** - "Add bullet list parsing" not "updates"
- **Branch for experiments** - Use feature branches for new generators

---

## Working with This Project

### As an AI Assistant

When working on SlidesForAll:

1. **Read research docs first** - Understand API constraints before implementing
2. **Follow the content model** - Don't leak platform-specific details into IR
3. **Test interoperability** - Verify content works in both Google Slides and PowerPoint
4. **Ask about ambiguity** - Use AskUserQuestion for design decisions
5. **Document decisions** - Update this file when making architectural choices

### Common Tasks

#### Task: Implement Markdown Parser

1. Read `markdown-to-slides.md` for parsing strategy
2. Use `unified` + `remark-parse` for AST
3. Transform AST nodes to content model
4. Extract speaker notes from HTML comments
5. Write tests with fixture markdown files

#### Task: Implement Google Slides Generator

1. Read `research-google-slides-api.md` for API patterns
2. Transform content model to batch update requests
3. Use predefined layouts (TITLE, TITLE_AND_BODY, etc.)
4. Handle text styling, images, lists, tables
5. Set speaker notes
6. Return presentation URL

#### Task: Add New Slide Layout

1. Update content model types in `src/model/types.ts`
2. Add layout logic to parser (how to detect it from markdown)
3. Implement in each generator (Google Slides, PPTX, HTML)
4. Update documentation

### Reading Order for New Contributors

1. `README.md` - Project overview and quick reference
2. `markdown-to-slides.md` - Understand parsing approach
3. `research-google-slides-api.md` - Primary target platform
4. `slides-interoperability.md` - Constraints and compatibility
5. `research-claude-pptx.md` - Secondary target (PPTX export)
6. This file (`CLAUDE.md`) - Development guidelines

---

## API Rate Limits (Important)

### Google Slides API Quotas

| Quota Type | Limit |
|------------|-------|
| Write requests per minute (project) | 600 |
| Write requests per minute (per user) | 60 |
| Read requests per minute (project) | 3,000 |
| Read requests per minute (per user) | 600 |

**Implication**: Batch updates are essential. Never call API in loops.

### Claude Code PPTX Skill

- **File size limit**: 30 MB (upload + download combined)
- **Context window**: 200K tokens (500K for Enterprise)
- **Practical slide count**: ~50-100 slides

---

## Known Limitations

### Google Slides API

- **No animations API** - Open feature request, manually create in UI
- **No transitions API** - Same as animations
- **Fixed page size** - 16:9 only via API (create manually for custom sizes)
- **Shadows are read-only** - Cannot programmatically add shadows
- **Custom fonts limited** - Google Fonts only

### PPTX ↔ Google Slides Conversion

- **Animations lost** - Complex animations → basic fade or removed
- **Audio stripped** - Embedded audio files removed
- **Font substitution** - Custom fonts → Arial or similar
- **Charts unlinked** - Sheets charts → static images on export

See `slides-interoperability.md` for complete matrix.

---

## Educational Context

This project is part of a K-12 STEAM education portfolio. Design considerations:

- **Simplicity over features** - Teachers and students should understand the workflow
- **Accessibility** - Generated slides should be keyboard-navigable, screen-reader friendly
- **Performance** - Fast generation for classroom use (limited bandwidth)
- **Collaboration** - Google Slides native collaboration is a key feature

See `~/.claude/CLAUDE.md` for global educational technology principles.

---

## Next Steps (Implementation Roadmap)

When ready to move from research to implementation:

1. **Initialize git repository** - `git init`, add `.gitignore`
2. **Set up TypeScript project** - `package.json`, `tsconfig.json`
3. **Implement content model types** - `src/model/types.ts`
4. **Build markdown parser** - `src/parser/` with tests
5. **Implement Google Slides generator** - `src/generators/google-slides.ts`
6. **Create CLI tool** - `src/cli/index.ts`
7. **Test with real presentations** - Validate in Google Slides and PowerPoint
8. **Add PPTX generator** - Via Claude PPTX skill or python-pptx
9. **Document usage** - Update README with examples
10. **Publish** - npm package or standalone binary

---

## Questions to Ask User Before Implementation

- [ ] Target platforms: Google Slides only, or also PPTX export?
- [ ] CLI only, or also library API?
- [ ] Authentication strategy for Google Slides? (OAuth, service account, etc.)
- [ ] Default theme/styling approach? (YAML frontmatter, separate config file?)
- [ ] Image handling: local files only, or also URLs?
- [ ] Code block syntax highlighting: Yes or plain monospace?
- [ ] Should we support existing Marp/Slidev syntax, or define our own?

---

## Related Projects

- **Marp**: Markdown to slides (HTML, PDF, PPTX) - mature, good defaults
- **Slidev**: Vue-based slides from Markdown - beautiful, dev-focused
- **reveal.js**: HTML presentations - most customizable
- **Pandoc**: Universal document converter - includes slide output

**Differentiation**: SlidesForAll targets Google Slides API directly for native collaboration features, optimized for educational use.

---

## Contact & Contribution

This is a personal educational technology project by Justin Lai.

For questions or suggestions, update this file directly or add notes to `notes.md`.

---

*Last updated: 2026-01-15*
