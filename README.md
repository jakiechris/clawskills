# Clawskills

## Project Overview

Clawskills is a skills library for agentic agents such as OpenClaw, Claude Code, and OpenCode, with a primary focus on improving OpenClaw's capabilities. Because OpenClaw does not natively support MCP servers, this repository uses the mcporter tool to convert valuable MCP servers into explicit, well-documented "skills." Each skill explains how to call the MCP server endpoints via mcporter commands so agents can use them reliably.

OpenClaw previously included a generic, built-in mcporter skill that aimed to bridge all MCP servers with a one-size-fits-all adapter. After many attempts and optimizations, that generic approach proved unreliable. Clawskills follows a case-by-case conversion strategy: one MCP server → one dedicated skill, optimized per server.

---

## Why case-by-case MCP → skill?

- Generic adapters are brittle: different MCP servers expose different schemas, behaviors, and edge cases. A single adapter cannot reliably cover them all.
- The OpenClaw built-in generic mcporter skill has shown poor reliability despite repeated optimization efforts.
- Converting MCP servers individually allows us to:
  - tailor commands and examples to real-world usage,
  - handle server-specific quirks,
  - provide more reliable, battle-tested skills for agentic agents.

---

## Skills Catalog

### chrome-devtools

- **Name:** chrome-devtools  
- **Source:** chrome-devtools-mcp (converted via mcporter)  
- **Purpose:** Browser automation and web interaction (navigation, DOM snapshots, form filling, clicks, screenshots, JS evaluation, network inspection, performance tracing, Lighthouse audits, file upload, dialog handling, etc.)  
- **Description:** This skill wraps the Chrome DevTools MCP server and exposes its capabilities as a skill via mcporter commands. It is intended for high-reliability browser automation tasks, RPA, scraping, and end-to-end interactions where precise element access and robust network/performance inspection are required.

Advantages (why choose chrome-devtools):
- More accurate element access and DOM inspection.
- More reliable form filling and interaction by UID.
- Richer network and performance inspection (traces, Lighthouse).
- Deeper access to DevTools protocol features than typical high-level browser libraries.

Quick comparison (high-level):
- Element accuracy: chrome-devtools > Playwright > OpenClaw built-in  
- Form reliability: chrome-devtools ≥ Playwright > OpenClaw built-in  
- Network & performance inspection: chrome-devtools >> Playwright > OpenClaw built-in

See full usage and examples in: `skills/chrome-devtools/SKILL.md` (contains detailed mcporter command examples).

### markitdown

- **Name:** markitdown
- **Source:** MarkItDown MCP (microsoft/markitdown)
- **Purpose:** Convert Office documents, PDFs, images, audio and web pages to Markdown.
- **Description:** A small MCP server around Microsoft's MarkItDown converter. It exposes one tool, `convert_to_markdown(uri)`, and accepts `file:`, `http:`, `https:` and `data:` URIs. Word, Excel, PowerPoint, text PDFs, HTML, CSV/JSON/XML, EPUB, ZIP, audio and YouTube links all come back as Markdown that an LLM can read directly.

Advantages (why choose markitdown):
- One tool, broad format coverage, minimal setup.
- Office files become clean Markdown tables and headings, one section per sheet or slide.
- Good default for "read this file and summarize it".

Limits:
- No OCR: scanned PDFs and photos of text convert to little or nothing.
- Images return EXIF metadata only (plus an optional LLM caption); use docling for image text.

See full usage and examples in: `skills/markitdown/SKILL.md`.

### docling

- **Name:** docling
- **Source:** Docling MCP (docling-project/docling-mcp)
- **Purpose:** Structured parsing of PDFs, Office files and images with layout, tables and OCR.
- **Description:** Wraps IBM's Docling pipeline in MCP. A file is converted once into a cached `DoclingDocument`; tools then export Markdown, save Markdown plus JSON, search text, or read individual items by anchor. The local pipeline runs OCR and table-structure detection, so scanned PDFs, screenshots and complex tables are handled properly.

Advantages (why choose docling):
- OCR for scans and images.
- Reading order, page numbers and table structure are preserved.
- Page-level export and anchor-based search instead of one large dump.
- Local mode needs no external service; remote mode can reuse a Docling Serve deployment.

