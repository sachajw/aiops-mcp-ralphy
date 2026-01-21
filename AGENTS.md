# Agent Coding Guidelines

## Project Structure

TypeScript monorepo with two packages:
- `cli/` - Ralphy CLI (Bun runtime)
- `landing/` - Next.js site (Node runtime)

## Build Commands

### CLI (`cli/`)
```bash
bun run dev                    # Run CLI directly
bun run build:all              # Build for all platforms
bun run build:darwin-arm64      # macOS ARM
bun run build:darwin-x64        # macOS Intel
bun run build:linux-x64         # Linux x64
bun run build:linux-arm64        # Linux ARM
bun run build:windows-x64        # Windows
bun run check                   # Run Biome linter + formatter
tsc --noEmit                  # Type check
```

### Landing (`landing/`)
```bash
npm run dev                    # Start Next.js dev server
npm run start                  # Start production server
npm run build                  # Build for production
npm run lint                   # Run Next.js linter
tsc --noEmit                  # Type check
```

## Running Tests

No test suite exists in this project. Tests are NOT required by default.

When adding tests to this codebase, configure test scripts in package.json and document the command to run a single test here.

## Code Style Guidelines

### Formatting
- **Indentation**: Tabs (required by Biome)
- **Line width**: 100 characters
- **Tool**: Biome (auto-organizes imports, lints, formats)
- Run `bun run check` from `cli/` or `npm run lint` from `landing/` before committing

### TypeScript Configuration
- **Strict mode**: Always enabled
- **Module system**: ESNext with bundler resolution
- **Target**: ES2022 (CLI) / ES2017 (landing)
- **No emit**: Type checking only
- **File extensions**: Allow `.ts` imports

### Imports
```typescript
// External dependencies
import pc from "picocolors";
import { z } from "zod";

// Relative imports within package (use .ts extensions)
import { loadBoundaries } from "../config/loader.ts";
import type { AIEngine } from "./types.ts";
```

### Naming Conventions
- **Variables/functions**: `camelCase` - `executeStreaming`, `workDir`
- **Classes**: `PascalCase` - `ClaudeEngine`, `ProgressSpinner`
- **Interfaces/Types**: `PascalCase` - `AIResult`, `ExecutionOptions`
- **Constants**: `UPPER_CASE` - `DEFAULT_OPTIONS`
- **Files**: `kebab-case` - `claude.ts`, `task-runner.ts`

### Error Handling
```typescript
// Always wrap async operations in try-catch
async function main(): Promise<void> {
  try {
    const result = await execute();
    return result;
  } catch (error) {
    logError(error instanceof Error ? error.message : String(error));
    process.exit(1);
  }
}

// Return structured error objects
if (error) {
  return { success: false, response: "", inputTokens: 0, outputTokens: 0, error };
}

// Simple catch blocks that ignore errors
try {
  const proc = Bun.spawn(["which", command]);
  return await proc.exited === 0;
} catch {
  return false;
}

### Function Documentation (JSDoc)
```typescript
/**
 * Build full prompt with project context, rules, boundaries, and task
 */
export function buildPrompt(options: PromptOptions): string {
  // ...
}
```

### Async/Await Patterns
- Always use `async/await`, never `.then()`
- Prefer `Promise.all()` for parallel operations
```typescript
const [stdout, stderr, exitCode] = await Promise.all([
  new Response(proc.stdout).text(),
  new Response(proc.stderr).text(),
  proc.exited,
]);
```

### Type Definitions
```typescript
// Use interfaces for object shapes with methods
export interface AIEngine {
  name: string;
  cliCommand: string;
  execute(prompt: string, workDir: string, options?: EngineOptions): Promise<AIResult>;
}

// Use Zod schemas for runtime validation
export const ProjectSchema = z.object({
  name: z.string().default(""),
  language: z.string().default(""),
});
export type Project = z.infer<typeof ProjectSchema>;
```

### Logging
```typescript
import { logInfo, logSuccess, logWarn, logError, logDebug } from "../ui/logger.ts";
logInfo("Starting...");
logSuccess("Done!");
logWarn("Warning");
logError("Failed");
logDebug("Debug", { key: "value" }); // Only when verbose=true
```

### File Organization
- Each module has `index.ts` re-exporting public API
- Group related files in directories: `cli/src/engines/`, `cli/src/config/`
- Keep files focused (<300 lines preferred)

## Runtime Specifics

### CLI (Bun)
- Target: Bun runtime (Bun.spawn, Bun.build)
- Type checking: `@types/bun`
- Build: `bun build --compile --minify --target=<platform>`

### Landing (Next.js)
- Target: Node.js 18+, Next.js 16 App Router, Tailwind CSS 4

## Common Patterns

### Config Loading
```typescript
const config = RalphyConfigSchema.parse(data); // Use Zod validation
const { task, autoCommit = true, workDir = process.cwd() } = options; // Default destructuring
```

### Command Execution
```typescript
const proc = Bun.spawn([command, ...args], {
  cwd: workDir,
  stdout: "pipe",
  stderr: "pipe",
  env: { ...process.env, ...env },
});
const exitCode = await proc.exited;
```

### Git Workflow
1. Create feature branch from main
2. Implement changes following style guidelines
3. Run `bun run check` (CLI) or `npm run lint` (landing)
4. Test manually, commit with descriptive message
5. Create PR for human review
