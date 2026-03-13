# Contributing to ccstatusline

Thanks for your interest in contributing! This guide will help you get started.

## Getting Started

1. Fork the repository
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/ccstatusline.git
   cd ccstatusline
   ```
3. Install dependencies:
   ```bash
   bun install
   ```

## Development

ccstatusline uses **Bun** as its primary runtime. Use Bun for all commands:

| Command | Purpose |
|---------|---------|
| `bun run start` | Launch interactive TUI configuration editor |
| `bun run example` | Test piped mode with a sample payload |
| `bun test` | Run the test suite (Vitest) |
| `bun test --watch` | Run tests in watch mode |
| `bun run lint` | TypeScript type check + ESLint (zero warnings) |
| `bun run lint:fix` | Apply ESLint auto-fixes (use intentionally) |
| `bun run build` | Build `dist/ccstatusline.js` for Node.js 14+ |

### Testing piped mode manually

```bash
echo '{"model":{"id":"claude-sonnet-4-5-20250929"},"transcript_path":"test.jsonl"}' | bun run src/ccstatusline.ts
```

## Code Style

- **4-space indentation** for TypeScript, TSX, JavaScript, and JSON
- Single quotes for strings
- Strict TypeScript (type checking via `tsc --noEmit`)
- ESLint with **zero warnings** policy
- **Never disable lint rules via comments** — fix the underlying issue instead
- Run `bun run lint` before committing

## Creating a New Widget

Widgets are the building blocks of ccstatusline. To create one:

1. **Implement the Widget interface** from `src/types/Widget.ts`:

   ```typescript
   export class MyWidget implements Widget {
       getDefaultColor(): string { return 'cyan'; }
       getDescription(): string { return 'Description for the TUI editor'; }
       getDisplayName(): string { return 'My Widget'; }
       getCategory(): string { return 'General'; }

       getEditorDisplay(item: WidgetItem): WidgetEditorDisplay {
           return { displayText: this.getDisplayName() };
       }

       render(item: WidgetItem, context: RenderContext, settings: Settings): string | null {
           // Return formatted string, or null to hide
           return 'output';
       }

       supportsRawValue(): boolean { return true; }
       supportsColors(item: WidgetItem): boolean { return true; }
   }
   ```

2. **Register it** in `src/utils/widgets.ts` by adding to the `widgetRegistry` Map

3. **Add tests** in `src/widgets/__tests__/MyWidget.test.ts`

4. **Test both modes**: Verify rendering in piped mode and appearance in the TUI editor

## Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

| Prefix | Purpose |
|--------|---------|
| `feat:` | New feature |
| `fix:` | Bug fix |
| `docs:` | Documentation only |
| `test:` | Adding or updating tests |
| `refactor:` | Code change that doesn't fix a bug or add a feature |
| `chore:` | Build process, tooling, or dependency changes |
| `deps:` | Dependency updates |

## Pull Requests

1. Create a feature branch from `main`
2. Make your changes with clear, focused commits
3. Ensure CI passes: `bun run lint && bun test && bun run build`
4. Fill out the PR template — describe what changed and how you tested it
5. Keep PRs focused on a single concern

## Reporting Bugs

Use the [Bug Report](https://github.com/sirmalloc/ccstatusline/issues/new?template=bug.yml) issue template. Include your OS, terminal, ccstatusline version, and whether the issue occurs in piped mode, TUI mode, or both.

## Suggesting Features

Use the [Feature Request](https://github.com/sirmalloc/ccstatusline/issues/new?template=feature.yml) issue template. Describe the problem you're solving, not just the solution you want.

## Security

For security vulnerabilities, please see our [Security Policy](SECURITY.md). Do not open public issues for security concerns.