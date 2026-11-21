---
title: Monorepos
description: Use lint-staged in a repository with many packages - one configuration file in each package, shared settings, running one package with --cwd and the rule that configuration files never merge.
---

A monorepo is one Git repository with many packages. Install lint-staged and the Git hook once, in the root of the repository. Then add a configuration file to each package that needs its own tasks.

```text
my-monorepo/
├── .husky/pre-commit          npx lint-staged
├── package.json               lint-staged is a dev dependency here
├── lint-staged.config.js      tasks for files in the root
└── packages/
    ├── web/
    │   └── lint-staged.config.js
    └── api/
        └── .lintstagedrc.json
```

## Which configuration a file uses

For each staged file, lint-staged uses the closest configuration file in the folder of the file or in a parent folder. It runs the tasks of that configuration in the folder of the configuration file. Thus each package can use the tool versions from its own `node_modules`.

lint-staged runs the tasks of all packages in one process. This is faster than one lint-staged process for each package, because the Git operations that lock the repository happen only once.

## Configuration files do not merge

A staged file uses only its closest configuration file. If that file has no pattern that matches, no task runs for the file. lint-staged does not use the root configuration as a fallback.

An example: the root configuration formats Markdown, and `packages/api/.lintstagedrc.json` has only a pattern for JavaScript.

```js
// lint-staged.config.js (root)
export default {
  '*.md': 'prettier --write',
}
```

```json
{ "*.js": "eslint" }
```

If you stage `packages/api/README.md`, Prettier does not run on it. The closest configuration is the one in `packages/api`, and it has no Markdown pattern.

## Share settings

To use the root tasks in a package too, import the root configuration into the configuration of the package. This works only with JavaScript or TypeScript files:

```js
// packages/web/lint-staged.config.js
import base from '../../lint-staged.config.js'

export default {
  ...base,
  '*.{js,ts}': 'eslint --max-warnings=0 --no-warn-ignored',
}
```

Files in `packages/web` now get the Markdown task from the root and the ESLint task.

## Run one package

To run lint-staged for one package only, use `--cwd`:

```bash
npx lint-staged --cwd packages/web
```

lint-staged then uses the configuration of `packages/web` and the staged files in that folder. Files in other packages are not checked.

To run all tasks in the root folder of the repository, not in the folder of each configuration file, use `--cwd .`.

## Where lint-staged finds the tools

A command such as `eslint` comes from `node_modules/.bin`. lint-staged looks for it first in the folder of the configuration file, then in the parent folders. This was broken in lint-staged 16.3.0 to 17.4.x and fixed in 17.5.0. If an older version uses the wrong tool version in a package, update lint-staged.
