# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Dev server with live reload
npx quartz build --serve

# Production build (outputs to public/)
npx quartz build

# Type check + prettier
npm run check

# Auto-format
npm run format

# Run all tests
npm test

# Run a single test file
tsx --test quartz/util/path.test.ts
```

## Architecture

Quartz is a static site generator that converts Markdown (from `content/`) into a website (`public/`). The build pipeline has three stages, each driven by plugins configured in `quartz.config.ts`:

1. **Transformers** (`quartz/plugins/transformers/`) — operate on individual files. They can hook into the unified/remark/rehype pipeline via `markdownPlugins` / `htmlPlugins`, or do raw text transforms via `textTransform`. Examples: `FrontMatter`, `ObsidianFlavoredMarkdown`, `CrawlLinks`.

2. **Filters** (`quartz/plugins/filters/`) — decide which processed files to include. `RemoveDrafts` is the canonical example (drops files with `draft: true` frontmatter).

3. **Emitters** (`quartz/plugins/emitters/`) — consume all processed content and write output files. Some emitters support `partialEmit` for incremental rebuilds (watch mode).

The orchestration lives in `quartz/build.ts`: glob → `parseMarkdown` → `filterContent` → `emitContent`. Watch mode reuses this pipeline with `chokidar` and coalesces rapid changes before rebuilding.

**Key config files:**
- `quartz.config.ts` — plugin list, theme, locale, analytics, `baseUrl`
- `quartz.layout.ts` — UI component layout for content pages vs. list pages

**Component system** (`quartz/components/`): UI components are JSX/TSX React-like components exported from `quartz/components/index.ts`. Layout is split into `SharedLayout` (head/header/footer, all pages) and `PageLayout` (left/right/beforeBody, per page type). Components can ship their own client-side scripts and styles via `componentResources.ts`.

**Plugin interface** (`quartz/plugins/types.ts`): All three plugin types are factory functions `(opts?) => PluginInstance`. Instances carry a `name` string plus lifecycle hooks specific to their type.

**Content** lives in `content/` — Obsidian-flavored Markdown. Files in `private/`, `templates/`, and `.obsidian/` are ignored (configured in `quartz.config.ts` `ignorePatterns`).
