# Copilot Instructions

These instructions guide AI assistants (and human contributors using AI) for the Chrome DevTools MCP repository. See `AGENTS.md` for general repository guidelines.

## Project Purpose

Provide a robust Model Context Protocol (MCP) server that grants controlled programmatic access to Chrome DevTools for automation, debugging, performance tracing, and emulation scenarios.

## Tech Stack

- Node.js >= 22.12
- TypeScript (ESM output)
- `puppeteer-core` + optional host Chrome via remote debugging
- `@modelcontextprotocol/sdk` for MCP server runtime
- Tests: Node built‑in test runner + snapshot tests
- Tool formatting utilities (console/network/snapshot formatters)

## High-Level Architecture

1. CLI (`src/cli.ts`) parses runtime flags.
2. `main.ts` registers tools and serializes execution via a global `Mutex`.
3. `McpContext` tracks pages, network, console, emulation state.
4. Each tool mutates state / queries browser then builds an `McpResponse`.
5. Formatting modules serialize complex data (network, accessibility snapshots, console events).

## Running in a Devcontainer

The devcontainer does NOT bundle Chrome. To use tools that require a browser:

1. Start Chrome on the host with remote debugging:
   ```bash
   /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
     --remote-debugging-port=9222 --user-data-dir="$HOME/.chrome-devtools-mcp-profile"
   ```
2. From inside the container, run the MCP server pointing to the host (macOS Docker default host gateway `host.docker.internal`):
   ```bash
   npx chrome-devtools-mcp@latest --browserUrl http://host.docker.internal:9222
   ```
3. Connect your MCP client (VS Code / Copilot Chat / Claude Code / etc.) using the standard config.

If you prefer the container to launch Chrome itself, you must add a Chromium-capable base image with required dependencies and disable sandboxing—this is intentionally out of scope here to keep the container minimal and host-secure.

## Typical Development Flow

```bash
npm install
npm run build
npm test
npm run docs   # refresh tool docs & README blocks
```

## Adding a New Tool (Quick Recipe)

1. Create `src/tools/<name>.ts` exporting a `ToolDefinition`.
2. Define a concise `description` and JSON Schema `inputSchema` (use string/number/boolean/object/array/enums only).
3. Acquire context resources (page, network, snapshot) via `McpContext` helpers.
4. Include desired response sections (`response.includePages()`, etc.).
5. Add tests in `tests/tools/<name>.test.ts`.
6. Run: build, tests, then `npm run docs`.

## Style & Formatting

- Run `npm run check-format` before committing.
- Avoid unrelated reformatting.
- Use descriptive error messages (e.g., "Page not found; run list_pages to inspect open tabs").

## Dependency Discipline

- Prefer zero new dependencies; justify any addition (security & bundle size awareness).
- Use native Node APIs first (`fs`, `URL`, `timers/promises`).

## Performance / Stability Notes

- All tools run serially (global mutex). Do not add parallel operations that mutate shared browser state.
- Use `wait_for` tool rather than bespoke sleeps.
- Reuse selected page; prefer targeted navigation.

## Testing Guidance

- Keep snapshots narrow; if they grow large, refactor formatter to allow field-by-field assertions.
- For new tools: test input validation, success path, and one error path.
- Run `npm run test:only -- <pattern>` during iterative development.

## Chrome Connection Scenarios

| Scenario | Recommendation |
|----------|----------------|
| Local development (host Chrome) | Launch Chrome manually; use `--browserUrl` in container |
| Headless CI (future) | Add a Chromium-enabled base image and pass `--headless` |
| Multiple isolated sessions | Use `--isolated` (temp user-data-dir) |
| Specific Chrome channel | Use `--channel=beta|canary|dev` (cannot combine with `--browserUrl`) |

## Commit & PR Workflow

1. Make focused change + tests.
2. Run: `npm run typecheck && npm run build && npm test && npm run check-format`.
3. Update docs if user-facing behavior changes.
4. Use Conventional Commit message.
5. Open PR with summary + rationale; include before/after if formatting or output changed.

## Safety & Guardrails

- Never execute arbitrary JS from user input outside controlled evaluation contexts already provided (the `evaluate_script` tool expects explicit input from the MCP client users).
- Do not add file-system reading/writing tools without approval.
- Avoid logging sensitive URLs or headers (use existing formatters that redact or limit fields where applicable).

## Troubleshooting

| Symptom | Hint |
|---------|------|
| "The browser is already running" error | Use `--isolated` or close existing Chrome with same user-data-dir |
| Tools hang waiting for events | Inspect network conditions / CPU throttling settings; reset via emulation tools |
| No pages listed | Open a new page via `new_page` tool or ensure Chrome started correctly |
| Connection refused to host.docker.internal | Confirm Chrome launched with `--remote-debugging-port=9222` and port not firewalled |

## Asking for Clarification

If a requirement is ambiguous, propose up to two interpretations and proceed with the most conservative one after a brief pause for feedback.

---
AI assistants should follow these instructions to produce safe, maintainable contributions.
