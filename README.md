# 🚫💩 lint-staged

[![npm version](https://badge.fury.io/js/lint-staged.svg)](https://badge.fury.io/js/lint-staged)

* == script 
  * allows
    * run tasks (formatters & linters) | staged git files
      * -- passed as -- arguments
      * / -- can be filtered by -- specified glob pattern
    * NOT let commit | your code base
  * by default, 
    * | BEFORE running a task, it creates a `git stash`
      * == backup of the original state
      * Reason: 🧠prevent data loss🧠

* _Examples:_ [video](https://asciinema.org/a/199934)
  ```
  $ git commit
  
  ✔ Backed up original state in git stash (5bda95f)
  ❯ Running tasks for staged files...
    ❯ packages/frontend/.lintstagedrc.json — 1 file
      ↓ *.js — no files [SKIPPED]
      ❯ *.{json,md} — 1 file
        ⠹ prettier --write
    ↓ packages/backend/.lintstagedrc.json — 2 files
      ❯ *.js — 2 files
        ⠼ eslint --fix
      ↓ *.{json,md} — no files [SKIPPED]
  ◼ Applying modifications from tasks...
  ◼ Cleaning up temporary files...
  ```

## Goal

* Code quality tasks (formatters & linters) 
  * should run
    * | BEFORE committing your code, 
      * Reason: 🧠avoid errors & enforce code style🧠
    * | files / will be committed
      * != ALL files

## Related blog posts and talks

- [Introductory Medium post - Andrey Okonetchnikov, 2016](https://medium.com/@okonetchnikov/make-linting-great-again-f3890e1ad6b8#.8qepn2b5l)
- [Running Jest Tests Before Each Git Commit - Ben McCormick, 2017](https://benmccormick.org/2017/02/26/running-jest-tests-before-each-git-commit/)
- [AgentConf presentation - Andrey Okonetchnikov, 2018](https://www.youtube.com/watch?v=-mhY7e-EsC4)
- [SurviveJS interview - Juho Vepsäläinen and Andrey Okonetchnikov, 2018](https://survivejs.com/blog/lint-staged-interview/)
- [Prettier your CSharp with `dotnet-format` and `lint-staged`](https://johnnyreilly.com/2020/12/22/prettier-your-csharp-with-dotnet-format-and-lint-staged)

## How to setup?

1. `npm install --save-dev lint-staged`
2. set up `pre-commit` git hook 
   - ⚠️required to run _lint-staged_ ⚠️
   - [git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
   - ways to configure git hooks
     - [Husky](https://github.com/typicode/husky)
3. install some tools (_Example:_ [ESLint](https://eslint.org) or [Prettier](https://prettier.io))
4. Configure _lint-staged_ / run code checkers & OTHER tasks
   - [Configuration](#configuration)

## Changelog

### Migration

#### v15

- Since `v15.0.0` _lint-staged_ no longer supports Node.js 16. Please upgrade your Node.js version to at least `18.12.0`.

#### v14

- Since `v14.0.0` _lint-staged_ no longer supports Node.js 14. Please upgrade your Node.js version to at least `16.14.0`.

#### v13

- Since `v13.0.0` _lint-staged_ no longer supports Node.js 12. Please upgrade your Node.js version to at least `14.13.1`, or `16.0.0` onward.
- Version `v13.3.0` was incorrectly released including code of version `v14.0.0`. This means the breaking changes of `v14` are also included in `v13.3.0`, the last `v13` version released

#### v12

- Since `v12.0.0` _lint-staged_ is a pure ESM module, so make sure your Node.js version is at least `12.20.0`, `14.13.1`, or `16.0.0`. Read more about ESM modules from the official [Node.js Documentation site here](https://nodejs.org/api/esm.html#introduction).

#### v10

- From `v10.0.0` onwards any new modifications to originally staged files will be automatically added to the commit.
  If your task previously contained a `git add` step, please remove this.
  The automatic behaviour ensures there are less race-conditions,
  since trying to run multiple git operations at the same time usually results in an error.
- From `v10.0.0` onwards, lint-staged uses git stashes to improve speed and provide backups while running.
  Since git stashes require at least an initial commit, you shouldn't run lint-staged in an empty repo.
- From `v10.0.0` onwards, lint-staged requires Node.js version 10.13.0 or later.
- From `v10.0.0` onwards, lint-staged will abort the commit if linter tasks undo all staged changes. To allow creating an empty commit, please use the `--allow-empty` option.

## Command line flags

```
❯ npx lint-staged --help
Usage: lint-staged [options]

Options:
  -V, --version                      output the version number
  --allow-empty                      allow empty commits when tasks revert all staged changes (default: false)
  -p, --concurrent <number|boolean>  the number of tasks to run concurrently, or false for serial (default: true)
  -c, --config [path]                path to configuration file, or - to read from stdin
  --cwd [path]                       run all tasks in specific directory, instead of the current
  -d, --debug                        print additional debug information (default: false)
  --diff [string]                    override the default "--staged" flag of "git diff" to get list of files.
                                     Implies "--no-stash".
  --diff-filter [string]             override the default "--diff-filter=ACMR" flag of "git diff" to get list of
                                     files
  --max-arg-length [number]          maximum length of the command-line argument string (default: 0)
  --no-stash                         disable the backup stash, and do not revert in case of errors. Implies
                                     "--no-hide-partially-staged".
  --no-hide-partially-staged         disable hiding unstaged changes from partially staged files
  -q, --quiet                        disable lint-staged’s own console output (default: false)
  -r, --relative                     pass relative filepaths to tasks (default: false)
  -x, --shell [path]                 skip parsing of tasks for better shell support (default: false)
  -v, --verbose                      show task output even when tasks succeed; by default only failed output is
                                     shown (default: false)
  -h, --help                         display help for command

Any lost modifications can be restored from a git stash:

  > git stash list
  stash@{0}: automatic lint-staged backup
  > git stash apply --index stash@{0}
```

- **`--allow-empty`**
  - TODO: By default, when tasks undo all staged changes, lint-staged will exit with an error and abort the commit. Use this flag to allow creating empty git commits.
- **`--concurrent [number|boolean]`**: Controls the [concurrency of tasks](#task-concurrency) being run by lint-staged. **NOTE**: This does NOT affect the concurrency of subtasks (they will always be run sequentially). Possible values are:
  - `false`: Run all tasks serially
  - `true` (default) : _Infinite_ concurrency. Runs as many tasks in parallel as possible.
  - `{number}`: Run the specified number of tasks in parallel, where `1` is equivalent to `false`.
- **`--config [path]`**: Manually specify a path to a config file or npm package name. Note: when used, lint-staged won't perform the config file search and will print an error if the specified file cannot be found. If '-' is provided as the filename then the config will be read from stdin, allowing piping in the config like `cat my-config.json | npx lint-staged --config -`.
- **`--cwd [path]`**: By default tasks run in the current working directory. Use the `--cwd some/directory` to override this. The path can be absolute or relative to the current working directory.
- **`--debug`**: Run in debug mode. When set, it does the following:
  - uses [debug](https://github.com/visionmedia/debug) internally to log additional information about staged files, commands being executed, location of binaries, etc. Debug logs, which are automatically enabled by passing the flag, can also be enabled by setting the environment variable `$DEBUG` to `lint-staged*`.
  - uses [`verbose` renderer](https://listr2.kilic.dev/renderers/verbose-renderer/) for `listr2`; this causes serial, uncoloured output to the terminal, instead of the default (beautified, dynamic) output.
    (the [`verbose` renderer](https://listr2.kilic.dev/renderers/verbose-renderer/) can also be activated by setting the `TERM=dumb` or `NODE_ENV=test` environment variables)
- **`--diff`**: By default tasks are filtered against all files staged in git, generated from `git diff --staged`. This option allows you to override the `--staged` flag with arbitrary revisions. For example to get a list of changed files between two branches, use `--diff="branch1...branch2"`. You can also read more from about [git diff](https://git-scm.com/docs/git-diff) and [gitrevisions](https://git-scm.com/docs/gitrevisions). This option also implies `--no-stash`.
- **`--diff-filter`**: By default only files that are _added_, _copied_, _modified_, or _renamed_ are included. Use this flag to override the default `ACMR` value with something else: _added_ (`A`), _copied_ (`C`), _deleted_ (`D`), _modified_ (`M`), _renamed_ (`R`), _type changed_ (`T`), _unmerged_ (`U`), _unknown_ (`X`), or _pairing broken_ (`B`). See also the `git diff` docs for [--diff-filter](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203).
- **`--max-arg-length`**: long commands (a lot of files) are automatically split into multiple chunks when it detects the current shell cannot handle them. Use this flag to override the maximum length of the generated command string.
- **`--no-stash`**: By default a backup stash will be created before running the tasks, and all task modifications will be reverted in case of an error. This option will disable creating the stash, and instead leave all modifications in the index when aborting the commit. Can be re-enabled with `--stash`. This option also implies `--no-hide-partially-staged`.
- **`--no-hide-partially-staged`**: By default, unstaged changes from partially staged files will be hidden. This option will disable this behavior and include all unstaged changes in partially staged files. Can be re-enabled with `--hide-partially-staged`
- **`--quiet`**: Supress all CLI output, except from tasks.
- **`--relative`**: Pass filepaths relative to `process.cwd()` (where `lint-staged` runs) to tasks. Default is `false`.
- **`--shell`**: By default task commands will be parsed for speed and security. This has the side-effect that regular shell scripts might not work as expected. You can skip parsing of commands with this option. To use a specific shell, use a path like `--shell "/bin/bash"`.
- **`--verbose`**: Show task output even when tasks succeed. By default only failed output is shown.

## Configuration

* ways to configure
  - | `package.json` OR [`package.yaml`](https://github.com/pnpm/pnpm/pull/1799),
    - `lint-staged` object 
    ```json
    {
      "lint-staged": {
        "*": "your-cmd"
      }
    }
    ```    
  - `.lintstagedrc`
    - `.lintstagedrc.json`
    - `.lintstagedrc.yaml`
    - `.lintstagedrc.yml`
    ```json
    {
      "*": "your-cmd"
    }
    ```
  - `.lintstagedrc.mjs` OR `lint-staged.config.mjs` / ESM format
    - == `export default { lintStageConfiguration }`
  - `.lintstagedrc.cjs` or `lint-staged.config.cjs` / CommonJS format
    - == `module.exports = { lintStageConfiguration }`
  - `lint-staged.config.js` or `.lintstagedrc.js` 
    - / ESM OR CommonJS format -- depending on -- your project's _package.json_ `"type": "module"`
  - Pass a configuration file -- via -- `--config` or `-c` flag

* 💡== object / 💡
  * EACH key == glob pattern -- via using [micromatch](https://github.com/micromatch/micromatch) 
  * EACH value == command to run
  * | JavaScript files,
    * [you can ALSO use a function](#using-js-configuration-files)
  * [| multi-package monorepo?"](#how-to-use-lint-staged-in-a-multi-package-monorepo)

### | JS
* -- via -- JSDoc syntax
  ```js
  /**
   * @filename: lint-staged.config.js
   * @type {import('lint-staged').Configuration}
   */
  export default {
    '*': 'prettier --write',
  }
  ```

### | TypeScript
* | [Node.js v22.6.0](https://github.com/nodejs/node/releases/tag/v22.6.0)
  * if you want to enable Node.js / execute TypeScript files WITHOUT additional configuration -> use `--experimental-strip-types` flag
  ```shell
  export NODE_OPTIONS="--experimental-strip-types"
  
  npx lint-staged --config lint-staged.config.ts
  ```

### Task concurrency

* TODO:
By default _lint-staged_ will run configured tasks concurrently. 
This means that for every glob, all the commands will be started at the same time. 
With the following config, both `eslint` and `prettier` will run at the same time:

```json
{
  "*.ts": "eslint",
  "*.md": "prettier --list-different"
}
```

This is typically not a problem since the globs do not overlap, and the commands do not make changes to the files, but only report possible errors (aborting the git commit). 
If you want to run multiple commands for the same set of files, you can use the array syntax to make sure commands are run in order. 
In the following example, `prettier` will run for both globs, and in addition `eslint` will run for `*.ts` files _after_ it. 
Both sets of commands (for each glob) are still started at the same time (but do not overlap).

```json
{
  "*.ts": ["prettier --list-different", "eslint"],
  "*.md": "prettier --list-different"
}
```

Pay extra attention when the configured globs overlap, and tasks make edits to files. 
For example, in this configuration `prettier` and `eslint` might try to make changes to the same `*.ts` file at the same time, causing a _race condition_:

```json
{
  "*": "prettier --write",
  "*.ts": "eslint --fix"
}
```

You can solve it using the negation pattern and the array syntax:

```json
{
  "!(*.ts)": "prettier --write",
  "*.ts": ["eslint --fix", "prettier --write"]
}
```

Another example in which tasks make edits to files and globs match multiple files but don't overlap:

```json
{
  "*.css": ["stylelint --fix", "prettier --write"],
  "*.{js,jsx}": ["eslint --fix", "prettier --write"],
  "!(*.css|*.js|*.jsx)": ["prettier --write"]
}
```

Or, if necessary, you can limit the concurrency using `--concurrent <number>` or disable it entirely with `--concurrent false`.

## Filtering files

* lint-staged uses [micromatch](https://github.com/micromatch/micromatch) / rules
  - if the glob pattern contains NO slashes (`/`) -> micromatch's `matchBase` option enabled -> globs match a file's basename REGARDLESS directory
    - _Examples:_
      - `"*.js"` == ALL JS files
        - _Examples:_ `/test.js` & `/foo/bar/test.js`
      - `"!(*test).js"` == ALL JS files / EXCEPT TO ending in `test.js`
        - _Example:_ NOT match `foo.test.js`
      - `"!(*.css|*.js)"` == ALL files / EXCEPT TO CSS & JS files
  - if the glob pattern does contain a slash (`/`) -> match for paths
    - _Examples:_
      - `"./*.js"` == ALL JS files | git repo root
        - _Examples:_ 
          - `/test.js`
          - NOT `/foo/bar/test.js`
      - `"foo/**/*.js"` == ALL JS files | `/foo` directory
        - _Examples:_
          - `/foo/bar/test.js`
          - NOT `/test.js`

* how does it work?
  * resolve AUTOMATICALLY the git root / NO configuration needed
  * pick the staged files / present | project directory
  * filter staged files -- via -- specified glob patterns
  * pass absolute paths | tasks -- as -- arguments
    * Reason: 🧠if they're executed | DIFFERENT working directory -> avoid confusion 🧠
      * (== `.git` directory != your `package.json` directory)

### Ignoring files

The concept of `lint-staged` is to run configured linter tasks (or other tasks) on files that are staged in git.
`lint-staged` will always pass a list of all staged files to the task, and ignoring any files should be configured in the task itself.

Consider a project that uses [`prettier`](https://prettier.io/) to keep code format consistent across all files.
The project also stores minified 3rd-party vendor libraries in the `vendor/` directory. 
To keep `prettier` from throwing errors on these files, the vendor directory should be added to prettier's ignore configuration, the `.prettierignore` file. 
Running `npx prettier .` will ignore the entire vendor directory, throwing no errors. 
When `lint-staged` is added to the project and configured to run prettier, all modified and staged files in the vendor directory will be ignored by prettier, even though it receives them as input.

In advanced scenarios, where it is impossible to configure the linter task itself to ignore files, but some staged files should still be ignored by `lint-staged`, it is possible to filter filepaths before passing them to tasks by using the function syntax. 
See [Example: Ignore files from match](#example-ignore-files-from-match).

## What commands are supported?

Supported are any executables installed locally or globally via `npm` as well as any executable from your \$PATH.

> Using globally installed scripts is discouraged, since lint-staged may not work for someone who doesn't have it installed.

`lint-staged` uses [execa](https://github.com/sindresorhus/execa#preferlocal) to locate locally installed scripts. So in your `.lintstagedrc` you can write:

```json
{
  "*.js": "eslint --fix"
}
```

This will result in _lint-staged_ running `eslint --fix file-1.js file-2.js`, when you have staged files `file-1.js`, `file-2.js` and `README.md`.

Pass arguments to your commands separated by space as you would do in the shell. See [examples](#examples) below.

## Running multiple commands in a sequence

You can run multiple commands in a sequence on every glob. To do so, pass an array of commands instead of a single one. This is useful for running autoformatting tools like `eslint --fix` or `stylefmt` but can be used for any arbitrary sequences.

For example:

```json
{
  "*.js": ["eslint", "prettier --write"]
}
```

going to execute `eslint` and if it exits with `0` code, it will execute `prettier --write` on all staged `*.js` files.

This will result in _lint-staged_ running `eslint file-1.js file-2.js`, when you have staged files `file-1.js`, `file-2.js` and `README.md`, and if it passes, `prettier --write file-1.js file-2.js`.

## Using JS configuration files

Writing the configuration file in JavaScript is the most powerful way to configure lint-staged (`lint-staged.config.js`, [similar](https://github.com/okonet/lint-staged#configuration), or passed via `--config`). 
From the configuration file, you can export either a single function or an object.

If the `exports` value is a function, it will receive an array of all staged filenames. 
You can then build your own matchers for the files and return a command string or an array of command strings. 
These strings are considered complete and should include the filename arguments, if wanted.

If the `exports` value is an object, its keys should be glob matches (like in the normal non-js config format). 
The values can either be like in the normal config or individual functions like described above. 
Instead of receiving all matched files, the functions in the exported object will only receive the staged files matching the corresponding glob key.

To summarize, by default _lint-staged_ automatically adds the list of matched staged files to your command, but when building the command using JS functions it is expected to do this manually.
For example:

```js
export default {
  '*.js': (stagedFiles) => [`eslint .`, `prettier --write ${stagedFiles.join(' ')}`],
}
```

This will result in _lint-staged_ first running `eslint .` (matching _all_ files), and if it passes, `prettier --write file-1.js file-2.js`, when you have staged files `file-1.js`, `file-2.js` and `README.md`.

### Function signature

The function can also be async:

```ts
(filenames: string[]) => string | string[] | Promise<string | string[]>
```

### Example: Export a function to build your own matchers

<details>
  <summary>Click to expand</summary>

```js
// lint-staged.config.js
import micromatch from 'micromatch'

export default (allStagedFiles) => {
  const shFiles = micromatch(allStagedFiles, ['**/src/**/*.sh'])
  if (shFiles.length) {
    return `printf '%s\n' "Script files aren't allowed in src directory" >&2`
  }
  const codeFiles = micromatch(allStagedFiles, ['**/*.js', '**/*.ts'])
  const docFiles = micromatch(allStagedFiles, ['**/*.md'])
  return [`eslint ${codeFiles.join(' ')}`, `mdl ${docFiles.join(' ')}`]
}
```

</details>

### Example: Wrap filenames in single quotes and run once per file

<details>
  <summary>Click to expand</summary>

```js
// .lintstagedrc.js
export default {
  '**/*.js?(x)': (filenames) => filenames.map((filename) => `prettier --write '${filename}'`),
}
```

</details>

### Example: Run `tsc` on changes to TypeScript files, but do not pass any filename arguments

<details>
  <summary>Click to expand</summary>

```js
// lint-staged.config.js
export default {
  '**/*.ts?(x)': () => 'tsc -p tsconfig.json --noEmit',
}
```

</details>

### Example: Run ESLint on entire repo if more than 10 staged files

<details>
  <summary>Click to expand</summary>

```js
// .lintstagedrc.js
export default {
  '**/*.js?(x)': (filenames) =>
    filenames.length > 10 ? 'eslint .' : `eslint ${filenames.join(' ')}`,
}
```

</details>

### Example: Use your own globs

<details>
  <summary>Click to expand</summary>

It's better to use the [function-based configuration (seen above)](https://github.com/okonet/lint-staged#example-export-a-function-to-build-your-own-matchers), if your use case is this.

```js
// lint-staged.config.js
import micromatch from 'micromatch'

export default {
  '*': (allFiles) => {
    const codeFiles = micromatch(allFiles, ['**/*.js', '**/*.ts'])
    const docFiles = micromatch(allFiles, ['**/*.md'])
    return [`eslint ${codeFiles.join(' ')}`, `mdl ${docFiles.join(' ')}`]
  },
}
```

</details>

### Example: Ignore files from match

<details>
  <summary>Click to expand</summary>

If for some reason you want to ignore files from the glob match, you can use `micromatch.not()`:

```js
// lint-staged.config.js
import micromatch from 'micromatch'

export default {
  '*.js': (files) => {
    // from `files` filter those _NOT_ matching `*test.js`
    const match = micromatch.not(files, '*test.js')
    return `eslint ${match.join(' ')}`
  },
}
```

Please note that for most cases, globs can achieve the same effect. For the above example, a matching glob would be `!(*test).js`.

</details>

### Example: Use relative paths for commands

<details>
  <summary>Click to expand</summary>

```js
import path from 'path'

export default {
  '*.ts': (absolutePaths) => {
    const cwd = process.cwd()
    const relativePaths = absolutePaths.map((file) => path.relative(cwd, file))
    return `ng lint myProjectName --files ${relativePaths.join(' ')}`
  },
}
```

</details>

## Reformatting the code

Tools like [Prettier](https://prettier.io), ESLint/TSLint, or stylelint can reformat your code according to an appropriate config by running `prettier --write`/`eslint --fix`/`tslint --fix`/`stylelint --fix`. Lint-staged will automatically add any modifications to the commit as long as there are no errors.

```json
{
  "*.js": "prettier --write"
}
```

Prior to version 10, tasks had to manually include `git add` as the final step. This behavior has been integrated into lint-staged itself in order to prevent race conditions with multiple tasks editing the same files. If lint-staged detects `git add` in task configurations, it will show a warning in the console. Please remove `git add` from your configuration after upgrading.

## Examples

All examples assume you've already set up lint-staged in the `package.json` file and [husky](https://github.com/typicode/husky) in its own config file.

```json
{
  "name": "My project",
  "version": "0.1.0",
  "scripts": {
    "my-custom-script": "linter --arg1 --arg2"
  },
  "lint-staged": {}
}
```

In `.husky/pre-commit`

```shell
# .husky/pre-commit

npx lint-staged
```

_Note: we don't pass a path as an argument for the runners. This is important since lint-staged will do this for you._

### ESLint with default parameters for `*.js` and `*.jsx` running as a pre-commit hook

<details>
  <summary>Click to expand</summary>

```json
{
  "*.{js,jsx}": "eslint"
}
```

</details>

### Automatically fix code style with `--fix` and add to commit

<details>
  <summary>Click to expand</summary>

```json
{
  "*.js": "eslint --fix"
}
```

This will run `eslint --fix` and automatically add changes to the commit.

</details>

### Reuse npm script

<details>
  <summary>Click to expand</summary>

If you wish to reuse a npm script defined in your package.json:

```json
{
  "*.js": "npm run my-custom-script --"
}
```

The following is equivalent:

```json
{
  "*.js": "linter --arg1 --arg2"
}
```

</details>

### Use environment variables with task commands

<details>
  <summary>Click to expand</summary>

Task commands _do not_ support the shell convention of expanding environment variables. To enable the convention yourself, use a tool like [`cross-env`](https://github.com/kentcdodds/cross-env).

For example, here is `jest` running on all `.js` files with the `NODE_ENV` variable being set to `"test"`:

```json
{
  "*.js": ["cross-env NODE_ENV=test jest --bail --findRelatedTests"]
}
```

</details>

### Automatically fix code style with `prettier` for any format Prettier supports

<details>
  <summary>Click to expand</summary>

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

</details>

### Automatically fix code style with `prettier` for JavaScript, TypeScript, Markdown, HTML, or CSS

<details>
  <summary>Click to expand</summary>

```json
{
  "*.{js,jsx,ts,tsx,md,html,css}": "prettier --write"
}
```

</details>

### Stylelint for CSS with defaults and for SCSS with SCSS syntax

<details>
  <summary>Click to expand</summary>

```json
{
  "*.css": "stylelint",
  "*.scss": "stylelint --syntax=scss"
}
```

</details>

### Run PostCSS sorting and Stylelint to check

<details>
  <summary>Click to expand</summary>

```json
{
  "*.scss": ["postcss --config path/to/your/config --replace", "stylelint"]
}
```

</details>

### Minify the images

<details>
  <summary>Click to expand</summary>

```json
{
  "*.{png,jpeg,jpg,gif,svg}": "imagemin-lint-staged"
}
```

<details>
  <summary>More about <code>imagemin-lint-staged</code></summary>

[imagemin-lint-staged](https://github.com/tomchentw/imagemin-lint-staged) is a CLI tool designed for lint-staged usage with sensible defaults.

See more on [this blog post](https://medium.com/@tomchentw/imagemin-lint-staged-in-place-minify-the-images-before-adding-to-the-git-repo-5acda0b4c57e) for benefits of this approach.

</details>
</details>

### Typecheck your staged files with flow

<details>
  <summary>Click to expand</summary>

```json
{
  "*.{js,jsx}": "flow focus-check"
}
```

</details>

### Integrate with Next.js

<details>
  <summary>Click to expand</summary>

```js
// .lintstagedrc.js
// See https://nextjs.org/docs/basic-features/eslint#lint-staged for details

const path = require('path')

const buildEslintCommand = (filenames) =>
  `next lint --fix --file ${filenames.map((f) => path.relative(process.cwd(), f)).join(' --file ')}`

module.exports = {
  '*.{js,jsx,ts,tsx}': [buildEslintCommand],
}
```

</details>

## Frequently Asked Questions

### The output of commit hook looks weird (no colors, duplicate lines, verbose output on Windows, …)

<details>
  <summary>Click to expand</summary>

Git 2.36.0 introduced a change to hooks where they were no longer run in the original TTY.
This was fixed in 2.37.0:

https://raw.githubusercontent.com/git/git/master/Documentation/RelNotes/2.37.0.txt

> - In Git 2.36 we revamped the way how hooks are invoked. One change
>   that is end-user visible is that the output of a hook is no longer
>   directly connected to the standard output of "git" that spawns the
>   hook, which was noticed post release. This is getting corrected.
>   (merge [a082345372](https://github.com/git/git/commit/a082345372) ab/hooks-regression-fix later to maint).

If updating Git doesn't help, you can try to manually redirect the output in your Git hook; for example:

```shell
# .husky/pre-commit

if sh -c ": >/dev/tty" >/dev/null 2>/dev/null; then exec >/dev/tty 2>&1; fi

npx lint-staged
```

Source: https://github.com/typicode/husky/issues/968#issuecomment-1176848345

</details>

### Can I use `lint-staged` via node?

<details>
  <summary>Click to expand</summary>

Yes!

```js
import lintStaged from 'lint-staged'

try {
  const success = await lintStaged()
  console.log(success ? 'Linting was successful!' : 'Linting failed!')
} catch (e) {
  // Failed to load configuration
  console.error(e)
}
```

Parameters to `lintStaged` are equivalent to their CLI counterparts:

```js
const success = await lintStaged({
  allowEmpty: false,
  concurrent: true,
  configPath: './path/to/configuration/file',
  cwd: process.cwd(),
  debug: false,
  maxArgLength: null,
  quiet: false,
  relative: false,
  shell: false,
  stash: true,
  verbose: false,
})
```

You can also pass config directly with `config` option:

```js
const success = await lintStaged({
  allowEmpty: false,
  concurrent: true,
  config: { '*.js': 'eslint --fix' },
  cwd: process.cwd(),
  debug: false,
  maxArgLength: null,
  quiet: false,
  relative: false,
  shell: false,
  stash: true,
  verbose: false,
})
```

The `maxArgLength` option configures chunking of tasks into multiple parts that are run one after the other. This is to avoid issues on Windows platforms where the maximum length of the command line argument string is limited to 8192 characters. Lint-staged might generate a very long argument string when there are many staged files. This option is set automatically from the cli, but not via the Node.js API by default.

</details>

### Using with JetBrains IDEs _(WebStorm, PyCharm, IntelliJ IDEA, RubyMine, etc.)_

<details>
  <summary>Click to expand</summary>

_**Update**_: The latest version of JetBrains IDEs now support running hooks as you would expect.

When using the IDE's GUI to commit changes with the `precommit` hook, you might see inconsistencies in the IDE and command line. This is [known issue](https://youtrack.jetbrains.com/issue/IDEA-135454) at JetBrains so if you want this fixed, please vote for it on YouTrack.

Until the issue is resolved in the IDE, you can use the following config to work around it:

husky v1.x

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "post-commit": "git update-index --again"
    }
  }
}
```

husky v0.x

```json
{
  "scripts": {
    "precommit": "lint-staged",
    "postcommit": "git update-index --again"
  }
}
```

_Thanks to [this comment](https://youtrack.jetbrains.com/issue/IDEA-135454#comment=27-2710654) for the fix!_

</details>

### How to use `lint-staged` in a multi-package monorepo?

<details>
  <summary>Click to expand</summary>

Install _lint-staged_ on the monorepo root level, and add separate configuration files in each package. When running, _lint-staged_ will always use the configuration closest to a staged file, so having separate configuration files makes sure tasks do not "leak" into other packages.

For example, in a monorepo with `packages/frontend/.lintstagedrc.json` and `packages/backend/.lintstagedrc.json`, a staged file inside `packages/frontend/` will only match that configuration, and not the one in `packages/backend/`.

**Note**: _lint-staged_ discovers the closest configuration to each staged file, even if that configuration doesn't include any matching globs. Given these example configurations:

```js
// ./.lintstagedrc.json
{ "*.md": "prettier --write" }
```

```js
// ./packages/frontend/.lintstagedrc.json
{ "*.js": "eslint --fix" }
```

When committing `./packages/frontend/README.md`, it **will not run** _prettier_, because the configuration in the `frontend/` directory is closer to the file and doesn't include it. You should treat all _lint-staged_ configuration files as isolated and separated from each other. You can always use JS files to "extend" configurations, for example:

```js
import baseConfig from '../.lintstagedrc.js'

export default {
  ...baseConfig,
  '*.js': 'eslint --fix',
}
```

To support backwards-compatibility, monorepo features require multiple _lint-staged_ configuration files present in the git repo. If you still want to run _lint-staged_ in only one of the packages in a monorepo, you can use the `--cwd` option (for example, `lint-staged --cwd packages/frontend`).

</details>

### Can I lint files outside of the current project folder?

<details>
  <summary>Click to expand</summary>

tl;dr: Yes, but the pattern should start with `../`.

By default, `lint-staged` executes tasks only on the files present inside the project folder(where `lint-staged` is installed and run from).
So this question is relevant _only_ when the project folder is a child folder inside the git repo.
In certain project setups, it might be desirable to bypass this restriction. See [#425](https://github.com/okonet/lint-staged/issues/425), [#487](https://github.com/okonet/lint-staged/issues/487) for more context.

`lint-staged` provides an escape hatch for the same(`>= v7.3.0`). For patterns that start with `../`, all the staged files are allowed to match against the pattern.
Note that patterns like `*.js`, `**/*.js` will still only match the project files and not any of the files in parent or sibling directories.

Example repo: [sudo-suhas/lint-staged-django-react-demo](https://github.com/sudo-suhas/lint-staged-django-react-demo).

</details>

### Can I run `lint-staged` in CI, or when there are no staged files?

<details>
  <summary>Click to expand</summary>

Lint-staged will by default run against files staged in git, and should be run during the git pre-commit hook, for example. It's also possible to override this default behaviour and run against files in a specific diff, for example
all changed files between two different branches. If you want to run _lint-staged_ in the CI, maybe you can set it up to compare the branch in a _Pull Request_/_Merge Request_ to the target branch.

Try out the `git diff` command until you are satisfied with the result, for example:

```
git diff --diff-filter=ACMR --name-only main...my-branch
```

This will print a list of _added_, _changed_, _modified_, and _renamed_ files between `main` and `my-branch`.

You can then run lint-staged against the same files with:

```
npx lint-staged --diff="main...my-branch"
```

</details>

### Can I use `lint-staged` with `ng lint`

<details>
  <summary>Click to expand</summary>

You should not use `ng lint` through _lint-staged_, because it's designed to lint an entire project. Instead, you can add `ng lint` to your git pre-commit hook the same way as you would run lint-staged.

See issue [!951](https://github.com/okonet/lint-staged/issues/951) for more details and possible workarounds.

</details>

### How can I ignore files from `.eslintignore`?

<details>
  <summary>Click to expand</summary>

ESLint throws out `warning File ignored because of a matching ignore pattern. Use "--no-ignore" to override` warnings that breaks the linting process ( if you used `--max-warnings=0` which is recommended ).

#### ESLint < 7

<details>
  <summary>Click to expand</summary>

Based on the discussion from [this issue](https://github.com/eslint/eslint/issues/9977), it was decided that using [the outlined script](https://github.com/eslint/eslint/issues/9977#issuecomment-406420893)is the best route to fix this.

So you can setup a `.lintstagedrc.js` config file to do this:

```js
import { CLIEngine } from 'eslint'

export default {
  '*.js': (files) => {
    const cli = new CLIEngine({})
    return 'eslint --max-warnings=0 ' + files.filter((file) => !cli.isPathIgnored(file)).join(' ')
  },
}
```

</details>

#### ESLint >= 7

<details>
  <summary>Click to expand</summary>

In versions of ESLint > 7, [isPathIgnored](https://eslint.org/docs/developer-guide/nodejs-api#-eslintispathignoredfilepath) is an async function and now returns a promise. The code below can be used to reinstate the above functionality.

Since [10.5.3](https://github.com/okonet/lint-staged/releases), any errors due to a bad ESLint config will come through to the console.

```js
import { ESLint } from 'eslint'

const removeIgnoredFiles = async (files) => {
  const eslint = new ESLint()
  const isIgnored = await Promise.all(
    files.map((file) => {
      return eslint.isPathIgnored(file)
    })
  )
  const filteredFiles = files.filter((_, i) => !isIgnored[i])
  return filteredFiles.join(' ')
}

export default {
  '**/*.{ts,tsx,js,jsx}': async (files) => {
    const filesToLint = await removeIgnoredFiles(files)
    return [`eslint --max-warnings=0 ${filesToLint}`]
  },
}
```

</details>

#### ESLint >= 8.51.0 && [Flat ESLint config](https://eslint.org/docs/latest/use/configure/configuration-files-new)

<details>
  <summary>Click to expand</summary>

ESLint v8.51.0 introduced [`--no-warn-ignored` CLI flag](https://eslint.org/docs/latest/use/command-line-interface#--no-warn-ignored). It suppresses the `warning File ignored because of a matching ignore pattern. Use "--no-ignore" to override` warning, so manually ignoring files via `eslint.isPathIgnored` is no longer necessary.

```json
{
  "*.js": "eslint --max-warnings=0 --no-warn-ignored"
}
```

**NOTE:** `--no-warn-ignored` flag is only available when [Flat ESLint config](https://eslint.org/docs/latest/use/configure/configuration-files-new) is used.

</details>

</details>

### How can I resolve TypeScript (`tsc`) ignoring `tsconfig.json` when `lint-staged` runs via Husky hooks?

<details>
  <summary>Click to expand</summary>

When running `lint-staged` via Husky hooks, TypeScript may ignore `tsconfig.json`, leading to errors like:

> **TS17004:** Cannot use JSX unless the '--jsx' flag is provided.  
> **TS1056:** Accessors are only available when targeting ECMAScript 5 and higher.  

See issue [#825](https://github.com/okonet/lint-staged/issues/825) for more details.  

#### Root Cause  

<details>
  <summary>Click to expand</summary>

1. `lint-staged` automatically passes matched staged files as arguments to commands.  
2. Certain input files can cause TypeScript to ignore `tsconfig.json`. For more details, see this TypeScript issue: [Allow tsconfig.json when input files are specified](https://github.com/microsoft/TypeScript/issues/27379).  

</details>  

#### Workaround 1: Use a [function signature](https://github.com/lint-staged/lint-staged?tab=readme-ov-file#example-run-tsc-on-changes-to-typescript-files-but-do-not-pass-any-filename-arguments) for the `tsc` command

<details>
  <summary>Click to expand</summary>

As suggested by @antoinerousseau in [#825 (comment)](https://github.com/lint-staged/lint-staged/issues/825#issuecomment-620018284), using a function prevents `lint-staged` from appending file arguments:  

**Before:**

```js
// package.json

"lint-staged": {
    "*.{ts,tsx}":[
      "tsc --noEmit",
      "prettier --write"
    ]
  }
```

**After:**

```js
// lint-staged.config.js
module.exports = {
  "*.{ts,tsx}": [
    () => "tsc --noEmit", 
    "prettier --write"
  ],
}
```

</details>

#### Workaround 2: Take the `sh` or `bash` to wrap the `tsc` command

<details>
  <summary>Click to expand</summary>

As suggested by @sombreroEnPuntas in [#825 (comment)](https://github.com/lint-staged/lint-staged/issues/825#issuecomment-674575655), wrapping `tsc` in a shell command prevents `lint-staged` from modifying its arguments:

**Before:**

```js
// package.json

"lint-staged": {
  "*.{ts,tsx}":[
    "tsc --noEmit",
    "prettier --write"
  ]
}
```

**After:**

```js
// package.json

"lint-staged": {
  "*.{ts,tsx}":[
    "bash -c 'tsc --noEmit'"
    "prettier --write"
  ]
}
```

**Note:** This approach may have cross-platform compatibility issues.

</details>

</details>
