---
title: Configuration
description: Where lint-staged looks for its configuration, the search order of the files, the task format, TypeScript configuration files and multiple configuration files.
---

A lint-staged configuration is an object. Each key is a glob pattern, and each value is the task to run on the staged files that match it.

```json
{
  "*.js": "eslint --fix",
  "*.{css,md}": "prettier --write"
}
```

With this configuration, if you stage `src/app.js`, `styles.css` and `README.md`, lint-staged runs:

```text
eslint --fix /home/me/project/src/app.js
prettier --write /home/me/project/styles.css /home/me/project/README.md
```

lint-staged adds the absolute paths of the files to the end of each command. To get relative paths, use `--relative`. See [Filtering files](/globs) for the glob rules.

## Configuration files

lint-staged reads the configuration from one of these files. Use only one of them in each folder. A `package.json` without a `lint-staged` key is not a configuration file.

| File | Format |
| --- | --- |
| `package.json` | The `lint-staged` object |
| `package.yaml`, `package.yml` | The `lint-staged` key (pnpm) |
| `.lintstagedrc` | YAML (JSON is also valid YAML) |
| `.lintstagedrc.json` | JSON |
| `.lintstagedrc.yaml`, `.lintstagedrc.yml` | YAML |
| `.lintstagedrc.mjs`, `lint-staged.config.mjs` | ES module: `export default { ... }` |
| `.lintstagedrc.cjs`, `lint-staged.config.cjs` | CommonJS: `module.exports = { ... }` |
| `.lintstagedrc.js`, `lint-staged.config.js` | ES module if `package.json` has `"type": "module"`, else CommonJS |
| `.lintstagedrc.mts`, `lint-staged.config.mts` | TypeScript, ES module |
| `.lintstagedrc.cts`, `lint-staged.config.cts` | TypeScript, CommonJS |
| `.lintstagedrc.ts`, `lint-staged.config.ts` | TypeScript, module type from `package.json` |

To use a different file, give its path with `--config` (or `-c`). Then lint-staged does not look for other files:

```bash
npx lint-staged --config ./config/lint-staged.json
```

To read a JSON configuration from standard input, use `-` as the path:

```bash
echo '{"*.js": "prettier --write"}' | npx lint-staged --config -
```

### YAML files need the yaml package

Since lint-staged 17, the `yaml` package is an optional dependency, and npm does not always install it. If you use a YAML configuration file, install it:

```bash
npm install --save-dev yaml
```

A `.lintstagedrc` file without an extension is read as YAML. If it contains JSON, rename it to `.lintstagedrc.json`. Then you do not need the `yaml` package.

## Values of a task

| Value | What lint-staged does |
| --- | --- |
| A string, `"eslint --fix"` | Runs the command with the matching file paths added to the end |
| An array of strings | Runs the commands one after the other. See [Tasks and commands](/tasks). |
| An array in an array | Runs the inner commands at the same time |
| A function | Runs the commands that the function returns. The function gets the file paths. See [JavaScript configuration](/js-config). |
| An object with `title` and `task` | Runs a JavaScript function as the task |

Functions and task objects are possible only in JavaScript and TypeScript configuration files.

## TypeScript configuration files

lint-staged has TypeScript types for its configuration. The `defineConfig` helper gives type checks and completion in your editor:

```ts
// lint-staged.config.mts
import { defineConfig } from 'lint-staged/config'

export default defineConfig({
  '*.{js,ts}': ['eslint --fix', 'prettier --write'],
  '*.md': 'prettier --write',
})
```

Node.js 22.18.0 and later run TypeScript files without extra flags. lint-staged 17.6 needs Node.js 22.22.1 or later, so a `.ts`, `.mts` or `.cts` file works without `NODE_OPTIONS`.

Use the `.mts` extension if your `package.json` does not have `"type": "module"`. With a `.ts` file and no module type, Node.js shows the warning `MODULE_TYPELESS_PACKAGE_JSON` on each run.

In a JavaScript file, you can get the same type checks with a JSDoc comment:

```js
/** @type {import('lint-staged').Configuration} */
export default {
  '*.js': 'eslint --fix',
}
```

## More than one configuration file

A project can have configuration files in different folders. For each staged file, lint-staged uses the closest configuration file, and only that file. The tasks run in the folder of that configuration file.

lint-staged does not merge configuration files. If the closest configuration has no glob that matches a staged file, no task runs for that file. See [Monorepos](/guides/monorepo) for an example and for a way to share settings.

## Debug the configuration

To see which configuration files lint-staged found and which commands it runs, use `--debug`:

```bash
npx lint-staged --debug
```

If lint-staged finds no valid configuration, it shows this error:

```text
✖ lint-staged could not find any valid configuration.
```
