# md2html

Convert markdown files to clean, printer-friendly HTML. Single Python 3 script, zero dependencies.
Created by Omar Youssef (omar@omaryoussef.com).

## Install

To use `md2html` from anywhere, symlink it into your PATH:

```bash
sudo ln -s "$(pwd)/md2html" /usr/local/bin/md2html
```

## Usage

```
md2html <file-or-folder> [destination-folder]
```

### Examples

```bash
# Convert a single file (output goes to the same directory)
./md-to-html README.md

# Convert all .md files in a folder
./md-to-html docs/

# Output to a different folder
./md-to-html docs/ build/

# Single file to a specific folder
./md-to-html README.md build/
```

## Output

Each `.md` file produces a self-contained `.html` file with:

- Embedded CSS (no external stylesheets)
- 8.5in max-width page layout
- Print-optimized `@page` margins
- Page break avoidance on headings, tables, and code blocks
- Clean sans-serif typography at 11pt
- Light table header backgrounds that won't waste ink

The HTML title is pulled from the first `# heading` in the file, falling back to the filename.

## What It Handles

### Block elements

| Element | Syntax |
|---------|--------|
| Headings | `# h1` through `###### h6` (with anchor IDs) |
| Paragraphs | Plain text separated by blank lines |
| Fenced code blocks | ` ``` ` or `~~~` with optional language tag |
| Tables | GFM pipe tables with header row |
| Split tables | Table rows separated by code blocks (no repeated header) |
| Blockquotes | `> text` (nested content supported) |
| Unordered lists | `- `, `* `, `+ ` markers (nested) |
| Ordered lists | `1. ` or `1) ` markers (nested) |
| Task lists | `- [ ] unchecked` and `- [x] checked` |
| Horizontal rules | `---`, `***`, `___` |

### Inline elements

| Element | Syntax |
|---------|--------|
| Bold | `**text**` or `__text__` |
| Italic | `*text*` or `_text_` |
| Bold italic | `***text***` |
| Inline code | `` `code` `` |
| Strikethrough | `~~text~~` |
| Links | `[text](url)` |
| Images | `![alt](url)` |

Bold and italic work correctly even when they wrap around inline code spans (e.g., `` **bold with `code` inside** ``).

### Automatic file tree detection

Code blocks containing Unicode box-drawing characters (`├`, `└`, `│`, `─`) are rendered with tighter line spacing, optimized for directory tree diagrams.

## How It Works

The script is a single-pass markdown-to-HTML converter in ~400 lines of Python. No AST, no intermediate representation — it reads lines top to bottom and emits HTML directly.

### Block parsing

The main loop in `md_to_html()` checks each line against block-level patterns in priority order:

1. Fenced code blocks (consumes lines until closing fence)
2. Headings
3. Horizontal rules
4. Tables with header + separator row
5. Orphan table rows (continuation rows after a code block interruption)
6. Blockquotes (recursively parses inner content)
7. Unordered and ordered lists (with nesting via indentation)
8. Blank lines (skipped)
9. Paragraphs (fallback — consumes lines until a block element starts)

### Inline parsing

Inline formatting uses a single compiled regex (`_INLINE_RE`) that tokenizes in one left-to-right pass. This avoids catastrophic backtracking that occurs when `**bold**` markers are split across regex substitution passes — a common pitfall when bold spans contain backtick code spans.

The regex matches tokens in this priority: code spans, images, links, bold+italic, bold, italic, strikethrough. Unmatched text passes through as-is. Bold and italic content is processed recursively to handle nesting.

### Table cell splitting

Table rows are split on `|` delimiters using a character-by-character walker (`split_table_row()`) that tracks backtick state. This prevents `|` characters inside inline code from being treated as cell boundaries.

## Requirements

Python 3.6+. Nothing else.

## License

MIT
