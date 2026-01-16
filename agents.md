# agents.md - Multi-Agent Workflow Guide

**Coordination guide for AI assistants working on SlidesForAll**

This document provides patterns for effective collaboration when multiple AI coding assistants (Claude Code, Cursor, GitHub Copilot, Windsurf, etc.) work on this project, whether in parallel or across sessions.

---

## Core Principles

### 1. Read First, Act Second

**Always read these files before starting work:**

```
Priority 1 (Must Read):
├── CLAUDE.md          # Project context, architecture, decisions
└── README.md          # Quick reference, current state

Priority 2 (Task-Specific):
├── markdown-to-slides.md          # If parsing markdown
├── research-google-slides-api.md  # If generating Google Slides
├── research-claude-pptx.md        # If generating PPTX
└── slides-interoperability.md     # If handling format conversion
```

### 2. Document As You Go

**Update files immediately after making decisions:**

- Architecture change? → Update `CLAUDE.md`
- New limitation discovered? → Update relevant research doc
- Implementation progress? → Update `README.md` checklist

### 3. Use the Content Model as Contract

**Never bypass the content model:**

```
✅ GOOD: Parse markdown → Content model → Generate slides
❌ BAD:  Parse markdown → Directly call Google Slides API
```

The content model (`Presentation`, `Slide`, `SlideElement` types) is the contract between components.

### 4. Batch Over Iterate

**Prefer parallel work over sequential when possible:**

```
✅ GOOD: Work on parser AND generator simultaneously (different agents)
❌ BAD:  Wait for parser to be 100% done before starting generator
```

As long as the content model interface is stable, components can develop in parallel.

---

## Task Delegation Patterns

### Pattern 1: Component-Based Parallelism

**Scenario**: Multiple agents working simultaneously

```
Agent A: Parser Implementation
├── src/parser/markdown.ts
├── src/parser/slides.ts
├── src/parser/notes.ts
└── tests/parser/*.test.ts

Agent B: Generator Implementation
├── src/generators/google-slides.ts
├── src/generators/pptx.ts
└── tests/generators/*.test.ts

Agent C: CLI & Integration
├── src/cli/index.ts
├── src/index.ts
└── tests/integration/*.test.ts
```

**Coordination**: Content model types (`src/model/types.ts`) are the shared interface. Lock this file first, then work independently.

### Pattern 2: Research-Then-Implement

**Scenario**: One agent researches, another implements

```
Session 1 (Research Agent):
├── Research API capabilities
├── Document in research-*.md
├── Update CLAUDE.md with findings
└── Propose content model

Session 2 (Implementation Agent):
├── Read research docs
├── Implement based on documented patterns
├── Ask questions if research incomplete
└── Report gaps back to CLAUDE.md
```

**Coordination**: Research docs are the handoff point. Implementation agent should NOT need to re-research APIs.

### Pattern 3: Feature Branches

**Scenario**: Experimental work that may not merge

```
Main Branch:
├── Stable parser
└── Google Slides generator (primary)

Feature Branch: pptx-generator
├── PPTX generator implementation
└── Claude PPTX skill integration

Feature Branch: marp-export
├── HTML/Marp generator
└── Theme CSS generation
```

**Coordination**: Use git branches. Document experimental status in `CLAUDE.md`.

---

## Communication Protocol

### When Starting a Session

1. **Announce intent** - "I'm implementing the markdown parser"
2. **Check for conflicts** - Read `CLAUDE.md` to see if work is in progress
3. **Lock resources** - Update `CLAUDE.md` with "In Progress" note
4. **Read dependencies** - Check research docs for your component

### When Completing Work

1. **Update README** - Check off completed items
2. **Document decisions** - Add to `CLAUDE.md` if architecture changed
3. **Unlock resources** - Remove "In Progress" note
4. **Summarize** - Add brief summary to `notes.md` or commit message

### When Blocked

1. **Document the blocker** - Add to `CLAUDE.md` under "Known Issues"
2. **Provide context** - What you tried, why it didn't work
3. **Ask specific questions** - Not "how to do X?" but "should we do X or Y?"
4. **Move to unblocked work** - Work on different component if possible

---

## Code Ownership & Merge Strategy

### File Ownership (Soft)

