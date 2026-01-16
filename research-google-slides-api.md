# Google Slides API Research Document

This document provides comprehensive research on the Google Slides API accessed via Google Apps Script, including capabilities, limitations, code patterns, and integration considerations for the SlidesForAll project.

**Last Updated**: 2026-01-15
**Research Status**: Complete
**Related Documents**: [research-claude-pptx.md](research-claude-pptx.md), [slides-interoperability.md](slides-interoperability.md), [markdown-to-slides.md](markdown-to-slides.md)

---

## Executive Summary

The Google Slides API via Apps Script provides a robust platform for programmatic presentation creation with:

- **Free tier** with generous quotas (600 writes/min per project, 3000 reads/min)
- **Two access methods**: Built-in Slides Service (simpler) and Advanced Slides Service (more powerful)
- **Strong template support** via mail-merge style text/image replacement
- **Full text styling control** but **no animation/transition API support**
- **PPTX export** via Drive API (with some feature loss)
- **Sheets chart integration** with live linking

**Key Limitation**: Animations and transitions cannot be created programmatically - this is a known feature request on Google's issue tracker.

---

## Table of Contents

1. [Overview](#overview)
2. [Core Capabilities](#core-capabilities)
3. [Two API Access Methods](#two-api-access-methods)
4. [Creating Slides Programmatically](#creating-slides-programmatically)
5. [Styling and Formatting](#styling-and-formatting)
6. [Working with Templates](#working-with-templates)
7. [Images and Media](#images-and-media)
8. [Tables and Charts](#tables-and-charts)
9. [Speaker Notes](#speaker-notes)
10. [Import/Export Formats](#importexport-formats)
11. [Authentication and Authorization](#authentication-and-authorization)
12. [Limitations and Constraints](#limitations-and-constraints)
13. [Best Practices](#best-practices)
14. [Code Examples](#code-examples)
15. [Units and Coordinate Systems](#units-and-coordinate-systems)
16. [Page Elements Deep Dive](#page-elements-deep-dive)
17. [Resources](#resources)

---

## Overview

The Google Slides API enables programmatic creation, reading, and editing of Google Slides presentations. It's particularly useful for:

- **Automated report generation** from data sources
- **Template-based document merge** workflows
- **Batch slide creation** from structured content
- **Integration with other Google Workspace** services (Sheets, Drive, Docs)

The API is free to use with generous quotas, making it suitable for educational tools and automation workflows.

### Key Characteristics

| Feature | Details |
|---------|---------|
| **Cost** | Free (no additional charges) |
| **Access Methods** | Built-in Slides Service, Advanced Slides Service |
| **Primary Pattern** | Batch updates via JSON requests |
| **Output Format** | Native Google Slides, exportable to PPTX/PDF |

---

## Core Capabilities

### What You CAN Do

1. **Presentation Management**
   - Create new presentations
   - Open existing presentations by ID or URL
   - Copy/duplicate presentations
   - Delete presentations (via Drive API)

2. **Slide Operations**
   - Create slides with predefined layouts
   - Duplicate, move, delete slides
   - Access slide content and properties
   - Set background colors/images

3. **Content Creation**
   - Insert shapes (rectangles, ellipses, text boxes, etc.)
   - Insert images from URLs or Drive
   - Insert tables with configurable rows/columns
   - Insert videos (YouTube, Drive)
   - Insert charts from Google Sheets
   - Insert lines and connectors
   - Insert Word Art

4. **Text Operations**
   - Insert, delete, replace text
   - Format text (bold, italic, underline, strikethrough)
   - Set font family, size, color
   - Create bulleted/numbered lists
   - Add hyperlinks
   - Find and replace across entire presentation

5. **Layout and Positioning**
   - Set element position (x, y coordinates)
   - Set element size (width, height)
   - Rotate elements
   - Apply transforms
   - Align elements
   - Z-order management (bring to front, send to back)

6. **Styling**
   - Fill colors (solid, pictures)
   - Outline/border properties
   - Text styles (per-character formatting)
   - Paragraph styles
   - Theme colors (limited editing)

7. **Speaker Notes**
   - Read speaker notes
   - Write/modify speaker notes text

8. **Charts Integration**
   - Embed charts from Google Sheets
   - Link charts for live updates
   - Refresh linked charts

---

## Two API Access Methods

Google Apps Script provides two ways to interact with Google Slides:

### 1. Built-in Slides Service (Recommended)

The native `SlidesApp` service provides a high-level, object-oriented interface.

```javascript
// Open a presentation
const presentation = SlidesApp.openById('PRESENTATION_ID');

// Get slides
const slides = presentation.getSlides();

// Create a new slide
const slide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY);

// Insert a shape
const shape = slide.insertShape(SlidesApp.ShapeType.RECTANGLE, 100, 100, 200, 100);
shape.getText().setText('Hello World');
```

**Key Classes:**
- `SlidesApp` - Entry point for creating/opening presentations
- `Presentation` - Top-level container
- `Slide` - Individual slide page
- `Shape` - Generic visual element
- `TextRange` - Text manipulation
- `Image`, `Video`, `Table`, `Line`, `Group`, `SheetsChart`

### 2. Advanced Slides Service

The Advanced Slides service provides direct access to the REST API, enabling more advanced operations via batch updates.

**Enabling the Service:**
1. Open your Apps Script project
2. Go to Services (+)
3. Add "Google Slides API"

```javascript
// Create a presentation using Advanced Service
function createPresentation() {
  const presentation = Slides.Presentations.create({
    title: 'My New Presentation'
  });
  return presentation.presentationId;
}
```

**When to Use Advanced Service:**
- Batch updates for better performance
- Operations not available in built-in service
- Direct API control needed
- Complex multi-step operations

---

## Creating Slides Programmatically

### Predefined Layouts

Google Slides provides 11 predefined layout types:

| Layout | Description |
|--------|-------------|
| `BLANK` | Empty slide, no placeholders |
| `TITLE` | Title and subtitle |
| `TITLE_AND_BODY` | Title with body content area |
| `TITLE_AND_TWO_COLUMNS` | Title with two-column body |
| `TITLE_ONLY` | Only a title placeholder |
| `SECTION_HEADER` | Section divider slide |
| `SECTION_TITLE_AND_DESCRIPTION` | Title/subtitle on one side, description on other |
| `ONE_COLUMN_TEXT` | Single column text layout |
| `MAIN_POINT` | Emphasizes a main point |
| `BIG_NUMBER` | Large number heading |
| `CAPTION_ONLY` | Caption at bottom |

**Note:** These layouts may not exist in all presentations if they've been deleted from the master.

### Creating Slides with Built-in Service

```javascript
function createSlidesExample() {
  const presentation = SlidesApp.create('My Presentation');

  // Append slides with different layouts
  const titleSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE);
  const contentSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY);
  const blankSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.BLANK);

  // Access placeholders on title slide
  const shapes = titleSlide.getShapes();
  shapes[0].getText().setText('Presentation Title');
  shapes[1].getText().setText('Subtitle goes here');

  return presentation.getId();
}
```

### Creating Slides with Advanced Service (Batch Update)

```javascript
function createSlideAdvanced(presentationId) {
  const pageId = Utilities.getUuid();

  const requests = [{
    createSlide: {
      objectId: pageId,
      insertionIndex: 1,
      slideLayoutReference: {
        predefinedLayout: 'TITLE_AND_TWO_COLUMNS'
      }
    }
  }];

  const response = Slides.Presentations.batchUpdate(
    { requests: requests },
    presentationId
  );

  return response.replies[0].createSlide.objectId;
}
```

### Object ID Best Practices

- Use UUIDs for unique IDs: `Utilities.getUuid()`
- IDs must be unique across all pages and elements
- Must start with `[a-zA-Z0-9_]`
- Can contain `[a-zA-Z0-9_-:]`
- Length: 5-50 characters
- Or omit to let the API generate IDs

---

## Styling and Formatting

### Text Styling

Text can be styled at the character level with the following properties:

| Property | Description |
|----------|-------------|
| `bold` | Bold text |
| `italic` | Italic text |
| `underline` | Underlined text |
| `strikethrough` | Strikethrough text |
| `fontFamily` | Font name (e.g., "Arial", "Times New Roman") |
| `fontSize` | Size in points |
| `foregroundColor` | Text color (RGB or theme color) |
| `backgroundColor` | Text highlight color |
| `link` | Hyperlink URL |
| `smallCaps` | Small capitals |
| `baselineOffset` | Superscript/subscript |

### Styling with Built-in Service

```javascript
function styleTextExample() {
  const presentation = SlidesApp.openById('PRESENTATION_ID');
  const slide = presentation.getSlides()[0];
  const shape = slide.getShapes()[0];
  const textRange = shape.getText();

  // Style all text
  textRange.getTextStyle()
    .setBold(true)
    .setFontSize(24)
    .setForegroundColor('#FF5733');

  // Style specific range (characters 0-4)
  textRange.getRange(0, 5).getTextStyle()
    .setItalic(true)
    .setUnderline(true);
}
```

### Styling with Advanced Service (Batch Update)

```javascript
function styleTextAdvanced(presentationId, shapeId) {
  const requests = [
    // Make characters 0-4 bold and italic
    {
      updateTextStyle: {
        objectId: shapeId,
        textRange: {
          type: 'FIXED_RANGE',
          startIndex: 0,
          endIndex: 5
        },
        style: {
          bold: true,
          italic: true
        },
        fields: 'bold,italic'
      }
    },
    // Set font and color for characters 5-9
    {
      updateTextStyle: {
        objectId: shapeId,
        textRange: {
          type: 'FIXED_RANGE',
          startIndex: 5,
          endIndex: 10
        },
        style: {
          foregroundColor: {
            opaqueColor: {
              rgbColor: {
                red: 0.0,
                green: 0.0,
                blue: 1.0
              }
            }
          },
          fontSize: {
            magnitude: 14,
            unit: 'PT'
          },
          fontFamily: 'Times New Roman'
        },
        fields: 'foregroundColor,fontSize,fontFamily'
      }
    }
  ];

  return Slides.Presentations.batchUpdate({ requests }, presentationId);
}
```

### Creating Bullet Points

```javascript
function createBullets(presentationId, shapeId) {
  const requests = [{
    createParagraphBullets: {
      objectId: shapeId,
      bulletPreset: 'BULLET_ARROW_DIAMOND_DISC'
    }
  }];

  return Slides.Presentations.batchUpdate({ requests }, presentationId);
}
```

**Available Bullet Presets:**
- `BULLET_DISC_CIRCLE_SQUARE`
- `BULLET_DIAMONDX_ARROW3D_SQUARE`
- `BULLET_CHECKBOX`
- `BULLET_ARROW_DIAMOND_DISC`
- `BULLET_STAR_CIRCLE_SQUARE`
- `BULLET_ARROW3D_CIRCLE_SQUARE`
- `BULLET_LEFTTRIANGLE_DIAMOND_DISC`
- `NUMBERED_DECIMAL_ALPHA_ROMAN`
- `NUMBERED_DECIMAL_ALPHA_ROMAN_PARENS`
- `NUMBERED_DECIMAL_NESTED`
- `NUMBERED_UPPERALPHA_ALPHA_ROMAN`
- `NUMBERED_UPPERROMAN_UPPERALPHA_DECIMAL`
- `NUMBERED_ZERODIGIT_ALPHA_ROMAN`

### Shape Styling

```javascript
function styleShape() {
  const presentation = SlidesApp.openById('PRESENTATION_ID');
  const slide = presentation.getSlides()[0];
  const shape = slide.insertShape(SlidesApp.ShapeType.RECTANGLE, 100, 100, 200, 100);

  // Fill color
  shape.getFill().setSolidFill('#3498db');

  // Border
  shape.getBorder()
    .setWeight(2)
    .getLineFill().setSolidFill('#2c3e50');

  // Content alignment
  shape.setContentAlignment(SlidesApp.ContentAlignment.MIDDLE);
}
```

---

## Working with Templates

Template-based workflows are a primary use case for the Google Slides API. The typical pattern is:

1. Create a template presentation with placeholder text
2. Copy the template
3. Replace placeholders with actual data
4. Optionally export or share

### Template Copy Pattern

```javascript
function createFromTemplate(templateId, newTitle) {
  // Copy the template using Drive API
  const copyFile = Drive.Files.copy(
    { title: newTitle },
    templateId
  );

  return copyFile.id;
}
```

### Text Merge (Mail Merge Pattern)

```javascript
function textMerge(presentationId, replacements) {
  // replacements = { '{{name}}': 'John Doe', '{{date}}': '2026-01-15' }

  const requests = Object.entries(replacements).map(([placeholder, value]) => ({
    replaceAllText: {
      containsText: {
        text: placeholder,
        matchCase: true
      },
      replaceText: value
    }
  }));

  return Slides.Presentations.batchUpdate({ requests }, presentationId);
}
```

### Image Merge Pattern

Replace placeholder shapes with images:

```javascript
function imageMerge(presentationId, imageReplacements) {
  // imageReplacements = { '{{logo}}': 'https://example.com/logo.png' }

  const requests = Object.entries(imageReplacements).map(([placeholder, imageUrl]) => ({
    replaceAllShapesWithImage: {
      imageUrl: imageUrl,
      imageReplaceMethod: 'CENTER_INSIDE', // or 'CENTER_CROP'
      containsText: {
        text: placeholder,
        matchCase: true
      }
    }
  }));

  return Slides.Presentations.batchUpdate({ requests }, presentationId);
}
```

### Complete Template Workflow Example

```javascript
function generateReportFromTemplate(templateId, data) {
  // 1. Copy template
  const newPresentationId = createFromTemplate(templateId, 'Report - ' + data.title);

  // 2. Replace text placeholders
  textMerge(newPresentationId, {
    '{{title}}': data.title,
    '{{date}}': data.date,
    '{{author}}': data.author,
    '{{summary}}': data.summary
  });

  // 3. Replace image placeholders
  imageMerge(newPresentationId, {
    '{{company_logo}}': data.logoUrl,
    '{{chart_image}}': data.chartUrl
  });

  // 4. Return the new presentation URL
  return 'https://docs.google.com/presentation/d/' + newPresentationId;
}
```

---

## Images and Media

### Image Constraints

| Constraint | Limit |
|------------|-------|
| Maximum file size | 50 MB |
| Maximum resolution | 25 megapixels |
| Supported formats | PNG, JPEG, GIF |
| URL length limit | 2 KB |
| URL requirement | Publicly accessible |

### Inserting Images

**Built-in Service:**
```javascript
function insertImage(slideId, imageUrl) {
  const presentation = SlidesApp.openById('PRESENTATION_ID');
  const slide = presentation.getSlideById(slideId);

  // Insert from URL
  const image = slide.insertImage(imageUrl);

  // Position and size
  image.setLeft(100);
  image.setTop(100);
  image.setWidth(300);
  image.setHeight(200);
}
```

**Advanced Service:**
```javascript
function insertImageAdvanced(presentationId, pageId, imageUrl) {
  const requests = [{
    createImage: {
      objectId: 'MyImage_' + Utilities.getUuid(),
      url: imageUrl,
      elementProperties: {
        pageObjectId: pageId,
        size: {
          height: { magnitude: 3000000, unit: 'EMU' },
          width: { magnitude: 4000000, unit: 'EMU' }
        },
        transform: {
          scaleX: 1,
          scaleY: 1,
          translateX: 1000000,
          translateY: 1000000,
          unit: 'EMU'
        }
      }
    }
  }];

  return Slides.Presentations.batchUpdate({ requests }, presentationId);
}
```

### Inserting Videos

```javascript
function insertVideo() {
  const presentation = SlidesApp.openById('PRESENTATION_ID');
  const slide = presentation.getSlides()[0];

  // Insert YouTube video
  const video = slide.insertVideo(
    'https://www.youtube.com/watch?v=VIDEO_ID',
    100, 100, 400, 300
  );
}
```

**Video Sources:**
- YouTube URLs
- Google Drive video files

---

## Tables and Charts

### Creating Tables

```javascript
function createTable() {
  const presentation = SlidesApp.openById('PRESENTATION_ID');
  const slide = presentation.getSlides()[0];

  // Create 3x4 table (3 rows, 4 columns)
  const table = slide.insertTable(3, 4);

  // Populate cells
  table.getCell(0, 0).getText().setText('Header 1');
  table.getCell(0, 1).getText().setText('Header 2');
  table.getCell(1, 0).getText().setText('Data 1');
  table.getCell(1, 1).getText().setText('Data 2');
}
```

### Table Operations via Advanced Service

```javascript
function tableOperations(presentationId, tableId) {
  const requests = [
    // Insert row
    {
      insertTableRows: {
        tableObjectId: tableId,
        cellLocation: { rowIndex: 1 },
        insertBelow: true,
        number: 1
      }
    },
    // Insert column
    {
      insertTableColumns: {
        tableObjectId: tableId,
        cellLocation: { columnIndex: 1 },
        insertRight: true,
        number: 1
      }
    },
    // Merge cells
    {
      mergeTableCells: {
        objectId: tableId,
        tableRange: {
          location: { rowIndex: 0, columnIndex: 0 },
          rowSpan: 1,
          columnSpan: 2
        }
      }
    }
  ];

  return Slides.Presentations.batchUpdate({ requests }, presentationId);
}
```

### Embedding Charts from Google Sheets

Charts must first exist in a Google Sheet, then can be embedded:

```javascript
function embedChart(presentationId, pageId, spreadsheetId, chartId) {
  const requests = [{
    createSheetsChart: {
      objectId: 'MyChart_' + Utilities.getUuid(),
      spreadsheetId: spreadsheetId,
      chartId: chartId,
      linkingMode: 'LINKED', // or 'NOT_LINKED_IMAGE'
      elementProperties: {
        pageObjectId: pageId,
        size: {
          height: { magnitude: 4000000, unit: 'EMU' },
          width: { magnitude: 4000000, unit: 'EMU' }
        },
        transform: {
          scaleX: 1,
          scaleY: 1,
          translateX: 100000,
          translateY: 100000,
          unit: 'EMU'
        }
      }
    }
  }];

  return Slides.Presentations.batchUpdate({ requests }, presentationId);
}
```

### Chart Linking Modes

| Mode | Use Case |
|------|----------|
| `LINKED` | Chart updates when source data changes, collaborators see link |
| `NOT_LINKED_IMAGE` | Static snapshot, no connection to source data |

### Refreshing Linked Charts

```javascript
function refreshChart(presentationId, chartObjectId) {
  const requests = [{
    refreshSheetsChart: {
      objectId: chartObjectId
    }
  }];

  return Slides.Presentations.batchUpdate({ requests }, presentationId);
}
```

---

## Speaker Notes

### Reading Speaker Notes

```javascript
function getSpeakerNotes(presentationId) {
  const presentation = SlidesApp.openById(presentationId);
  const slides = presentation.getSlides();

  const notes = slides.map((slide, index) => {
    const notesPage = slide.getNotesPage();
    const notesShape = notesPage.getSpeakerNotesShape();
    const notesText = notesShape.getText().asString();

    return {
      slideIndex: index,
      notes: notesText.trim()
    };
  });

  return notes;
}
```

### Writing Speaker Notes

```javascript
function setSpeakerNotes(presentationId, slideIndex, notesText) {
  const presentation = SlidesApp.openById(presentationId);
  const slide = presentation.getSlides()[slideIndex];
  const notesPage = slide.getNotesPage();
  const notesShape = notesPage.getSpeakerNotesShape();

  // Clear existing notes and set new text
  notesShape.getText().setText(notesText);
}
```

**Note:** Only the text content of speaker notes is editable. Other properties of the notes page are read-only.

---

## Import/Export Formats

### Exporting to PowerPoint (PPTX)

The Google Slides API does not directly support export. Use the Drive API or URL-based export:

```javascript
function exportToPptx(presentationId, folderId) {
  const url = 'https://docs.google.com/presentation/d/' + presentationId + '/export/pptx';

  const params = {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer ' + ScriptApp.getOAuthToken()
    }
  };

  const response = UrlFetchApp.fetch(url, params);
  const blob = response.getBlob();

  const folder = DriveApp.getFolderById(folderId);
  const file = folder.createFile(blob.setName('presentation.pptx'));

  return file.getUrl();
}
```

### Exporting to PDF

```javascript
function exportToPdf(presentationId, folderId) {
  const url = 'https://docs.google.com/presentation/d/' + presentationId + '/export/pdf';

  const params = {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer ' + ScriptApp.getOAuthToken()
    }
  };

  const response = UrlFetchApp.fetch(url, params);
  const blob = response.getBlob();

  const folder = DriveApp.getFolderById(folderId);
  const file = folder.createFile(blob.setName('presentation.pdf'));

  return file.getUrl();
}
```

### Importing PowerPoint Files

PowerPoint files can be imported via Google Drive:

1. **Upload to Drive** - Use Drive API to upload PPTX file
2. **Convert on Upload** - Set `convert: true` in upload options
3. **Or Open and Save As** - Open PPTX in Slides, save as Google Slides

```javascript
function importPptx(pptxBlob, folderId) {
  const folder = DriveApp.getFolderById(folderId);

  // Upload and convert
  const file = folder.createFile(pptxBlob);

  // Convert to Google Slides
  const convertedFile = Drive.Files.copy(
    {
      title: file.getName().replace('.pptx', ''),
      mimeType: 'application/vnd.google-apps.presentation'
    },
    file.getId()
  );

  // Optionally delete the original PPTX
  file.setTrashed(true);

  return convertedFile.id;
}
```

### Import Limitations

When converting from PPTX to Google Slides, some features may not transfer:

- Complex animations and transitions
- Certain fonts (replaced with similar alternatives)
- Some embedded objects
- Advanced SmartArt
- Audio files
- Vector graphics
- Macros/VBA

See [slides-interoperability.md](slides-interoperability.md) for detailed compatibility notes.

---

## Authentication and Authorization

### OAuth 2.0 Scopes

| Scope | Level | Description |
|-------|-------|-------------|
| `https://www.googleapis.com/auth/presentations` | Sensitive | Full access to all Slides |
| `https://www.googleapis.com/auth/presentations.readonly` | Sensitive | Read-only access to all Slides |
| `https://www.googleapis.com/auth/drive.file` | Non-sensitive | Access only to files opened/created by app |
| `https://www.googleapis.com/auth/drive` | Restricted | Full Drive access |
| `https://www.googleapis.com/auth/drive.readonly` | Restricted | Read all Drive files |
| `https://www.googleapis.com/auth/spreadsheets.readonly` | Sensitive | Required for embedding Sheets charts |

**Recommendation:** Use the narrowest scope possible. `drive.file` is preferred as it's non-sensitive.

### Apps Script Authorization

Apps Script handles OAuth automatically when you run a script:

1. Script analyzes code for required scopes
2. User sees authorization prompt on first run
3. Token is stored and refreshed automatically

### Setting Scopes Explicitly

For published add-ons or stricter control, set scopes in `appsscript.json`:

```json
{
  "oauthScopes": [
    "https://www.googleapis.com/auth/presentations",
    "https://www.googleapis.com/auth/drive.file"
  ]
}
```

### Accessing OAuth Token

For advanced operations (like URL-based export):

```javascript
const token = ScriptApp.getOAuthToken();
```

---

## Limitations and Constraints

### API Rate Limits

| Quota Type | Limit |
|------------|-------|
| Read requests per minute (project) | 3,000 |
| Read requests per minute (per user) | 600 |
| Expensive reads (getThumbnail) per minute | 300 |
| Expensive reads per user per minute | 60 |
| Write requests per minute (project) | 600 |
| Write requests per minute (per user) | 60 |
| Daily requests | Unlimited (within per-minute quotas) |

**Error Handling:** Exceeding quotas returns HTTP 429. Implement exponential backoff.

### Apps Script Execution Limits

| Limit | Value |
|-------|-------|
| Script execution time | 6 minutes |
| Custom function execution | 30 seconds |
| Simultaneous executions | 30 |
| Triggers per user | 20 |
| URL Fetch response size | 50 MB |

### Feature Limitations

| Feature | Limitation |
|---------|------------|
| **Animations** | Cannot be created/edited via API (open feature request) |
| **Transitions** | Cannot be created/edited via API |
| **Theme colors** | Only first 12 ThemeColorTypes editable, only on Master pages |
| **Audio** | Limited support |
| **Shadows** | Read-only |
| **Custom fonts** | Must be Google Fonts |
| **Vector graphics** | Not supported (rasterized on import) |
| **Tables in groups** | Cannot be grouped |
| **Videos in groups** | Cannot be grouped |
| **Placeholder shapes** | Cannot be grouped |

### Object Constraints

| Object Type | Constraints |
|-------------|-------------|
| **Images** | Max 50 MB, max 25 megapixels, PNG/JPEG/GIF only |
| **Image URLs** | Max 2 KB, must be publicly accessible |
| **Object IDs** | 5-50 chars, alphanumeric + `_-:` |
| **Table cells** | Max merge depends on table size |

---

## Best Practices

### 1. Use Batch Updates

Combine multiple operations into single API calls:

```javascript
// GOOD: Single batch update
const requests = [
  { createSlide: {...} },
  { insertText: {...} },
  { updateTextStyle: {...} }
];
Slides.Presentations.batchUpdate({ requests }, presentationId);

// BAD: Multiple separate calls
createSlide(presentationId, ...);
insertText(presentationId, ...);
updateTextStyle(presentationId, ...);
```

### 2. Use Field Masks

Only update what you need:

```javascript
{
  updateTextStyle: {
    objectId: shapeId,
    style: { bold: true },
    fields: 'bold'  // Only update bold, preserve other styles
  }
}
```

### 3. Generate UUIDs for Object IDs

```javascript
const objectId = Utilities.getUuid();
```

### 4. Handle Rate Limits

```javascript
function batchUpdateWithRetry(presentationId, requests, maxRetries = 5) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return Slides.Presentations.batchUpdate({ requests }, presentationId);
    } catch (e) {
      if (e.message.includes('429') && attempt < maxRetries - 1) {
        const waitTime = Math.pow(2, attempt) * 1000 + Math.random() * 1000;
        Utilities.sleep(waitTime);
      } else {
        throw e;
      }
    }
  }
}
```

### 5. Use Templates Over Programmatic Creation

For complex layouts, create a template in the Slides UI and use the API for data population.

### 6. Validate Before Batch Updates

```javascript
function validateRequests(requests) {
  // Check for unique object IDs
  const ids = new Set();
  for (const request of requests) {
    const objectId = request.createSlide?.objectId ||
                     request.createShape?.objectId ||
                     request.createImage?.objectId;
    if (objectId) {
      if (ids.has(objectId)) {
        throw new Error(`Duplicate object ID: ${objectId}`);
      }
      ids.add(objectId);
    }
  }
  return true;
}
```

### 7. Cache Presentation Data

Minimize API calls by caching presentation structure:

```javascript
function getPresentationCached(presentationId) {
  const cache = CacheService.getScriptCache();
  const cached = cache.get('pres_' + presentationId);

  if (cached) {
    return JSON.parse(cached);
  }

  const presentation = Slides.Presentations.get(presentationId);
  cache.put('pres_' + presentationId, JSON.stringify(presentation), 300);
  return presentation;
}
```

---

## Code Examples

### Complete Presentation Generation

```javascript
function generatePresentation(data) {
  // Create presentation
  const presentation = Slides.Presentations.create({
    title: data.title
  });
  const presentationId = presentation.presentationId;

  // Get the default slide's ID
  const defaultSlideId = presentation.slides[0].objectId;

  // Build all requests
  const requests = [];

  // Delete default slide
  requests.push({
    deleteObject: { objectId: defaultSlideId }
  });

  // Create title slide
  const titleSlideId = Utilities.getUuid();
  requests.push({
    createSlide: {
      objectId: titleSlideId,
      slideLayoutReference: { predefinedLayout: 'TITLE' }
    }
  });

  // Create content slides
  data.slides.forEach((slideData, index) => {
    const slideId = Utilities.getUuid();
    const textBoxId = Utilities.getUuid();

    requests.push({
      createSlide: {
        objectId: slideId,
        insertionIndex: index + 1,
        slideLayoutReference: { predefinedLayout: 'TITLE_AND_BODY' }
      }
    });
  });

  // Execute batch update
  Slides.Presentations.batchUpdate({ requests }, presentationId);

  return presentationId;
}
```

### Data-Driven Slide Generation from Sheets

```javascript
function generateSlidesFromSheet(spreadsheetId, sheetName, templateId) {
  // Get data from sheet
  const sheet = SpreadsheetApp.openById(spreadsheetId).getSheetByName(sheetName);
  const data = sheet.getDataRange().getValues();
  const headers = data[0];

  // Process each row
  const presentations = [];
  for (let i = 1; i < data.length; i++) {
    const row = data[i];
    const rowData = {};
    headers.forEach((header, index) => {
      rowData['{{' + header + '}}'] = row[index];
    });

    // Create presentation from template
    const newPresId = createFromTemplate(templateId, row[0] + ' Presentation');
    textMerge(newPresId, rowData);
    presentations.push(newPresId);
  }

  return presentations;
}
```

### Export All Slides as Images

```javascript
function exportSlidesAsImages(presentationId, folderId) {
  const presentation = Slides.Presentations.get(presentationId);
  const folder = DriveApp.getFolderById(folderId);

  const imageUrls = [];
  presentation.slides.forEach((slide, index) => {
    const thumbnail = Slides.Presentations.Pages.getThumbnail(
      presentationId,
      slide.objectId,
      {
        'thumbnailProperties.mimeType': 'PNG',
        'thumbnailProperties.thumbnailSize': 'LARGE'
      }
    );

    const response = UrlFetchApp.fetch(thumbnail.contentUrl, {
      headers: { 'Authorization': 'Bearer ' + ScriptApp.getOAuthToken() }
    });

    const blob = response.getBlob().setName('slide_' + (index + 1) + '.png');
    const file = folder.createFile(blob);
    imageUrls.push(file.getUrl());
  });

  return imageUrls;
}
```

---

## Units and Coordinate Systems

### EMU (English Metric Units)

The Google Slides API uses EMU as its primary unit for positioning and sizing:

| Unit | EMU Equivalent |
|------|----------------|
| 1 inch | 914,400 EMU |
| 1 point | 12,700 EMU |
| 1 centimeter | 360,000 EMU |
| 1 pixel (96 DPI) | 9,525 EMU |

### Coordinate System

- **Origin**: Top-left corner of the slide
- **X-axis**: Increases to the right
- **Y-axis**: Increases downward
- **Default slide size**: 10" x 5.625" (16:9 aspect ratio)

### Transform Matrix

Page elements use affine transform matrices for positioning:

```javascript
{
  transform: {
    scaleX: 1,      // Horizontal scale factor
    scaleY: 1,      // Vertical scale factor
    shearX: 0,      // Horizontal shear
    shearY: 0,      // Vertical shear
    translateX: 0,  // X position in EMU
    translateY: 0,  // Y position in EMU
    unit: 'EMU'
  }
}
```

### Rotation

Rotation is achieved through the scale and shear components:

```javascript
// 90-degree rotation clockwise
{
  scaleX: 0,
  scaleY: 0,
  shearX: 1,
  shearY: -1
}
```

---

## Page Elements Deep Dive

### Element Types

The Slides API supports 8 distinct page element types:

| Type | Description | Key Features |
|------|-------------|--------------|
| **Shape** | Basic visual objects | Rectangles, ellipses, text boxes; can contain text |
| **Image** | Imported graphics | PNG, JPEG, GIF; max 50MB, 25 megapixels |
| **Video** | Embedded video | YouTube or Google Drive sources |
| **Table** | Grid of content | Row/column manipulation, cell merging |
| **Line** | Lines and connectors | Can connect shapes, various arrow styles |
| **Group** | Element collection | Treated as single unit for transforms |
| **WordArt** | Stylized text | Behaves like shape, limited text editing |
| **SheetsChart** | Embedded chart | Can be linked for live updates |

### Property Inheritance

Elements can inherit properties from:

1. **Master slides** - Base template styles
2. **Layout slides** - Specific layout variations
3. **Individual slides** - Override inherited properties

### Placeholder Types

Placeholders define content areas that inherit from layouts:

```
TITLE, SUBTITLE, BODY, HEADER, FOOTER,
SLIDE_NUMBER, DATE_AND_TIME, SLIDE_IMAGE,
OBJECT, CHART, TABLE, MEDIA
```

### Working with Placeholders

```javascript
function getPlaceholders(slideId) {
  const presentation = SlidesApp.openById('PRESENTATION_ID');
  const slide = presentation.getSlideById(slideId);
  const placeholders = slide.getPlaceholders();

  placeholders.forEach(placeholder => {
    const type = placeholder.getPlaceholderType();
    const index = placeholder.getPlaceholderIndex();
    Logger.log(`Type: ${type}, Index: ${index}`);
  });
}
```

### Z-Order Management

```javascript
// Built-in service
shape.bringToFront();
shape.sendToBack();
shape.bringForward();
shape.sendBackward();

// Advanced service - use updatePageElementsZOrder request
{
  updatePageElementsZOrder: {
    pageElementObjectIds: ['shape1', 'shape2'],
    operation: 'BRING_TO_FRONT' // or SEND_TO_BACK, BRING_FORWARD, SEND_BACKWARD
  }
}
```

---

## Linked Slides Feature

### Cross-Presentation Slide Linking

You can create slides that link to source slides in other presentations:

```javascript
function appendLinkedSlide(targetPresentationId, sourceSlide) {
  const presentation = SlidesApp.openById(targetPresentationId);

  // Append with linking mode
  const linkedSlide = presentation.appendSlide(
    sourceSlide,
    SlidesApp.SlideLinkingMode.LINKED
  );

  return linkedSlide;
}

function refreshLinkedSlide(presentationId, slideId) {
  const presentation = SlidesApp.openById(presentationId);
  const slide = presentation.getSlideById(slideId);

  // Sync with source
  slide.refreshSlide();
}

function unlinkSlide(presentationId, slideId) {
  const presentation = SlidesApp.openById(presentationId);
  const slide = presentation.getSlideById(slideId);

  // Break the link
  slide.unlink();
}
```

### Linking Behavior

- Parent master and layout pages are copied if they don't exist
- `refreshSlide()` updates content from source
- `unlink()` breaks the connection permanently

---

## Presentation Metadata

### Available Metadata Fields

```javascript
{
  presentationId: "unique-id",
  pageSize: {
    width: { magnitude: 9144000, unit: "EMU" },
    height: { magnitude: 5143500, unit: "EMU" }
  },
  slides: [...],
  title: "Presentation Title",
  masters: [...],
  layouts: [...],
  locale: "en_US",
  revisionId: "revision-string",
  notesMaster: {...}
}
```

### Important Limitation

**Page size cannot be changed via API** - All programmatically created presentations are 16:9 (10" x 5.625"). If you need different dimensions, create the presentation manually first, then use the API to modify content.

---

## Error Handling Patterns

### Common Error Codes

| HTTP Code | Meaning | Solution |
|-----------|---------|----------|
| 400 | Invalid Request | Check request syntax and parameters |
| 401 | Unauthorized | Verify authentication token |
| 403 | Permission Denied | Check OAuth scopes and file permissions |
| 404 | Not Found | Verify presentationId and objectId |
| 429 | Too Many Requests | Implement exponential backoff |
| 500 | Server Error | Retry with backoff |

### Robust Error Handling

```javascript
function safeApiCall(apiFunction, maxRetries = 5) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return apiFunction();
    } catch (error) {
      const message = error.message || '';

      // Rate limit - backoff and retry
      if (message.includes('429') || message.includes('RATE_LIMIT')) {
        const waitTime = Math.min(Math.pow(2, attempt) * 1000 + Math.random() * 1000, 64000);
        Logger.log(`Rate limited. Waiting ${waitTime}ms before retry ${attempt + 1}`);
        Utilities.sleep(waitTime);
        continue;
      }

      // Server error - retry
      if (message.includes('500') || message.includes('503')) {
        Utilities.sleep(1000 * (attempt + 1));
        continue;
      }

      // Client error - don't retry
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

---

## Comparison with Direct PPTX Generation

For the SlidesForAll project, understanding the tradeoffs is important:

| Feature | Google Slides API | Direct PPTX (python-pptx, etc.) |
|---------|-------------------|----------------------------------|
| **Animations** | Not supported | Full support |
| **Transitions** | Not supported | Full support |
| **Custom fonts** | Google Fonts only | Any embedded font |
| **Page size** | Fixed 16:9 | Any dimension |
| **Real-time collaboration** | Native | Not available |
| **Version history** | Native | Not available |
| **Export quality** | High (native export) | Native format |
| **Offline use** | Requires connectivity | Fully offline |
| **Audio embedding** | Limited | Full support |
| **Complex charts** | Via Sheets only | Direct embedding |
| **Platform** | Web-based | Any platform |
| **Cost** | Free | Free (libraries) |

### Recommendation for SlidesForAll

Consider a **hybrid approach**:
1. Use Google Slides API for collaborative, web-based workflows
2. Use direct PPTX generation (via Claude or python-pptx) for animation-heavy presentations
3. Provide export/import between formats with clear feature compatibility notes

See [research-claude-pptx.md](research-claude-pptx.md) for Claude's PPTX capabilities and [slides-interoperability.md](slides-interoperability.md) for format conversion details.

---

## Resources

### Official Documentation

- [Google Slides API Overview](https://developers.google.com/workspace/slides/api/guides/overview)
- [Apps Script Slides Service Reference](https://developers.google.com/apps-script/reference/slides)
- [Advanced Slides Service](https://developers.google.com/apps-script/advanced/slides)
- [Extending Google Slides Guide](https://developers.google.com/apps-script/guides/slides)
- [Editing and Styling Text](https://developers.google.com/workspace/slides/api/guides/styling)
- [Usage Limits](https://developers.google.com/workspace/slides/api/limits)
- [OAuth 2.0 Scopes](https://developers.google.com/workspace/slides/api/scopes)
- [Pages and Page Elements](https://developers.google.com/workspace/slides/api/concepts/page-elements)
- [Transforms](https://developers.google.com/workspace/slides/api/concepts/transforms)
- [Size and Position](https://developers.google.com/workspace/slides/api/guides/transform)
- [Merge Data into Presentations](https://developers.google.com/workspace/slides/api/guides/merge)
- [Speaker Notes](https://developers.google.com/workspace/slides/api/guides/notes)

### Code Samples

- [Google Workspace Apps Script Samples](https://github.com/googleworkspace/apps-script-samples/blob/main/slides/api/Snippets.gs)
- [Slides API Quickstart](https://developers.google.com/workspace/slides/api/quickstart/apps-script)

### Tutorials and Guides

- [Generating Slides from Spreadsheet Data](https://workspace.google.com/blog/developers-practitioners/generating-slides-from-spreadsheet-data)
- [Adding Charts from Sheets](https://developers.google.com/workspace/slides/api/guides/add-chart)
- [Working with Speaker Notes](https://developers.google.com/workspace/slides/api/guides/notes)
- [Create a Copy of Presentation](https://spreadsheet.dev/create-copy-google-slides-presentation-apps-script)
- [Using Google Slides API with Apps Script](https://www.labnol.org/code/20285-google-slides-api)

### Related Project Documents

- [research-claude-pptx.md](research-claude-pptx.md) - Claude's PPTX generation capabilities
- [slides-interoperability.md](slides-interoperability.md) - Cross-format compatibility matrix
- [markdown-to-slides.md](markdown-to-slides.md) - Markdown parsing for slide content

---

## Summary for SlidesForAll Project

### Key Takeaways

1. **Google Slides API is powerful but not universal** - Great for Google ecosystem, but animations/transitions require manual creation via UI

2. **Template-based workflows are the sweet spot** - Create templates in UI, use API for data population via `replaceAllText` and `replaceAllShapesWithImage`

3. **Batch updates are essential** - Combine operations for performance and atomicity; never call batchUpdate in loops

4. **Export to PPTX is possible** - Via Drive API/URL export, but some features may not transfer perfectly (especially animations)

5. **Import from PPTX has limitations** - Complex animations, certain fonts, and some embedded content may be lost or converted

6. **Speaker notes are fully accessible** - Can read and write programmatically (text only, not formatting)

7. **Charts integration is excellent** - Embedded Sheets charts can be linked and refreshed dynamically

8. **Page size is fixed** - Cannot programmatically create non-16:9 presentations; must start from manual template

9. **No animation API** - This is a known limitation with an open feature request on Google's issue tracker

10. **Free with generous quotas** - 600 writes/min, 3000 reads/min per project; no daily limits if per-minute quotas respected

### Recommended Architecture for SlidesForAll

1. **Input Processing**: Parse markdown/structured content into slide data model (see [markdown-to-slides.md](markdown-to-slides.md))
2. **Template Selection**: Match content to appropriate predefined layouts or custom templates
3. **Batch Generation**: Use Advanced Slides Service with batch updates for efficiency
4. **Export Options**: Provide PPTX and PDF export via Drive API URL endpoints
5. **Preview**: Use thumbnail API for slide previews (note: 60 requests/min/user limit)
6. **Hybrid Strategy**: Consider direct PPTX generation for animation-heavy presentations (see [research-claude-pptx.md](research-claude-pptx.md))

### Decision Matrix: Google Slides API vs Direct PPTX

| Use Case | Recommended Approach |
|----------|---------------------|
| Collaborative presentations | Google Slides API |
| Animation-heavy presentations | Direct PPTX generation |
| Template-based reports | Google Slides API |
| Offline-first workflows | Direct PPTX generation |
| Real-time data dashboards | Google Slides API (with Sheets) |
| Custom page sizes | Direct PPTX generation |
| K-12 classroom use | Google Slides API (collaboration features) |

See [slides-interoperability.md](slides-interoperability.md) for format conversion strategies and [markdown-to-slides.md](markdown-to-slides.md) for content parsing approaches.
