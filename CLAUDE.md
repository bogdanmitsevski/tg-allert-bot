# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

This is a NestJS 12 starter (TypeScript, Fastify) that will become Job Matcher: a backend where users upload CVs and get matching vacancies from DOU, Djinni, and Work.ua. `description.md` holds the product description, architecture, data model, API, and the step-by-step implementation plan. Nothing from the plan exists yet: there is no database, auth, S3 storage, Gemini integration, or vacancy collection. `README.md` is the stock Nest template and has no project-specific information.

## Commands

Requires Node 24.9 or later (`.nvmrc`), because Jest can only load the ESM-only Nest 12 packages from that version on.

```bash
npm install           # also points git at the hooks in .githooks/
npm run start:dev     # run with watch mode (listens on 0.0.0.0:3000, hardcoded in src/main.ts)
npm run build         # compile to dist/ (the output directory is cleared on each build)
npm run start:prod    # run the compiled build (node dist/main)
npm run typecheck     # type-check src/ and test/ (Jest does not check types)
npm run lint          # ESLint; runs with --fix, so it modifies files
npm run lint:check    # ESLint without changing files
npm run format        # Prettier write on src/ and test/
npm test              # unit tests
npm run test:e2e      # e2e tests
npm run test:cov      # unit tests with coverage (written to coverage/)
```

Run a subset of tests. A direct `npx jest` call needs the flag that the npm scripts already set:

```bash
NODE_OPTIONS=--experimental-vm-modules npx jest src/app.controller.spec.ts  # single unit test file
NODE_OPTIONS=--experimental-vm-modules npx jest -t "should return"          # unit tests matching a name
npm run test:e2e -- app                                                     # e2e test files matching a pattern
```

## Structure and conventions

- `src/main.ts` bootstraps `AppModule`, the root module, which wires up `AppController` and `AppService`.
- The HTTP layer is Fastify (`@nestjs/platform-fastify`). Express-only packages such as `multer`, `helmet`, or Express middleware do not work: use the Fastify counterparts, for example `FileInterceptor` from `@nestjs/platform-fastify/multipart` and `@fastify/helmet`.
- Unit tests are `*.spec.ts` files next to the source in `src/` (Jest config is in `package.json`). E2e tests are `*.e2e-spec.ts` files in `test/` (config in `test/jest-e2e.json`). They create the full `AppModule` with `FastifyAdapter`, await `getInstance().ready()`, and send requests with `app.inject()` instead of `supertest`.
- The project compiles to CommonJS and loads the ESM-only Nest packages through `require(esm)`. `jest.mock` does not apply to ESM modules, so replace dependencies through Nest DI (`overrideProvider`).
- TypeScript runs in strict mode. `ts-jest` uses `isolatedModules`, so tests do not report type errors: run `npm run typecheck` as well.
- ESLint allows explicit `any`. Prettier uses single quotes and trailing commas.

## Testing external services

- Tests never call real AWS S3, Gemini, or the vacancy sites. Wrap each of them in a Nest provider so tests can replace it.
- Vacancy parsers are tested against saved RSS and HTML samples in `test/fixtures/`.
- Test CVs must be synthetic. Never put a real CV or other personal data in fixtures, logs, or commits.
- Once the plan's first step adds `docker-compose.yml`, integration and e2e tests need its PostgreSQL test database (`DATABASE_URL_TEST`). The user starts it with `docker compose up -d`.

## Agents and safety

- `.claude/agents/` defines `orchestrator`, `coder`, and `tester`. Start the orchestrator with `claude --agent orchestrator`: its `Agent(coder, tester)` restriction only applies when it runs as the main thread.
- Work on a branch. `.githooks/pre-push` blocks direct pushes to `main`, and `git push` asks for confirmation.
- On a fresh clone, run the first `npm install` in a regular terminal. The sandbox cannot write `.git/config`, so `prepare` would silently fail to enable the hooks. `git rev-parse --git-path hooks` must print `.githooks`.
- `.githooks/pre-commit` and `pre-push` run gitleaks with `.gitleaks.toml` and block the action if gitleaks is missing (`brew install gitleaks`). Do not bypass them.
- `.claude/settings.json` runs Bash in a sandbox: `.env` and `secrets/` are unreadable and undeletable, and direct network connections fail. Only npm and GitHub are allowlisted, but a command can still request another host through the sandbox proxy if it is approved.
- The user runs anything that needs `.env`, real AWS or Gemini, the vacancy sites, or Docker in their own terminal.
