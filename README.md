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
2. Browse this repository's skills:
   - Example: `skills/chrome-devtools/SKILL.md`
3. Example commands:
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

---

## Project Structure (example)

- `skills/` — individual skill folders (each contains a SKILL.md with usage and examples)  
- `docs/` — additional documentation (optional)  
- `CONTRIBUTING.md` — guidelines for contributing new skills (optional)

---

## Resources

- Chrome DevTools MCP repository: https://github.com/ChromeDevTools/chrome-devtools-mcp  
- mcporter documentation: http://mcporter.dev  
- See each skill's detailed usage in its SKILL.md under `skills/<skill-name>/`

---

Last updated: 2026-03-07
