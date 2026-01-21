# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Ralphy** is an autonomous AI coding loop that orchestrates AI agents to complete development tasks until a PRD is finished. It's a TypeScript monorepo with two packages:

- `cli/` - Main Ralphy CLI tool (Bun runtime)
- `landing/` - Next.js marketing website (Node runtime)

## Build Commands

### CLI (`cli/`) - Bun Runtime
```bash
bun run dev                    # Run CLI directly
bun run build:all              # Build for all platforms
bun run build:darwin-arm64     # macOS ARM
bun run build:darwin-x64       # macOS Intel
bun run build:linux-x64        # Linux x64
bun run build:linux-arm64      # Linux ARM
bun run build:windows-x64      # Windows
bun run check                  # Run Biome linter + formatter
tsc --noEmit                   # Type check
```

### Landing (`landing/`) - Node Runtime
```bash
npm run dev                    # Start Next.js dev server
npm run build                  # Build for production
npm run start                  # Start production server
npm run lint                   # Run Next.js linter
tsc --noEmit                   # Type check
```

**Note**: This project has no test suite. Tests are NOT required by default.

## Code Style

- **Indentation**: Tabs (required by Biome)
- **Line width**: 100 characters
- **Tool**: Biome for linting/formatting
- Run `bun run check` from `cli/` or `npm run lint` from `landing/` before committing

Naming conventions:
- Variables/functions: `camelCase`
- Classes/Interfaces/Types: `PascalCase`
- Constants: `UPPER_CASE`
- Files: `kebab-case`

## Architecture Overview

The CLI follows a **plugin-based architecture** with these key layers:

1. **CLI Layer** (`cli/src/cli/`) - Argument parsing with Commander.js, command implementations for run/task/init/config

2. **Configuration System** (`cli/src/config/`) - Zod-validated config loading, project type auto-detection (Node/Python/Go/Rust), progress tracking

3. **AI Engine Abstraction** (`cli/src/engines/`) - `AIEngine` interface with multiple implementations (Claude, OpenCode, Cursor, Codex, Qwen, Droid). Each engine adapts its specific CLI to a standard `AIResult` type.

4. **Task Sources** (`cli/src/tasks/`) - Pluggable task sources: Markdown checkboxes, YAML lists, GitHub Issues. Each source implements `TaskSource` interface with `markComplete()` for progress tracking.

5. **Execution Strategies** (`cli/src/execution/`) - Sequential (single-threaded) vs Parallel (multiple agents with Git worktrees). Parallel mode uses isolated worktrees to prevent conflicts, with AI-powered merge conflict resolution.

6. **Git Operations** (`cli/src/git/`) - Worktree management, branch creation, merging with AI conflict resolution, PR creation via GitHub CLI

7. **UI Layer** (`cli/src/ui/`) - Colored logging, progress spinners with step detection, desktop notifications

### Data Flow

```
CLI args → parseArgs() → RuntimeOptions
    ↓
runLoop() / runTask()
    ↓
TaskSource (Markdown/YAML/GitHub)
    ↓
buildPrompt() → adds context + rules + boundaries from .ralphy/config.yaml
    ↓
AIEngine.execute() → runs claude/opencode/etc with streaming JSON output
    ↓
AIResult (success, response, tokens)
    ↓
TaskSource.markComplete()
    ↓
Git operations (branch/merge/PR)
```

### Key Patterns

- **Strategy Pattern**: Multiple AI engines implement `AIEngine` interface
- **Factory Pattern**: Task sources created based on file type detection
- **Template Method**: `BaseAIEngine` provides base execution with command execution utilities
- **Streaming JSON**: Engines stream progress updates; step detection parses tool calls ("Reading code", "Testing", etc.)

### Critical Design Decisions

1. **Zod for Validation** - All configuration uses runtime type validation
2. **Git Worktrees for Parallelism** - Isolated working directories prevent conflicts between agents
3. **Auto-Detection** - Project type inferred from package.json/pyproject.toml/go.mod
4. **Browser Automation** - Optional agent-browser integration for UI testing
5. **AI Merge Resolution** - In parallel mode, AI resolves Git conflicts during auto-merge

### Runtime Differences

- **CLI**: Uses Bun runtime (`Bun.spawn`, `Bun.build`) with `@types/bun`
- **Landing**: Node.js 18+, Next.js 16 App Router, Tailwind CSS 4
