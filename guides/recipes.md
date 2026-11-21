---
title: Recipes
description: Tested lint-staged configurations for Prettier, ESLint, Biome, Stylelint, Vitest, Jest and npm scripts.
---

Each recipe is a configuration for `package.json` or `.lintstagedrc.json`. The recipes were tested on 21 November 2026 with lint-staged 17.6.1, Node.js 22.22, ESLint 10.15, Prettier 3.10, Biome 2.6, Stylelint 18.1 and Vitest 5.1.

Install each tool as a dev dependency before you use its recipe.

## Prettier for all file types that it supports

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

`--ignore-unknown` tells Prettier to skip files that it cannot format, such as images. Without it, Prettier fails on these files.

## ESLint with automatic fixes, then Prettier

```json
{
  "*.{js,jsx,ts,tsx}": ["eslint --fix --max-warnings=0 --no-warn-ignored", "prettier --write"],
  "!(*.{js,jsx,ts,tsx})": "prettier --ignore-unknown --write"
}
```

- `--max-warnings=0` makes warnings fail the commit, not only errors.
- `--no-warn-ignored` stops a warning for staged files that your ESLint configuration ignores. See [ESLint and ignored files](/guides/eslint-ignored-files).
- The second pattern excludes the script files. Thus ESLint and Prettier never edit the same file at the same time. See [Prevent conflicts between tasks](/tasks#prevent-conflicts-between-tasks).

## ESLint check only

To report errors and change nothing:

```json
{
  "*.{js,ts}": "eslint --max-warnings=0 --no-warn-ignored"
}
```

If ESLint finds an error, the commit stops and the output shows the problem:

```text
✖ eslint --max-warnings=0 --no-warn-ignored
  1:7  error  'x' is assigned a value but never used  no-unused-vars
✖ 1 problem (1 error, 0 warnings)
```

## Biome

Biome formats and lints in one tool:

```json
{
  "*": "biome check --write --no-errors-on-unmatched --files-ignore-unknown=true"
}
```

- `--no-errors-on-unmatched` prevents an error when no staged file is a file type that Biome handles.
- `--files-ignore-unknown=true` skips file types that Biome does not know.

## Stylelint

```json
{
  "*.{css,scss}": "stylelint --fix"
}
```

## Tests for the changed code

Vitest finds the test files that import the staged files:

```json
{
  "*.{js,ts}": "vitest related --run"
}
```

`--run` runs the tests once. Without it, Vitest starts in watch mode, and the commit never ends.

With Jest, use `--findRelatedTests`. `--passWithNoTests` prevents a failure when no test imports the staged files:

```json
{
  "*.{js,ts}": "jest --bail --findRelatedTests --passWithNoTests"
}
```

To set an environment variable for the tests, use `cross-env`, because lint-staged does not use a shell:

```json
{
  "*.{js,ts}": "cross-env NODE_ENV=test jest --findRelatedTests --passWithNoTests"
}
```

## Reuse an npm script

```json
{
  "scripts": {
    "lint:files": "eslint --max-warnings=0"
  },
  "lint-staged": {
    "*.js": "npm run lint:files --"
  }
}
```

The `--` passes the file paths to the script. Each task then also starts npm. If speed matters, put the command directly in the lint-staged configuration.

## TypeScript type check

`tsc` must check the full project. Use a function, so that lint-staged does not add file paths:

```js
// lint-staged.config.js
export default {
  '*.{ts,tsx}': () => 'tsc --noEmit',
}
```

See [TypeScript type checks](/guides/typescript-tsc).

## Next.js

Next.js 16 removed the `next lint` command, and `next build` does not lint any more ([upgrade guide](https://nextjs.org/docs/app/guides/upgrading/version-16)). Run ESLint or Biome directly, with a recipe on this page.

## More than one recipe

Combine recipes in one configuration. Patterns for different file types run at the same time:

```json
{
  "*.{js,ts}": ["eslint --fix --max-warnings=0 --no-warn-ignored", "prettier --write"],
  "*.css": ["stylelint --fix", "prettier --write"],
  "*.{json,md,yml}": "prettier --write"
}
```
