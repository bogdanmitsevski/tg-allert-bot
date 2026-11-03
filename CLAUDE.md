# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

This is an unmodified NestJS 10 starter (TypeScript, Express). The name suggests a Telegram alert bot, but nothing bot-related exists yet: there is no Telegram dependency, config handling, or persistence. `README.md` is the stock Nest template and has no project-specific information.

## Commands

```bash
npm install
npm run start:dev     # run with watch mode (listens on port 3000, hardcoded in src/main.ts)
npm run build         # compile to dist/ (the output directory is cleared on each build)
npm run start:prod    # run the compiled build (node dist/main)
npm run lint          # ESLint; runs with --fix, so it modifies files
npm run format        # Prettier write on src/ and test/
npm test              # unit tests
npm run test:e2e      # e2e tests
npm run test:cov      # unit tests with coverage (written to coverage/)
```

Run a subset of tests:

```bash
npx jest src/app.controller.spec.ts        # single unit test file
npx jest -t "should return"                # unit tests matching a name
npm run test:e2e -- app                    # e2e test files matching a pattern
```

## Structure and conventions

- `src/main.ts` bootstraps `AppModule`, the root module, which wires up `AppController` and `AppService`.
- Unit tests are `*.spec.ts` files next to the source in `src/` (Jest config is in `package.json`). E2e tests are `*.e2e-spec.ts` files in `test/` (config in `test/jest-e2e.json`) and use `supertest` against the full `AppModule`.
- TypeScript is not in strict mode: `strictNullChecks` and `noImplicitAny` are off, so the compiler will not catch null or implicit-`any` errors.
- ESLint allows explicit `any`. Prettier uses single quotes and trailing commas.