| File(s) | Primary Owner | Merge Strategy |
|---------|--------------|----------------|
| `src/model/types.ts` | Architecture Agent | Must review all changes |
| `src/parser/*.ts` | Parser Agent | Can modify freely |
| `src/generators/*.ts` | Generator Agents | One generator per agent |
| `tests/*.test.ts` | Respective component owner | Co-located with code |
| `CLAUDE.md`, `README.md` | All agents | Append-only, coordinate |
| Research docs | Read-only during implementation | Update for corrections only |

### Merge Conflicts

**If two agents modify the same file:**

1. **Content model changes** - Discuss and align on types first
2. **Implementation changes** - Later agent rebases or merges
3. **Documentation changes** - Append new sections rather than edit existing
4. **Test changes** - Should rarely conflict if well-organized

---

## Testing Responsibilities

### Unit Tests

**Rule**: Each component owner writes unit tests for their component.

```
src/parser/markdown.ts      → tests/parser/markdown.test.ts
src/generators/google.ts    → tests/generators/google.test.ts
```

### Integration Tests

**Rule**: Integration tests are shared responsibility.

```
tests/integration/
├── markdown-to-slides.test.ts       # Full pipeline test
├── fixtures/
│   ├── simple.md                    # Test input
│   ├── complex.md                   # Test input
│   └── expected-output.json         # Expected content model
```

### Manual Tests

**Rule**: Generator owners validate output in actual platforms.

- Google Slides generator → Open in Google Slides, verify rendering
- PPTX generator → Open in PowerPoint AND Google Slides (import test)
- HTML generator → Open in browser, verify layout

---

## Common Scenarios

### Scenario 1: "I want to add a new slide layout"

**Steps:**

1. **Check content model** - Does `Slide.layout` support it?
   - If no: Propose type change in `CLAUDE.md`, get approval
   - If yes: Proceed

2. **Update parser** - Detect new layout from markdown structure

3. **Update generators** - Implement layout in each generator
   - Google Slides: Map to predefined layout or create custom
   - PPTX: Map to corresponding PPTX layout
   - HTML: Add CSS styling

4. **Test** - Add fixture markdown file, verify all generators

5. **Document** - Update `markdown-to-slides.md` with layout syntax

### Scenario 2: "I found a limitation in the Google Slides API"

**Steps:**

1. **Verify** - Check `research-google-slides-api.md` to see if already documented

2. **Document** - Add to "Limitations" section in research doc

3. **Decide** - Does this affect architecture?
   - If yes: Update `CLAUDE.md` with decision (workaround, accept limitation, etc.)
   - If no: Just document the limitation

4. **Communicate** - Add note to `README.md` if it affects user-visible features

### Scenario 3: "I want to refactor the content model"

**Steps:**

1. **Propose** - Add proposal to `CLAUDE.md` under "Proposed Changes"

2. **Get feedback** - User or other agents review

3. **Create migration plan** - Document impact on parser and generators

4. **Implement incrementally**:
   - Update types
   - Update parser to use new types
   - Update generators to use new types
   - Update tests

5. **Document** - Update `CLAUDE.md` with new architecture

### Scenario 4: "I'm starting fresh and don't know what to work on"

**Steps:**

1. **Read `README.md`** - Check "Next Steps" section

2. **Read `CLAUDE.md`** - Understand current state

3. **Check for "In Progress" notes** - Avoid duplicate work

4. **Pick an unclaimed task** - Ideally a leaf node (no dependencies)

5. **Announce** - Update `CLAUDE.md` with "In Progress: [Task] - [Your Name/Session]"

---

## Quality Standards

### Code Review Checklist

Before committing code, verify:

- [ ] TypeScript compiles without errors (`npm run build` or `tsc`)
- [ ] Tests pass (`npm test`)
- [ ] Code follows style guide (Prettier formatted)
- [ ] Public functions have JSDoc comments
- [ ] No platform-specific logic in content model
- [ ] Batch updates used for API calls (not loops)
- [ ] Error handling in place for API failures
- [ ] Content model types are immutable (prefer readonly)

### Documentation Review Checklist

Before updating docs, verify:

- [ ] No contradictions with existing docs
- [ ] Links to related docs included (`[[wikilinks]]` style)
- [ ] Examples are tested and accurate
- [ ] Limitations clearly stated
- [ ] Last updated date set

---

## Anti-Patterns to Avoid

### ❌ Platform Leakage

