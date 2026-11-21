---
title: Node.js API
description: Run lint-staged from your own Node.js code with the lintStaged function - its options, the custom logger, the return value and the errors.
---

You can run lint-staged from a Node.js script, for example in your own Git hook tool or a release script.

```js
import lintStaged from 'lint-staged'

try {
  const passed = await lintStaged({ concurrent: 2, verbose: true })
  process.exitCode = passed ? 0 : 1
} catch (error) {
  // For example, the installed Git is older than 2.32.0
  console.error(error)
  process.exitCode = 1
}
```

lint-staged is an ES module. Use `import`, or `await import('lint-staged')` in CommonJS code.

## Signature

```ts
lintStaged(options?: Options, logger?: Logger): Promise<boolean>
```

| Result | Meaning |
| --- | --- |
| Resolves to `true` | All tasks passed |
| Resolves to `false` | A task failed, an empty commit was stopped, or `failOnChanges` found changes. Also when lint-staged found no valid configuration, for example because a configuration file has a syntax error. lint-staged already wrote the reason to the logger. |
| Rejects | Other errors, for example a Git version older than 2.32.0 |

## Options

The options have the same names as the [CLI flags](/cli), in camel case. The CLI and the API use the same parser since lint-staged 17.4.0, so the defaults are the same.

| Option | Type | Default | CLI flag |
| --- | --- | --- | --- |
| `all` | boolean | `false` | `--all` |
| `allowEmpty` | boolean | `false` (`true` with `all`) | `--allow-empty` |
| `color` | boolean | detected | none |
| `concurrent` | boolean or number | `true` | `--concurrent` |
| `config` | object | none | none |
| `configPath` | string | none | `--config` |
| `continueOnError` | boolean | `false` | `--continue-on-error` |
| `cwd` | string | `process.cwd()` | `--cwd` |
| `debug` | boolean | `false` | `--debug` |
| `diff` | string | none | `--diff` |
| `diffFilter` | string | `"ACMR"` | `--diff-filter` |
| `failOnChanges` | boolean | `false` | `--fail-on-changes` |
| `hideAll` | boolean | `false` | `--hide-all` |
| `hidePartiallyStaged` | boolean | `true` | `--no-hide-partially-staged` |
| `hideUnstaged` | boolean | `false` | `--hide-unstaged` |
| `maxArgLength` | number | platform limit | `--max-arg-length` |
| `quiet` | boolean | `false` | `--quiet` |
| `relative` | boolean | `false` | `--relative` |
| `revert` | boolean | `true` (`false` with `failOnChanges` or no stash) | `--no-revert` |
| `stash` | boolean | `true` (`false` with `all` or `diff`) | `--no-stash` |
| `verbose` | boolean | `false` | `--verbose` |

Two options exist only in the API:

- `config` gives the configuration as an object. lint-staged then does not look for configuration files.
- `color` turns colors in the output on or off. By default, lint-staged detects color support, and it obeys the `FORCE_COLOR` and `NO_COLOR` environment variables.

```js
const passed = await lintStaged({
  config: { '*.js': 'eslint --max-warnings=0' },
  color: false,
})
```

## Custom logger

The second argument replaces `console`. It must have the functions `log`, `warn`, `error` and `debug`. Use it to collect the output, for example to show it in your own tool:

```js
import lintStaged from 'lint-staged'

const lines = []
const logger = {
  log: (...args) => lines.push(args.join(' ')),
  warn: (...args) => lines.push(args.join(' ')),
  error: (...args) => lines.push(args.join(' ')),
  debug: () => {},
}

const passed = await lintStaged({ quiet: true }, logger)
if (!passed) console.log(lines.join('\n'))
```

## TypeScript types

The package includes types. Import them with `import type`:

```ts
import lintStaged, { type Options, type Configuration } from 'lint-staged'

const config: Configuration = { '*.ts': 'eslint' }
const options: Options = { config, concurrent: false }
await lintStaged(options)
```
