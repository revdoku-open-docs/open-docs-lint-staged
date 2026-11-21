---
title: JavaScript configuration
description: Use functions in a lint-staged configuration to build commands from the staged file list, run a command without file names, or run JavaScript code as a task.
---

A JavaScript or TypeScript configuration file can contain functions. With a function, you decide which command runs and which file paths it gets. See [Configuration](/configuration) for the file names.

There are three forms:

| Form | The function gets | The function returns |
| --- | --- | --- |
| The whole configuration is a function | All staged files | A command or an array of commands |
| A glob pattern has a function as its value | The staged files that match the pattern | A command or an array of commands |
| A glob pattern has an object with `title` and `task` | The staged files that match the pattern | Nothing. The function is the task. |

The file paths are absolute, unless you use `--relative`. A command that a function returns is complete: lint-staged does not add file paths to it.

## Run a command without file names

Some tools must check the full project, for example the TypeScript compiler. Return the command from a function that ignores its argument:

```js
// lint-staged.config.js
export default {
  '*.{ts,tsx}': () => 'tsc --noEmit',
}
```

The command runs once if one or more `.ts` or `.tsx` files are staged. See [TypeScript type checks](/guides/typescript-tsc) for the reason.

## Build the command from the file list

```js
// lint-staged.config.js
export default {
  // Lint the full project when many files change, else only the staged files
  '*.js': (files) => (files.length > 10 ? 'eslint .' : `eslint ${files.join(' ')}`),

  // Run Prettier once for each file, with the path in quotes
  '*.md': (files) => files.map((file) => `prettier --write '${file}'`),
}
```

Paths can contain spaces. If you join paths into one command, put each path in quotes.

## Ignore files from a match

Filter the list before you build the command. This example uses picomatch, which lint-staged also uses:

```js
// lint-staged.config.js
import picomatch from 'picomatch'

const isFixture = picomatch('**/fixtures/**')

export default {
  '*.js': (files) => {
    const kept = files.filter((file) => !isFixture(file))
    return kept.length ? `eslint ${kept.join(' ')}` : []
  },
}
```

An empty array runs no command. Install picomatch as a dev dependency if you import it.

## Use relative paths

Some tools need paths relative to the project folder:

```js
// lint-staged.config.js
import path from 'node:path'

export default {
  '*.ts': (files) => {
    const relative = files.map((file) => path.relative(process.cwd(), file))
    return `my-linter --files ${relative.join(' ')}`
  },
}
```

The `--relative` flag gives the same result for all tasks.

## Return more than one command

A function can return an array. The commands run one after the other. An array in the array runs at the same time (since lint-staged 17.6.0, also when a function returns it):

```js
// lint-staged.config.js
export default {
  '*.js': () => [['eslint --max-warnings=0 .', 'prettier --check .']],
}
```

## Make the whole configuration a function

The function gets all staged files. Use it when one decision depends on files of different types:

```js
// lint-staged.config.js
import picomatch from 'picomatch'

const isScript = picomatch('**/*.{js,ts}')
const isDoc = picomatch('**/*.md')

export default (files) => {
  const scripts = files.filter((file) => isScript(file))
  const docs = files.filter((file) => isDoc(file))
  const commands = []
  if (scripts.length) commands.push(`eslint ${scripts.join(' ')}`)
  if (docs.length) commands.push(`prettier --check ${docs.join(' ')}`)
  return commands
}
```

## Run JavaScript code as a task

An object with `title` and `task` runs a function in the lint-staged process. No command starts. To fail, the function throws an error or returns a rejected promise.

```js
// lint-staged.config.js
import { defineConfig } from 'lint-staged/config'

export default defineConfig({
  '*': {
    title: 'Block .env files',
    task: async (files, { log }) => {
      const envFiles = files.filter((file) => /(^|\/)\.env(\.|$)/.test(file))
      if (envFiles.length > 0) {
        log('Do not commit these files: %s', envFiles.join(', '))
        throw new Error('.env files are staged')
      }
    },
  },
})
```

The output:

```text
✖ Block .env files

✖ Failed to run tasks for staged files!
...
✖ Block .env files:
Do not commit these files: /home/me/project/.env.local

Error: .env files are staged
```

The `log` function was added in lint-staged 17.6.0. It uses the format of Node.js `util.format()`, and each call writes one line. lint-staged shows the log only when the task fails, or when you use `--verbose`.

lint-staged runs a task function once, with all matching files. It does not divide the file list into parts for task functions (since 17.2.0).

## Share settings between files

A configuration file is a module, so it can import another one. For an example, see [Monorepos](/guides/monorepo#share-settings).
