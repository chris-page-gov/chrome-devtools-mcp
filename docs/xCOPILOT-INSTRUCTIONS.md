# Copilot Instructions

These instructions help AI assistants contribute effectively and safely. AGENTS.md contains Repository Guidelines

## Project Purpose

Wrap the public OpenLibrary REST search as GraphQL using GraphQL Mesh, then expose a single safe tool over MCP for AI assistants. The goal is to provide a reproducible, open-source demo for searching OpenLibrary via GraphQL and MCP.

## Tech Stack

- use a devcontainer
- Node.js 18.17+
- GraphQL Mesh for OpenAPI-to-GraphQL translation
- MCP server for tool exposure
- Testing: Node.js built-in `assert`
- HTTP: `cross-fetch`

## Conventions

- Source code in `src/` (ES modules, `.mjs`)
- MCP server and config in `mcp/`
- Tests in `test/` mirroring module structure
- Keep functions small; prefer pure, side-effect free logic where possible
- Use JSDoc for public functions (1–2 line summary + args/returns if non-trivial)
- Keep line length <= 100
- Use 2-space indent, LF endings; no hard tabs
- Markdown style: first line must be a single H1, increment heading levels by 1, fenced code blocks MUST specify a language (`bash`, `js`, `json`, etc.)
- No trailing whitespace; lists should use `-` (preferred) or numbered lists
- For nested lists indent by 2 spaces (never tabs)

## Dependency Management

- Add runtime dependencies to `package.json` and run `npm install`
- Dev-only dependencies should be added with `--save-dev`
- For any Python package management after the initial devcontainer build, use `uv` (preferred) instead of `pip`.

## Devcontainer Best Practices

When editing `.devcontainer/devcontainer.json` or adding setup commands:

- Use an official base image with the required major Node.js version. This project currently targets Node 20 (update the `image` tag; do **not** mix manual Node installs with the image's built-in version).
- Keep `postCreateCommand` minimal and idempotent. Prefer simply: `"postCreateCommand": "npm install"`.
- Avoid `corepack enable` in `postCreateCommand` under a non-root user; it attempts to create symlinks in `/usr/local/bin` and commonly fails with `EACCES`. If you need additional package managers (pnpm, yarn), install them explicitly via `npm install -g` or add a feature that provides them.
- Use `postStartCommand` only for informational echoes (e.g. printing Node and npm versions) – avoid gating logic or environment warnings that may confuse automation.
- Validate JSON after edits (e.g. `node -e "JSON.parse(require('fs').readFileSync('.devcontainer/devcontainer.json','utf8'))"`).
- Do not embed global tool upgrades (like `npm install -g npm@latest`) in `postCreateCommand`; perform them manually if required or document them.
- Keep the file formatting stable (no hard wraps mid-key) to reduce churn in diffs and prevent malformed writes by automation.

### Example Minimal devcontainer.json

```jsonc
{
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "postCreateCommand": "npm install",
  "postStartCommand": "echo 'Dev container ready: Node $(node -v), npm $(npm -v)'"
}


## Testing & Quality Gates

1. Run `npm test` (smoke test)
2. Run linter if added (e.g., `eslint .`)
3. Markdown lint: ensure no hard tabs and proper fenced code languages

## Commit / PR Guidelines

- Small, focused commits; imperative tense in subject ("Add OpenLibrary search tool")
- Include reasoning or tradeoffs in commit body when non-obvious
- Update documentation for user-visible changes
- Before committing, verify: no tab characters (`git grep $'\t' || true` should return nothing you authored), and tests pass

## Adding Features

1. Start with a failing test describing behavior
2. Implement minimal typed code to pass tests
3. Refactor while keeping coverage
4. Update docs
5. Implement fully working code, including remote calls, adding tests to demonstrate compliance with requirements.
6. Re-run a tab scan on changed markdown files if any were edited and use Lint where available to check for style issues.

## Non-Goals (avoid implementing unless discussed)

- Full-text indexing of book contents
- Complex distributed crawling
- Persisting large blobs outside OpenLibrary API

## Security / Safety

- Never execute untrusted shell input
- Avoid following symlinks outside the root path without explicit opt-in

## Future Ideas (Backlog Hints)

- Add more fields to the OpenLibrary schema and expose via Mesh
- CLI entry point for search
- Caching for repeated queries
- Add automated markdown lint and a pre-commit hook to enforce no hard tabs

## How to Ask for Clarification

If requirements are ambiguous, propose 1–2 concrete interpretations and proceed with the most reasonable after a short wait for feedback.

---

AI assistants: adhere to these instructions. When unsure, ask or create a small draft PR for review.
