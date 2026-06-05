# CLAUDE.md

## About This Project

**Standup Wheel of Fortune** - A single-file HTML/CSS/JS spinning wheel app for randomizing standup order. Zero dependencies, zero build step.

## Architecture

Everything lives in `index.html`. No framework, no bundler, no external assets. The app uses:

- Canvas API for wheel rendering
- Web Audio API for sound effects (tick + fanfare)
- localStorage for team persistence
- FileReader API for .txt uploads

There is no server component. Open the file in a browser and it works.

## Development

There is no build step. To work on it:

```bash
open index.html
# or use any local server, e.g.:
python3 -m http.server 8000
```

No linter, no formatter, no test suite configured. The spec (`wheel-spec.md`) serves as the source of truth for intended behavior.

## Key Implementation Details

- All state is in plain JS variables at the top of the `<script>` block
- Color assignment is index-based and sticky (removing a name does not reassign colors)
- Spin physics use cubic ease-out over a fixed duration (10s normal, 3s single-name)
- The flapper uses spring physics with velocity kicks at slice boundaries
- localStorage key: `standup-wheel-files` (JSON object mapping filenames to name arrays)
- The edit modal handles both "edit existing" and "create new" flows

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire application |
| `wheel-spec.md` | Complete technical specification |
| `test-names.txt` | Sample names for testing |
| `images/` | Screenshots for documentation |

## Conventions

- Self-contained: all changes go into `index.html` unless adding documentation
- No external dependencies. Ever.
- When modifying behavior, update `wheel-spec.md` to match
- Keep the carnival/game show aesthetic consistent (dark theme, gold accents, glow effects)
