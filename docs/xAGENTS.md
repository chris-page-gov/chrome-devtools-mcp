# Repository Guidelines

## Project Structure & Module Organization

- `src/`: JavaScript/Node.js source code (GraphQL Mesh wrapper, search logic).
- `mcp/`: MCP server and configuration (`server.mjs`, `mcp.json`).
- `test/`: Test suite (`test.mjs`).
- `docs/`: Documentation and instructions.
- Root configs: `mesh.config.yml`, `openlibrary.yaml`, `package.json`.

## Build, Test, and Development Commands

- Install dependencies: `npm install`
- For any Python package management after the initial devcontainer build, use `uv` (preferred) instead of `pip`.
- Start GraphQL Mesh: `npm run mesh:dev`
- Start MCP server: `npm run mcp:server`
- Run tests: `npm test`

## Coding Style & Naming Conventions

- Node.js 18+, ES modules (`.mjs`), 2-space indent, LF endings; max line length 100.
- Use `cross-fetch` for HTTP requests.
- Naming: modules and functions `camelCase`, constants `UPPER_SNAKE_CASE`.
- Source code lives under `src/`.

## Testing Guidelines

- Framework: Node.js built-in `assert` for tests.
- Name tests clearly; prefer small, deterministic unit tests.
- Keep/raise coverage when changing behavior; add tests for regressions.

## Commit & Pull Request Guidelines

- Follow Conventional Commits: `type(scope): subject`.
  Types: feat, fix, docs, style, refactor, perf, test, chore, build, ci.
- PRs: include clear description, linked issues, and before/after notes. For API/tool changes, show example command/output.
- Run tests locally before opening a PR.

## Docs & Changelog Sync

- Always update documentation for user-visible changes: new commands, API/tool changes, or config updates.
- Keep docs current when code changes:
  - `README.md`: quick start, usage examples, new commands.
  - `COPILOT-INSTRUCTIONS.md`: agent and AI assistant guidelines.
- Do not bump version; maintainers will cut releases. Leave changes under [Unreleased] when using a changelog.

## Security & Configuration Tips

- Do not commit secrets or local environment files; `.env*` is gitignored.
- When changing endpoints, prefer environment variables (see `.env.example`).
- Use a devcontainer based on standard VS Code devcontainer images, with required features and environment variables read from the host (prefer this over .env files).
- For Python, use `uv` for package management after the initial build (which may use `pip` for simplicity).
- Large data or logs should be gitignored.

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
```

## Devcontainer Editing Guidance
To keep the development container reliable and avoid permission errors:

- Target the desired Node.js major version via the image tag instead of custom install scripts.
- Keep postCreateCommand lean (npm install only). Do not add corepack enable unless you are certain the container runs as root;<vscode_annotation details='%5B%7B%22title%22%3A%22hardcoded-credentials%22%2C%22description%22%3A%22Embedding%20credentials%20in%20source%20code%20risks%20unauthorized%20access%22%7D%5D'> it</vscode_annotation> will fail with EACCES when it tries to create symlinks.
- For yarn / pnpm usage, prefer an explicit global install (npm install -g pnpm) or a devcontainer feature, rather than relying on Corepack shims.
- Use postStartCommand only for non-critical informational output (printing versions, etc.).
- Avoid embedding global npm upgrades automatically; document them instead. Automatic upgrades can cause unexpected drift in CI or between contributor environments.
- After changing .devcontainer/devcontainer.json, validate JSON quickly:

```bash
  node -e "JSON.parse(require('fs').readFileSync('.devcontainer/devcontainer.json','utf8'))"
```

- Keep formatting compact and avoid arbitrary line wraps to minimize accidental corruption by automated tooling.

Minimal Template

```json
{
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "postCreateCommand": "npm install",
  "postStartCommand": "echo 'Dev container ready: Node $(node -v), npm $(npm -v)'"
}
```

## Agent-Specific Notes

- Make minimal, focused changes; do not rename files or APIs without discussion.
- Obey these guidelines for any files you touch and update docs/tests alongside code changes.