Limits:
- First local conversion downloads several hundred MB of models and is slow on CPU.
- Heavier than markitdown for simple, clean files.

See full usage and examples in: `skills/docling/SKILL.md`.

### excel

- **Name:** excel
- **Source:** Excel MCP Server (haris-musa/excel-mcp-server)
- **Purpose:** Read and edit Excel workbooks (`.xlsx`, `.xlsm`, `.xltx`, `.xltm`) without Microsoft Excel.
- **Description:** Built on `openpyxl`, this server exposes 46 tools for sheets, ranges, formulas, tables, charts, PivotTables and CSV import/export. Agents can list sheets, read a specific range as values or formulas, search cells, and write results back atomically.

Advantages (why choose excel):
- Cell-level access instead of a whole-file Markdown dump.
- Formula-aware reads and a built-in calculation engine for files never opened in Excel.
- Read-only mode and folder confinement (`--allow-dir`) for safe parsing.

Limits:
- Spreadsheets only; use markitdown or docling for Word, PowerPoint and PDF.
- Legacy `.xls` is not supported.

See full usage and examples in: `skills/excel/SKILL.md`.

### Which file-parsing skill?

| Task | Skill |
| --- | --- |
| Read a DOCX/PPTX/XLSX/PDF and summarize it | `markitdown` |
| OCR a scanned PDF, screenshot or photo | `docling` |
| Keep table layout and page structure | `docling` |
| Read one Excel range, formula or sheet | `excel` |
| Write data back to an `.xlsx` file | `excel` |

---

## Important Recommendation

We strongly recommend disabling or removing OpenClaw's built-in generic mcporter skill — its effectiveness is poor and it may conflict with purpose-built skills from this repository.

Typical Windows path (example):
~\AppData\Roaming\npm\node_modules\openclaw\skills\mcporter

Recommended actions:
- Disable: rename the folder to `mcporter.disabled` or `mcporter.bak`.  
- Remove: delete the folder if you will rely on case-by-case skills from this repository.

Note: Do this only if you understand the change and its implications for your OpenClaw installation.

---

## Quick Start

1. Install mcporter (required to call MCP servers):
   - npm: `npm install -g mcporter`
2. Install a file-parsing server on demand (Python servers are easiest to run with [uv](https://docs.astral.sh/uv/)):
   - `uvx --from markitdown-mcp markitdown-mcp`
   - `uvx excel-mcp-server stdio`
   - `uvx --from 'docling-mcp[local]' docling-mcp-server --transport stdio`
3. Browse this repository's skills:
   - Example: `skills/chrome-devtools/SKILL.md`
   - File parsing: `skills/markitdown/SKILL.md`, `skills/docling/SKILL.md`, `skills/excel/SKILL.md`
4. Example commands:
   - Open a new page:
     ```
     mcporter call chrome-devtools.new_page url="https://example.com"
     ```
   - Take a snapshot:
     ```
     mcporter call chrome-devtools.take_snapshot
     ```
   - Fill a form field:
     ```
     mcporter call chrome-devtools.fill uid="2_6" value="John"
     ```
   - Convert a Word file to Markdown:
     ```
     mcporter call markitdown.convert_to_markdown uri="file:///home/me/report.docx"
     ```
   - Read a range from an Excel workbook:
     ```
     mcporter call excel.read_range path="q1.xlsx" sheet="Sales" range="A1:D10"
     ```
   - OCR a scanned PDF with Docling:
     ```
     mcporter call docling.convert_document_into_docling_document source="/home/me/scan.pdf"
     ```

---

## Project Structure (example)

- `skills/` — individual skill folders (each contains a SKILL.md with usage and examples)  
- `docs/` — additional documentation (optional)  
- `CONTRIBUTING.md` — guidelines for contributing new skills (optional)

---

## Resources

- Chrome DevTools MCP repository: https://github.com/ChromeDevTools/chrome-devtools-mcp  
- MarkItDown: https://github.com/microsoft/markitdown  
- Docling MCP: https://github.com/docling-project/docling-mcp  
- Excel MCP Server: https://github.com/haris-musa/excel-mcp-server  
- mcporter documentation: http://mcporter.dev  
- See each skill's detailed usage in its SKILL.md under `skills/<skill-name>/`

---

Last updated: 2026-11-01
