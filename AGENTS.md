# Repository Guidelines

This document provides focused guidance for AI agents and human contributors working on the Chrome DevTools MCP server. It adapts the template instructions to this codebase.

## Project Structure

- `src/`: TypeScript source (MCP server entrypoint, tools, formatters, context, browser lifecycle)
- `tests/`: Test suite (Node.js built-in test runner + snapshots)
- `scripts/`: Build & doc generation utilities
- `docs/`: Documentation (`tool-reference.md`, this file, Copilot instructions)
- Root configs: `package.json`, `tsconfig.json`, `eslint.config.mjs`, `server.json`

## Build, Test, Development

- Install deps: `npm install`
- Build: `npm run build`
- Run tests: `npm test`
- Update snapshots: `npm run test:update-snapshots`
- Generate docs: `npm run docs` (builds then updates tool docs & README tool list)
- Start MCP server (dev): `npx chrome-devtools-mcp@latest` (uses published build)
- Start local build: `npm start`
- Debug logs: prefix command with `DEBUG=mcp:*` (already set inside devcontainer)

## Node & Runtime

- Node >= 22.12.0 (devcontainer pins an appropriate 22.x image)
- ES Modules (`"type": "module"`)
- TypeScript source compiled to `build/`

## Coding Conventions

- 2‑space indentation, LF endings, no trailing whitespace
- Keep functions small; extract helpers when cyclomatic complexity grows
- Prefer async/await over promise chains
- Public exported functions: brief JSDoc (summary + params only if non-trivial)
- Avoid broad `any`; use inferred types or explicit interfaces
- Use existing utilities (formatters, pagination, Mutex) before adding dependencies
- Log only when it helps debugging (use the central `logger` debug namespaces)

## Tool Architecture (MCP)

A tool = definition in `src/tools/*.ts` exporting `{name, description, inputSchema?, handler}`.
Registration occurs in `main.ts`, wrapping handlers with a global `Mutex` to serialize access to a single shared browser/context.

When adding a tool:
1. Define schema (JSON Schema draft 2020-12 subset) using primitive/object/enum types only.
2. Validate inputs early; throw user-facing errors with actionable messages.
3. Interact with context via the `Context` interface (`ToolDefinition.ts`).
4. Populate response by calling `response.include*()` methods before returning.
5. Add focused tests under `tests/tools/<tool>.test.ts`.
6. Run `npm run docs` to regenerate tool docs and README section.

## Testing Guidelines

- Use Node test runner (`--test`) – keep tests deterministic
- Snapshots: update only with intentional changes (`test:update-snapshots`)
- Mock minimal surfaces (e.g., network request objects) – prefer real behavior for formatters
- Add regression tests for any bug fix

## Performance & Reliability

- Avoid unnecessary page reloads; reuse selected page
- Use `wait_for` semantics already provided (do not invent new polling unless necessary)
- Leverage `McpContext.waitForEventsAfterAction` after input & navigation actions

## Commit & PR Guidelines

- Conventional Commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `perf:`, `chore:`, `build:`, `ci:`
- Keep commits small & logically grouped
- Always run: build + tests + lint (`npm run typecheck && npm run check-format`)
- Update docs for any user-facing tool / option changes

## Documentation Sync

Run `npm run docs` after adding/modifying tools or CLI options to refresh:
- `docs/tool-reference.md`
- README tools and options auto-generated blocks

## Security & Safety

- Never execute untrusted shell commands from tool inputs
- Do not expose arbitrary file system reads/writes beyond current design
- Avoid leaking sensitive local environment variables in logs
- Validate user-provided URLs with `new URL()` pattern (as CLI does)

## Devcontainer Notes

The devcontainer isolates Node/TypeScript toolchain while allowing Chrome to run on the host. Use the `--browserUrl http://127.0.0.1:9222` flag to connect to a host-launched Chrome (see instructions in `COPILOT-INSTRUCTIONS.md`).

## Adding Browser-Dependent Features

1. Extend `McpContext` with state or behavior (keep state cohesive)
2. Add formatters if serialization complexity grows
3. Write tests that simulate context changes without launching a real browser where possible

## Release Process

Version is bumped via standard release automation (e.g., release-please). Do not manually edit `CHANGELOG.md` except for unreleased notes if maintainers request it.

## Agent-Specific Guidance

- Keep diffs minimal; do not reformat unrelated files
- Always add/adjust tests for behavior changes
- Provide clear error messages; prefer actionable suggestions
- If uncertain about a design choice, open a draft PR with rationale summary

## Quick Checklist Before PR

- [ ] Added/updated tests
- [ ] `npm run build` succeeds
- [ ] `npm test` green
- [ ] `npm run typecheck` passes
- [ ] `npm run check-format` passes
- [ ] Docs regenerated if tools/options changed

---
Adhere to these guidelines to ensure consistent, maintainable contributions.