```typescript
// BAD: Google Slides concepts leak into content model
interface Slide {
  objectId: string;  // Google Slides specific
  emu_width: number; // EMU is Google Slides unit
}

// GOOD: Platform-agnostic content model
interface Slide {
  id: string;
  width: number;  // In points or percentage
}
```

### ❌ Silent Failures

```typescript
// BAD: Fail silently
function parseMarkdown(md: string): Presentation {
  try {
    return parse(md);
  } catch (e) {
    return { slides: [] };  // User doesn't know parsing failed!
  }
}

// GOOD: Propagate errors
function parseMarkdown(md: string): Presentation {
  try {
    return parse(md);
  } catch (e) {
    throw new Error(`Failed to parse markdown: ${e.message}`);
  }
}
```

### ❌ Overwriting Others' Work

```typescript
// BAD: Rewrite entire file without reading it
await Write({
  file_path: "src/model/types.ts",
  content: "// My new types..."
});

// GOOD: Read first, then Edit
const existing = await Read({ file_path: "src/model/types.ts" });
// Review, then edit specific sections
await Edit({
  file_path: "src/model/types.ts",
  old_string: "...",
  new_string: "..."
});
```

### ❌ Tight Coupling

```typescript
// BAD: Parser directly calls Google Slides API
function parseSlide(md: string) {
  const slide = parseMarkdownSlide(md);
  Slides.Presentations.batchUpdate({ ... });  // Coupling!
}

// GOOD: Parser returns content model
function parseSlide(md: string): Slide {
  return {
    title: extractTitle(md),
    elements: extractElements(md)
  };
}
```

---

## Emergency Protocols

### If Work is Accidentally Duplicated

1. **Compare implementations** - Which is more complete/correct?
2. **Merge best of both** - Take best parts from each
3. **Update CLAUDE.md** - Document what happened, how to avoid
4. **Add coordination note** - For future agents

### If Breaking Change Needed

1. **Document impact** - What breaks? What needs updating?
2. **Create migration branch** - Don't break main branch
3. **Update all affected components** - Parser, generators, tests
4. **Test thoroughly** - Full integration test suite
5. **Merge atomically** - All-or-nothing commit

### If Research is Wrong

1. **Verify** - Double-check the API/documentation
2. **Update research doc** - Correct the mistake
3. **Update CLAUDE.md** - If architecture decision was based on wrong info
4. **Notify** - Add note to `notes.md` about correction
5. **Fix implementations** - If any code was built on wrong assumption

---

## Success Metrics

**A well-coordinated multi-agent workflow should have:**

- ✅ Zero merge conflicts (good file ownership)
- ✅ Consistent content model usage (good abstraction)
- ✅ Up-to-date documentation (good discipline)
- ✅ Passing tests (good quality)
- ✅ No duplicate work (good communication)
- ✅ Fast iteration (good parallelism)

---

## Example Session Log

```markdown
## 2026-01-15 14:30 - Agent A (Parser Implementation)

**Task**: Implement markdown parser

**Status**: In Progress

**Files Modified**:
- src/parser/markdown.ts (created)
- tests/parser/markdown.test.ts (created)

**Dependencies**: None (content model already defined)

**Blockers**: None

**Next**: Complete heading extraction, then move to list parsing

---

## 2026-01-15 15:45 - Agent B (Google Slides Generator)

**Task**: Implement Google Slides generator

**Status**: In Progress

**Files Modified**:
- src/generators/google-slides.ts (created)

**Dependencies**: Content model types (stable)

**Blockers**: Need Google API credentials for testing

**Next**: Implement text formatting, then test with real API

---

## 2026-01-15 17:00 - Agent A (Parser Implementation)

**Task**: Markdown parser complete

**Status**: Complete

**Commit**: abc123 "Add markdown parser with full slide extraction"

**Tests**: 15/15 passing

**Next**: Ready for integration testing with generator
```

---

## Quick Reference Commands

### For Claude Code

```bash
# Read all context before starting
/read CLAUDE.md
/read README.md

# Create work log
/write notes.md  # Append session notes

# Coordinate with other agents
/edit CLAUDE.md  # Update "In Progress" section
```

### For Cursor/Other IDEs

- Read `CLAUDE.md` and `README.md` first
- Use git branches for experimental work
- Update docs inline as you work
- Run tests before committing: `npm test`

---

*Last updated: 2026-01-15*
