# Copilot Instructions for ccstatusline

## Project Context

ccstatusline is a customizable status line formatter for Claude Code CLI, built with TypeScript and Bun. It operates in two modes:
- **Piped mode**: Receives JSON from Claude Code stdin, renders a formatted status line
- **TUI mode**: Interactive React/Ink terminal UI for configuration

## Code Style

- **4-space indentation** for TypeScript/TSX/JS/JSON
- Single quotes for strings
- Strict TypeScript (`noEmit` with `tsc`)
- ESLint with zero warnings policy — never disable lint rules via comments
- Use `bun run lint` to check; `bun run lint:fix` only for intentional auto-fixes

## Development

- Use **Bun** instead of Node.js for all commands:
  - `bun install` (not npm install)
  - `bun test` (Vitest via Bun)
  - `bun run build` (builds Node.js 14+ compatible dist)
  - `bun run lint` (TypeScript + ESLint)
- Build produces `dist/ccstatusline.js` targeting Node.js 14+

## Widget Pattern

All widgets implement the `Widget` interface from `src/types/Widget.ts`:

```typescript
interface Widget {
    getDefaultColor(): string;
    getDescription(): string;
    getDisplayName(): string;
    getCategory(): string;
    getEditorDisplay(item: WidgetItem): WidgetEditorDisplay;
    render(item: WidgetItem, context: RenderContext, settings: Settings): string | null;
    supportsRawValue(): boolean;
    supportsColors(item: WidgetItem): boolean;
}
```

Widgets are registered in `src/utils/widgets.ts` via the `widgetRegistry` Map.

## Testing

- Framework: Vitest (run via `bun test`)
- Test files: `src/**/__tests__/*.test.ts`
- Write tests for new widgets in `src/widgets/__tests__/`

## Commit Messages

Use conventional commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`, `deps:`

## Security

- Never hardcode secrets or credentials
- Validate external input at system boundaries
- Custom Command widgets execute shell commands by design — this is documented behavior