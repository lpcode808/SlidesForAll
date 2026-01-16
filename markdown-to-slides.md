# Markdown to Slides

**Strategies for parsing Markdown content into slide structures**

**Last Updated**: 2026-01-15
**Related**: [[research-claude-pptx]] | [[research-google-slides-api]] | [[slides-interoperability]]

---

## Overview

Converting Markdown to slides requires two transformations:
1. **Parse** - Understand the Markdown structure
2. **Map** - Translate Markdown elements to slide concepts

This document explores parsing strategies, slide delimiters, and mapping conventions.

---

## Slide Delimiter Conventions

### Horizontal Rules (Most Common)

Used by: Marp, Slidev, reveal.js, Deckset

```markdown
# Slide 1 Title

Content for slide 1

---

# Slide 2 Title

Content for slide 2

---

# Slide 3 Title

Content for slide 3
```

**Pros**: Simple, familiar, minimal syntax
**Cons**: Can't use `---` for other purposes

### Level-1 Headers

Each `# Header` starts a new slide:

```markdown
# Introduction

Welcome to the presentation

# Problem Statement

Here's what we're solving

# Solution

Our approach
```

**Pros**: Natural mapping, clear hierarchy
**Cons**: Only one level of hierarchy visible in slide list

### Level-2 Headers with Sections

`#` = section, `##` = slide:

```markdown
# Section 1: Background

## Context

What you need to know

## History

How we got here

# Section 2: Proposal

## Approach

Our methodology
```

**Pros**: Supports grouping slides into sections
**Cons**: Requires consistent header usage

### Explicit Directives

Used by: Pandoc, some custom systems

```markdown
::: slide
# Title
Content here
:::

::: slide
# Another Title
More content
:::
```

**Pros**: Explicit, no ambiguity
**Cons**: Non-standard Markdown, verbose

---

## Markdown Elements → Slide Components

### Basic Mapping

| Markdown | Slide Component |
|----------|-----------------|
| `# H1` | Slide title |
| `## H2` | Subtitle or section header |
| `### H3` | Content heading |
| Paragraph | Body text |
| `- item` | Bullet point |
| `1. item` | Numbered list |
| `![](url)` | Image |
| `**bold**` | Bold text |
| `*italic*` | Italic text |
| `[text](url)` | Hyperlink |
| `> quote` | Callout/quote box |
| ``` code ``` | Code block |
| `| table |` | Table |

### Extended Mapping (Presenter Features)

| Markdown | Slide Component |
|----------|-----------------|
| `<!-- notes: ... -->` | Speaker notes |
| `<!-- .slide: bg=blue -->` | Slide background |
| `<!-- .element: class=fragment -->` | Animation trigger |

---

## Parsing Strategies

### Strategy 1: Simple Split

```javascript
function parseMarkdownToSlides(markdown) {
  const slides = markdown.split(/\n---\n/);

  return slides.map(slideContent => {
    const lines = slideContent.trim().split('\n');
    const titleMatch = lines[0].match(/^#\s+(.+)/);

    return {
      title: titleMatch ? titleMatch[1] : '',
      body: titleMatch ? lines.slice(1).join('\n').trim() : slideContent
    };
  });
}
```

**Best for**: Simple presentations, quick prototypes

### Strategy 2: AST-Based (Recommended)

Use a Markdown parser to build an AST, then walk it:

```javascript
import { unified } from 'unified';
import remarkParse from 'remark-parse';

function parseMarkdownAST(markdown) {
  const tree = unified()
    .use(remarkParse)
    .parse(markdown);

  const slides = [];
  let currentSlide = { title: '', elements: [] };

  for (const node of tree.children) {
    if (node.type === 'thematicBreak') {
      // --- delimiter = new slide
      if (currentSlide.elements.length > 0 || currentSlide.title) {
        slides.push(currentSlide);
      }
      currentSlide = { title: '', elements: [] };
    } else if (node.type === 'heading' && node.depth === 1 && !currentSlide.title) {
      currentSlide.title = extractText(node);
    } else {
      currentSlide.elements.push(transformNode(node));
    }
  }

  // Don't forget the last slide
  if (currentSlide.elements.length > 0 || currentSlide.title) {
    slides.push(currentSlide);
  }

  return slides;
}

function extractText(node) {
  if (node.type === 'text') return node.value;
  if (node.children) return node.children.map(extractText).join('');
  return '';
}

function transformNode(node) {
  switch (node.type) {
    case 'paragraph':
      return { type: 'text', content: extractText(node) };
    case 'list':
      return {
        type: node.ordered ? 'numberedList' : 'bulletList',
        items: node.children.map(li => extractText(li))
      };
    case 'image':
      return { type: 'image', url: node.url, alt: node.alt };
    // ... more transformations
    default:
      return { type: 'raw', node };
  }
}
```

**Best for**: Production systems, complex formatting needs

### Strategy 3: Hybrid (Marp-Style)

Let an existing tool do the heavy lifting:

```javascript
import { Marp } from '@marp-team/marp-core';

const marp = new Marp();
const { html, css } = marp.render(markdown);

// Parse the HTML output for slide structure
// Or use Marp's internal slide data
```

**Best for**: When you want battle-tested parsing

---

## Speaker Notes Conventions

### HTML Comment Style (Most Compatible)

```markdown
# Slide Title

Visible content

<!--
Speaker notes go here.
Multiple lines supported.
-->
```

### Marp Style

