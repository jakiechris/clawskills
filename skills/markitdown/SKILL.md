---
name: markitdown
description: Convert Office documents, PDFs, images, audio and web pages to Markdown with Microsoft's MarkItDown MCP server. Use when an agent needs to read a local or remote file quickly.
---

# MarkItDown Skill

## Overview

[MarkItDown](https://github.com/microsoft/markitdown) is Microsoft's converter for turning
documents into Markdown that an LLM can read. `markitdown-mcp` wraps it in an MCP server with
a single tool, `convert_to_markdown(uri)`.

Use this skill for fast, read-only extraction. It is the quickest way to get Office, PDF and
web content as Markdown. For scanned PDFs, table structure or OCR, use the `docling` skill
instead; for cell-level Excel work, use the `excel` skill.

## Supported inputs

| Input | Formats |
| --- | --- |
| Office | `.docx`, `.xlsx`, `.pptx` |
| Documents | `.pdf` (text layer only), `.epub`, `.html`, `.htm`, `.ipynb`, `.msg`, `.zip` |
| Data/text | `.csv`, `.json`, `.xml`, `.txt`, `.md` |
| Media | Images (`.jpg`, `.jpeg`, `.png`), audio (`.mp3`, `.wav`, `.m4a`) |
| Web | `http:`/`https:` URLs, YouTube links, Wikipedia pages, RSS/Atom feeds |

Images are read for EXIF/metadata and, when an LLM client is configured, an optional
description. MarkItDown does not OCR images on its own; a photo of text comes back empty.

## Install

```bash
# pip
pip install markitdown-mcp

# or run without installing (uv)
uvx --from markitdown-mcp markitdown-mcp
```

The package declares `markitdown[all]`, so the Office, PDF, audio and web converters are
installed with it.

## Register with mcporter

Add the server to `config/mcporter.json` (project) or `~/.mcporter/mcporter.json` (all projects):

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "uvx",
      "args": ["--from", "markitdown-mcp", "markitdown-mcp"]
    }
  }
}
```

If you installed it with pip, a shorter definition works:

```json
{ "mcpServers": { "markitdown": { "command": "markitdown-mcp" } } }
```

Check that mcporter can see the tool:

```bash
mcporter list markitdown
# function convert_to_markdown(uri: string): convert_to_markdownOutput;
```

## Usage

Always pass a URI. For local files, use an absolute `file://` URI:

```bash
# Word
mcporter call markitdown.convert_to_markdown uri="file:///home/me/docs/report.docx"

# Excel (each sheet becomes a Markdown section)
mcporter call markitdown.convert_to_markdown uri="file:///home/me/docs/sales.xlsx"

# PowerPoint (one section per slide)
mcporter call markitdown.convert_to_markdown uri="file:///home/me/docs/deck.pptx"

# Text-based PDF
mcporter call markitdown.convert_to_markdown uri="file:///home/me/docs/paper.pdf"

# Remote file or web page
mcporter call markitdown.convert_to_markdown uri="https://example.com/report.pdf"
mcporter call markitdown.convert_to_markdown uri="https://example.com/article.html"
```

Windows paths look like `uri="file:///C:/Users/me/report.docx"`. Inline content can be passed
as a `data:` URI, which is handy for small text blobs.

A `file://` conversion of a Word document returns, for example:

```
# Quarterly Report

This document summarizes the Q3 results for the parsing skill demo.

|  |  |
| --- | --- |
| Region | Revenue |
| APAC | 128000 |
```

An `.xlsx` file comes back as one Markdown table per sheet, prefixed with the sheet name
(`## Sales`). A `.pptx` file comes back with `<!-- Slide number: 1 -->` markers and the slide
text. A text PDF keeps paragraph breaks and form feeds between pages.

## Notes and limits

- **No OCR.** Scanned PDFs and photos of text produce little or no output. Use `docling` with
  `DOCLING_MCP_DO_OCR=true` for those.
- **Image metadata only.** A PNG usually converts to an empty string unless `exiftool` is
  installed or an LLM caption client is configured.
- **Large files.** Conversions above the default 60-second mcporter call timeout fail. Pass
  `--timeout 120000` (milliseconds) for big PDFs or workbooks.
- **Plugins.** Third-party MarkItDown plugins are off by default. Set
  `MARKITDOWN_ENABLE_PLUGINS=true` in the server's `env` to enable them.
- **Security.** With STDIO transport the server can read any file the user can read. Keep the
  optional HTTP mode (`--http`) bound to `127.0.0.1`; it has no authentication.

## Troubleshooting

| Symptom | Cause / fix |
| --- | --- |
| `UnsupportedFormatException` | The extension is unknown. Check the file type; convert exotic formats first. |
| Empty output for a PDF | The PDF has no text layer (a scan). Use `docling` with OCR. |
| Empty output for an image | No EXIF data and no LLM caption configured. Use `docling` for OCR. |
| `Could not read the resource` | Use an absolute `file://` URI, or check file permissions. |
| Conversion times out | Pass `--timeout 120000` to `mcporter call`, or raise the client timeout. |

## Resources

- MarkItDown: https://github.com/microsoft/markitdown
- MarkItDown MCP package: https://pypi.org/project/markitdown-mcp/
- mcporter configuration: https://mcporter.sh