```markdown
# Slide Title

Visible content

---

<!-- This is a speaker note for the previous slide -->

# Next Slide
```

### Custom Delimiter

```markdown
# Slide Title

Visible content

???

Speaker notes after the triple question mark
```

### Frontmatter Per-Slide (Slidev Style)

```markdown
---
notes: |
  Speaker notes can go in YAML frontmatter
  for each slide.
---

# Slide Title

Visible content
```

---

## Image Handling

### Basic Images

```markdown
![Alt text](./image.png)
```

Maps to: Image element, positioned by layout rules

### Sized Images

```markdown
![Alt text](./image.png){width=300}
```

Or platform-specific:

```markdown
<!-- Marp: -->
![width:300px](./image.png)

<!-- Slidev: -->
<img src="./image.png" width="300" />
```

### Background Images

```markdown
<!-- Marp frontmatter: -->
---
backgroundImage: url('./bg.jpg')
---

<!-- Or inline directive: -->
![bg](./background.jpg)

# Title Over Background
```

---

## Code Blocks

### Basic Code

````markdown
```javascript
function hello() {
  console.log('Hello');
}
```
````

Maps to: Code shape with syntax highlighting (if supported)

### Code with Line Highlighting

````markdown
```javascript {2-3}
function hello() {
  console.log('Hello');  // highlighted
  console.log('World');  // highlighted
}
```
````

**Note**: Line highlighting typically requires custom rendering; most slide APIs don't support it natively.

---

## Recommended Content Model

A platform-agnostic intermediate representation:

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
  | ShapeElement
  | CodeElement;

interface TextElement {
  type: 'text';
  content: string;
  style?: TextStyle;
}

interface BulletListElement {
  type: 'bulletList';
  items: (string | BulletListElement)[];  // Nested lists
}

interface ImageElement {
  type: 'image';
  url: string;
  alt?: string;
  width?: number;
  height?: number;
}

interface TableElement {
  type: 'table';
  headers?: string[];
  rows: string[][];
}

interface CodeElement {
  type: 'code';
  language?: string;
  content: string;
  highlightLines?: number[];
}

interface ThemeConfig {
  primaryColor: string;
  secondaryColor: string;
  fontFamily: string;
  fontSize: number;
}
```

---

## Example: Full Pipeline

```javascript
// 1. Input: Markdown
const markdown = `
# Quarterly Review

---

# Revenue Highlights

- Q4 revenue: $12.4M
- YoY growth: 15%
- New customers: 847

![](./revenue-chart.png)

<!--
Key talking points:
- Strong finish to the year
- Beat projections by 8%
-->

---

# Next Steps

1. Expand into European market
2. Launch enterprise tier
3. Hire 20 engineers
`;

// 2. Parse to content model
const presentation = parseMarkdownToSlides(markdown);
// Result:
// {
//   slides: [
//     { title: 'Quarterly Review', elements: [], notes: '' },
//     {
//       title: 'Revenue Highlights',
//       elements: [
//         { type: 'bulletList', items: ['Q4 revenue: $12.4M', ...] },
//         { type: 'image', url: './revenue-chart.png' }
//       ],
//       notes: 'Key talking points:\n- Strong finish...'
//     },
//     { title: 'Next Steps', elements: [...], notes: '' }
//   ]
// }

// 3. Generate to target platform
if (target === 'google-slides') {
  generateGoogleSlides(presentation);
} else if (target === 'pptx') {
  generatePPTX(presentation);
}
```

---

## Existing Tools

### Marp

- **Input**: Markdown with `---` delimiters
- **Output**: HTML, PDF, PPTX
- **Pros**: Mature, good defaults, VS Code extension
- **Cons**: Limited layout customization

```bash
npx @marp-team/marp-cli slide.md --pptx
```

### Slidev

- **Input**: Markdown with YAML frontmatter
- **Output**: HTML (SPA), PDF
- **Pros**: Beautiful defaults, presenter mode, animations
- **Cons**: No direct PPTX export

### reveal.js

- **Input**: HTML or Markdown
- **Output**: HTML presentation
- **Pros**: Most customizable, plugins ecosystem
- **Cons**: Steeper learning curve

### Pandoc

- **Input**: Various formats including Markdown
- **Output**: PPTX, reveal.js, beamer, etc.
- **Pros**: Universal converter, scriptable
- **Cons**: Less polished slide output

```bash
pandoc slides.md -o slides.pptx
```

---

## Integration with SlidesForAll

### Recommended Approach

1. **Parse with remark** - Use `unified` + `remark-parse` for AST
2. **Custom delimiter handling** - Support `---` and `# heading` as slide breaks
3. **Speaker notes via comments** - `<!-- notes: ... -->` syntax
4. **Output to content model** - Platform-agnostic representation
5. **Generate via API** - Use [[research-google-slides-api]] or [[research-claude-pptx]]

### Sample Implementation Outline

```
src/
├── parser/
│   ├── markdown.ts      # Markdown → AST
│   ├── slides.ts        # AST → Slide content model
│   └── notes.ts         # Extract speaker notes
├── generators/
│   ├── google-slides.ts # Content model → Google Slides API calls
│   └── pptx.ts          # Content model → PPTX (via Claude or python-pptx)
└── index.ts             # CLI or API entry point
```

---

## Related Documents

- [[research-claude-pptx]] - Claude PPTX generation for output
- [[research-google-slides-api]] - Google Slides API for output
- [[slides-interoperability]] - What content survives format conversion

---

*Last updated: 2026-01-15*
