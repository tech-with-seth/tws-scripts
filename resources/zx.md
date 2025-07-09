# Getting Started

## Overview

```js
#!/usr/bin/env zx

await $`cat package.json | grep name`;

const branch = await $`git branch --show-current`;
await $`dep deploy --branch=${branch}`;

await Promise.all([$`sleep 1; echo 1`, $`sleep 2; echo 2`, $`sleep 3; echo 3`]);

const name = 'foo bar';
await $`mkdir /tmp/${name}`;
```

Bash is great, but when it comes to writing more complex scripts,
many people prefer a more convenient programming language.
JavaScript is a perfect choice, but the Node.js standard library
requires additional hassle before using. The `zx` package provides
useful wrappers around `child_process`, escapes arguments and
gives sensible defaults.

## Install

```bash
npm install zx
```

or many [other ways](/setup)

## Usage

Write your scripts in a file with an `.mjs` extension in order to
use `await` at the top level. If you prefer the `.js` extension,
wrap your scripts in something like `void async function () {...}()`. [TypeScript](./typescript.md) is also supported.

Add the following shebang to the beginning of your `zx` scripts:

```bash
#!/usr/bin/env zx
```

Now you will be able to run your script like so:

```bash
chmod +x ./script.mjs
./script.mjs
```

Or via the [CLI](cli.md):

```bash
zx ./script.mjs
```

All functions (`$`, `cd`, `fetch`, etc) are available straight away
without any imports.

Or import globals explicitly (for better autocomplete in VS Code).

```js
import 'zx/globals';
```

### `` $`command` ``

Executes a given command using the `spawn` func
and returns [`ProcessPromise`](process-promise.md). It supports both sync and async modes.

```js
const list = await $`ls -la`;
const dir = $.sync`pwd`;
```

Everything passed through `${...}` will be automatically escaped and quoted.

```js
const name = 'foo & bar';
await $`mkdir ${name}`;
```

**There is no need to add extra quotes.** Read more about it in
[quotes](quotes.md).

You can pass an array of arguments if needed:

```js
const flags = ['--oneline', '--decorate', '--color'];
await $`git log ${flags}`;
```

In async mode, zx awaits any `thenable` in literal before executing the command.

```js
const a1 = $`echo foo`;
const a2 = new Promise((resolve) => setTimeout(resolve, 20, ['bar', 'baz']));

await $`echo ${a1} ${a2}`; // foo bar baz
```

If the executed program returns a non-zero exit code,
[`ProcessOutput`](#processoutput) will be thrown.

```js
try {
    await $`exit 1`;
} catch (p) {
    console.log(`Exit code: ${p.exitCode}`);
    console.log(`Error: ${p.stderr}`);
}
```

### `ProcessOutput`

```ts
class ProcessOutput {
    readonly stdout: string;
    readonly stderr: string;
    readonly signal: string;
    readonly exitCode: number;
    // ...
    toString(): string; // Combined stdout & stderr.
    valueOf(): string; // Returns .toString().trim()
}
```

The output of the process is captured as-is. Usually, programs print a new
line `\n` at the end.
If `ProcessOutput` is used as an argument to some other `$` process,
**zx** will use stdout and trim the new line.

```js
const date = await $`date`;
await $`echo Current date is ${date}.`;
```

## License

[Apache-2.0](https://github.com/google/zx/blob/main/LICENSE)

Disclaimer: _This is not an officially supported Google product._

## Setup

## Requirements

-   Linux, macOS, or Windows
-   JavaScript Runtime:
    -   Node.js >= 12.17.0
    -   Bun >= 1.0.0
    -   Deno 1.x, 2.x
    -   GraalVM Node.js
-   Some kind of bash or PowerShell

## Install

::: code-group

```bash [npm]
npm install zx     # add -g to install globally
```

```bash [npx]
npx zx script.js         # run script without installing the zx package
npx zx@8.6.0 script.js   # pin to a specific zx version
```

```bash [yarn]
yarn add zx
```

```bash [pnpm]
pnpm add zx
```

```bash [bun]
bun install zx
```

```bash [deno]
deno install -A npm:zx

# zx requires additional permissions: --allow-read --allow-sys --allow-env --allow-run
```

```bash [jsr]
npx jsr add @webpod/zx
deno add jsr:@webpod/zx

# https://jsr.io/docs/using-packages
```

```bash [docker]
docker pull ghcr.io/google/zx:8.5.0
docker run -t ghcr.io/google/zx:8.5.0 -e="await \$({verbose: true})\`echo foo\`"
docker run -t -i -v ./:/script ghcr.io/google/zx:8.5.0 script/t.js
```

```bash [brew]
brew install zx
```

:::

### Channels

zx is distributed in several versions, each with its own set of features.

| Channel  | Description                                                                                     | Install              |
| -------- | ----------------------------------------------------------------------------------------------- | -------------------- |
| `latest` | Mainline releases with the latest features and improvements.                                    | `npm i zx`           |
| `lite`   | [A minimalistic version of zx](./lite), suitable for lightweight scripts.                       | `npm i zx@lite`      |
| `dev`    | Development snapshots with the latest changes, may be unstable.                                 | `npm i zx@dev`       |
| `legacy` | Legacy supporting versions for compatibility with older scripts, no new features, only bugfixes | `npm i zx@<version>` |

Detailed comparison: [versions](./versions).

Please check the download sources carefully. Official links:

-   [npmjs](https://www.npmjs.com/package/zx)
-   [GH npm](https://github.com/google/zx/pkgs/npm/zx)
-   [GH repo](https://github.com/google/zx)
-   [GH docker](https://github.com/google/zx/pkgs/container/zx)
-   [JSR](https://jsr.io/@webpod/zx)
-   [Homebrew](https://github.com/Homebrew/homebrew-core/blob/master/Formula/z/zx.rb)

### GitHub

To fetch zx directly from the GitHub:

```bash
# Install via git
npm i google/zx
npm i git@github.com:google/zx.git

# Fetch from the GH pkg registry
npm i --registry=https://npm.pkg.github.com @google/zx
```

### Docker

If you'd prefer to run scripts in a container, you can pull the zx image from the [ghcr.io](https://ghcr.io).
[node:24-alpine](https://hub.docker.com/_/node) is used as [a base](https://github.com/google/zx/blob/main/dcr/Dockerfile).

```shell
docker pull ghcr.io/google/zx:8.5.0
docker run -t ghcr.io/google/zx:8.5.0 -e="await \$({verbose: true})\`echo foo\`"
docker run -t -i -v ./:/script ghcr.io/google/zx:8.5.0 script/t.js
```

## Bash

zx mostly relies on bash, so make sure it's available in your environment. If you're on Windows, consider using [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install) or [Git Bash](https://git-scm.com/downloads).
By default, zx looks for bash binary, but you can switch to PowerShell by invoking `usePowerShell()` or `usePwsh()`.

```js
import { useBash, usePowerShell, usePwsh } from 'zx';

usePowerShell(); // Use PowerShell.exe
usePwsh(); // Rely on pwsh binary (PowerShell v7+)
useBash(); // Switch back to bash
```

## Package

### Hybrid

zx is distributed as a [hybrid package](https://2ality.com/2019/10/hybrid-npm-packages.html): it provides both CJS and ESM entry points.

```js
import { $ } from 'zx';

const { $ } = require('zx');
```

It also contains built-in TypeScript libdefs. But `@types/fs-extra` and `@types/node` are required to be installed on user's side.

```bash
npm i -D @types/fs-extra @types/node
```

```ts
import { type Options } from 'zx';

const opts: Options = {
    quiet: true,
    timeout: '5s'
};
```

### Bundled

We use [esbuild](https://dev.to/antongolub/how-and-why-do-we-bundle-zx-1ca6) to produce a static build that allows us to solve several issues at once:

-   Reduce the pkg size and install time.
-   Make npx (yarn dlx / bunx) invocations reproducible.
-   Provide support for a wide range of Node.js versions: from [12 to 25](https://github.com/google/zx/blob/61d03329349770d90fda3c9e26f7ef09f869a096/.github/workflows/test.yml#L195).
-   Make auditing easier: complete code is in one place.

### Composite

zx exports several entry points adapted for different use cases:

-   `zx` – the main entry point, provides all the features.
-   `zx/global` – to populate the global scope with zx functions.
-   `zx/cli` – to run zx scripts from the command line.
-   `zx/core` – to use zx template spawner as part of 3rd party libraries with alternating set of utilities.

### Typed

The library is written in TypeScript 5 and provides comprehensive type definitions for TS users.

-   Libdefs are bundled via [dts-bundle-generator](https://github.com/timocov/dts-bundle-generator).
-   Compatible with TS 4.0 and later.
-   Requires `@types/node` and `@types/fs-extra` to be installed.

## Process Promise

The `$` returns a `ProcessPromise` instance, which inherits native `Promise`. When resolved, it becomes a [`ProcessOutput`](./process-output.md).

```js
const p = $`command`; // ProcessPromise
const o = await p; // ProcessOutput
```

By default, `$` spawns a new process immediately, but you can delay the start to trigger in manually.

```ts
const p = $({ halt: true })`command`;
const o = await p.run();
```

## `stage`

Shows the current process stage: `initial` | `halted` | `running` | `fulfilled` | `rejected`

```ts
const p = $`echo foo`;
p.stage; // 'running'
await p;
p.stage; // 'fulfilled'
```

## `stdin`

Returns a writable stream of the stdin process. Accessing
this getter will trigger execution of a subprocess with [`stdio('pipe')`](#stdio).

Do not forget to end the stream.

```js
const p = $`while read; do echo $REPLY; done`;
p.stdin.write('Hello, World!\n');
p.stdin.end();
```

By default, each process is created with stdin in _inherit_ mode.

## `stdout`/`stderr`

Returns a readable streams of stdout/stderr process.

```js
const p = $`npm init`;
for await (const chunk of p.stdout) {
    echo(chunk);
}
```

## `exitCode`

Returns a promise which resolves to the exit code of the process.

```js
if (await $`[[ -d path ]]`.exitCode == 0) {
...
}
```

## `json(), text(), lines(), buffer(), blob()`

Output formatters collection.

```js
const p = $`echo 'foo\nbar'`;

await p.text(); // foo\n\bar\n
await p.text('hex'); // 666f6f0a0861720a
await p.buffer(); // Buffer.from('foo\n\bar\n')
await p.lines(); // ['foo', 'bar']

// You can specify a custom lines delimiter if necessary:
await $`touch foo bar baz; find ./ -type f -print0`.lines('\0'); // ['./bar', './baz', './foo']

// If the output is a valid JSON, parse it in place:
await $`echo '{"foo": "bar"}'`.json(); // {foo: 'bar'}
```

## `[Symbol.asyncIterator]`

Returns an async iterator for the process stdout.

```js
const p = $`echo "Line1\nLine2\nLine3"`;
for await (const line of p) {
    console.log(line);
}

// Custom delimiter can be specified:
for await (const line of $({
    delimiter: '\0'
})`touch foo bar baz; find ./ -type f -print0`) {
    console.log(line);
}
```

## `pipe()`

Redirects the output of the process.

```js
await $`echo "Hello, stdout!"`.pipe(fs.createWriteStream('/tmp/output.txt'));

await $`cat /tmp/output.txt`;
```

`pipe()` accepts any kind `Writable`, `ProcessPromise` or a file path.
You can pass a string to `pipe()` to implicitly create a receiving file. The previous example is equivalent to:

```js
await $`echo "Hello, stdout!"`.pipe('/tmp/output.txt');
```

Chained streams become _thenables_, so you can `await` them:

```js
const p = $`echo "hello"`
    .pipe(getUpperCaseTransform())
    .pipe(fs.createWriteStream(tempfile())); // <- stream
const o = await p;
```

And the `ProcessPromise` itself is compatible with the standard `Stream.pipe` API:

```js
const { stdout } = await fs
    .createReadStream(await fs.writeFile(file, 'test'))
    .pipe(getUpperCaseTransform())
    .pipe($`cat`);
```

Pipes can be used to show a real-time output of the process:

```js
await $`echo 1; sleep 1; echo 2; sleep 1; echo 3;`.pipe(process.stdout);
```

And the time machine is in stock! You can pipe the process at any phase: on start, in the middle, or even after the end. All chunks will be buffered and processed in the right order.

```js
const result = $`echo 1; sleep 1; echo 2; sleep 1; echo 3`;
const piped1 = result.pipe`cat`;
let piped2;

setTimeout(() => {
    piped2 = result.pipe`cat`;
}, 1500)(await piped1)
    .toString()(
        // '1\n2\n3\n'
        await piped2
    )
    .toString(); // '1\n2\n3\n'
```

This mechanism allows you to easily split streams to multiple consumers:

```js
const p = $`some-command`;
const [o1, o2] = await Process.all([p.pipe`log`, p.pipe`extract`]);
```

The `pipe()` method can combine `$` processes. Same as `|` in bash:

```js
const greeting = await $`printf "hello"`
    .pipe($`awk '{printf $1", world!"}'`)
    .pipe($`tr '[a-z]' '[A-Z]'`);

echo(greeting);
```

Use combinations of `pipe()` and [`nothrow()`](#nothrow):

```js
await $`find ./examples -type f -print0`
    .pipe($`xargs -0 grep ${'missing' + 'part'}`.nothrow())
    .pipe($`wc -l`);
```

And literals! Pipe does support them too:

```js
await $`printf "hello"`.pipe`awk '{printf $1", world!"}'`
    .pipe`tr '[a-z]' '[A-Z]'`;
```

By default, `pipe()` API operates with `stdout` stream, but you can specify `stderr` as well:

```js
const p = $`echo foo >&2; echo bar`;
const o1 = (await p.pipe.stderr`cat`).toString(); // 'foo\n'
const o2 = (await p.pipe.stdout`cat`).toString(); // 'bar\n'
```

Btw, the signal, if specified, will be transmitted through pipeline.

```js
const ac = new AbortController();
const { signal } = ac;
const p = $({ signal, nothrow: true })`echo test`.pipe`sleep 999`;
setTimeout(() => ac.abort(), 50);

try {
    await p;
} catch ({ message }) {
    message; // The operation was aborted
}
```

In short, combine anything you want:

```js
const getUpperCaseTransform = () =>
    new Transform({
        transform(chunk, encoding, callback) {
            callback(null, String(chunk).toUpperCase());
        }
    });

// $ > stream (promisified) > $
const o1 = await $`echo "hello"`.pipe(getUpperCaseTransform()).pipe($`cat`);

o1.stdout; //  'HELLO\n'

// stream > $
const file = tempfile();
await fs.writeFile(file, 'test');
const o2 = await fs
    .createReadStream(file)
    .pipe(getUpperCaseTransform())
    .pipe($`cat`);

o2.stdout; //  'TEST'
```

## `kill()`

Kills the process and all children.

By default, signal `SIGTERM` is sent. You can specify a signal via an argument.

```js
const p = $`sleep 999`;
setTimeout(() => p.kill('SIGINT'), 100);
await p;
```

## `abort()`

Terminates the process via an `AbortController` signal.

```js
const ac = new AbortController();
const { signal } = ac;
const p = $({ signal })`sleep 999`;

setTimeout(() => ac.abort('reason'), 100);
await p;
```

If `ac` or `signal` is not provided, it will be autocreated and could be used to control external processes.

```js
const p = $`sleep 999`;
const { signal } = p;

const res = fetch('https://example.com', { signal });
p.abort('reason');
```

## `stdio()`

Specifies a standard input-output for the process.

```js
const h$ = $({ halt: true });
const p1 = h$`read`.stdio('inherit', 'pipe', null).run();
const p2 = h$`read`.stdio('pipe').run(); // sets ['pipe', 'pipe', 'pipe']
```

Keep in mind, `stdio` should be set before the process is started, so the preset syntax might be preferable:

```js
await $({ stdio: ['pipe', 'pipe', 'pipe'] })`read`;
```

## `nothrow()`

Changes behavior of `$` to not throw an exception on non-zero exit codes. Equivalent to [`$({nothrow: true})` option](./api#nothrow).

```js
await $`grep something from-file`.nothrow();

// Inside a pipe():
await $`find ./examples -type f -print0`
    .pipe($`xargs -0 grep something`.nothrow())
    .pipe($`wc -l`);

// Accepts a flag to switch nothrow mode for the specific command
$.nothrow = true;
await $`echo foo`.nothrow(false);
```

If only the `exitCode` is needed, you can use [`exitCode`](#exitcode) directly:

```js
if ((await $`[[ -d path ]]`.exitCode) == 0) {
    //...
}

// Equivalent of:

if ((await $`[[ -d path ]]`.nothrow()).exitCode == 0) {
    //...
}
```

## `quiet()`

Changes behavior of `$` to enable suppress mode.

```js
// Command output will not be displayed.
await $`grep something from-file`.quiet();

$.quiet = true;
await $`echo foo`.quiet(false); // Disable for the specific command
```

## `verbose()`

Enables verbose output. Pass `false` to disable.

```js
await $`grep something from-file`.verbose();

$.verbose = true;
await $`echo foo`.verbose(false); // Turn off verbose mode once
```

## `timeout()`

Kills the process after a specified timeout.

```js
await $`sleep 999`.timeout('5s');

// Or with a specific signal.
await $`sleep 999`.timeout('5s', 'SIGKILL');
```

## TypeScript

zx is written in TypeScript and provides the corresponding libdefs out of the box. Types are TS 4+ compatible. Write code in any suitable format `.ts`, `.mts`, `.cts` or add [a custom loader](./cli#non-standard-extension).

```ts
// script.ts
import { $ } from 'zx';

const list = await $`ls -la`;
```

Some runtimes like [Bun](https://bun.sh/) or [Deno](https://deno.com/) have built-in TS support. Node.js requires additional setup. Configure your project according to the [ES modules contract](https://nodejs.org/api/packages.html#packages_type):

-   Set [`"type": "module"`](https://nodejs.org/api/packages.html#packages_type)
    in **package.json**
-   Set [`"module": "ESNext"`](https://www.typescriptlang.org/tsconfig/#module)
    in **tsconfig.json**.

Using TypeScript compiler is the most straightforward way, but native TS support from runtimes is gradually increasing.

::: code-group

```bash [node]
# Since Node.js v22.6.0
node --experimental-strip-types script.js
```

```bash [npx]
# Since Node.js v22.6.0
NODE_OPTIONS="--experimental-strip-types" zx script.js
```

```bash [tsc]
npm install typescript

tsc script.ts

node script.js
```

```bash [ts-node]
npm install ts-node

ts-node script.ts
# or via node loader
node --loader ts-node/esm script.ts
```

```bash [swc-node]
npm install swc-node

swc-node script.ts
```

```bash [tsx]
npm install tsx

tsx script.ts
# or
node --import=tsx script.ts
```

```bash [bun]
bun script.ts
```

```bash [deno]
deno run --allow-read --allow-sys --allow-env --allow-run script.ts
```

:::

## FAQ

## Passing env variables

```js
process.env.FOO = 'bar';
await $`echo $FOO`;
```

## Passing array of values

When passing an array of values as an argument to `$`, items of the array will
be escaped
individually and concatenated via space.

Example:

```js
const files = [...]
await $`tar cz ${files}`
```

## Importing into other scripts

It is possible to make use of `$` and other functions via explicit imports:

```js
#!/usr/bin/env node
import { $ } from 'zx';

await $`date`;
```

## Attaching a profile

By default `child_process` does not include aliases and bash functions.
But you are still able to do it by hand. Just attach necessary directives
to the `$.prefix`.

```js
$.prefix += 'export NVM_DIR=$HOME/.nvm; source $NVM_DIR/nvm.sh; ';
await $`nvm -v`;
```

## Using GitHub Actions

The default GitHub Action runner comes with `npx` installed.

```yaml
jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            #      - uses: actions/setup-node@v4
            #        with:
            #          node-version: 22

            - name: Build with zx
              env:
                  FORCE_COLOR: 3
              run: |
                  npx zx <<'EOF'
                  await $`...`
                  EOF
```

## Verbose and Quiet

zx has internal logger, which captures events if a condition is met:

| Event  | Verbose | Quiet   | Description                |
| ------ | ------- | ------- | -------------------------- |
| stdout | `true`  | `false` | Spawned process stdout     |
| stderr | `any`   | `false` | Process stderr data        |
| cmd    | `true`  | `false` | Command execution          |
| fetch  | `true`  | `false` | Fetch resources by http(s) |
| cd     | `true`  | `false` | Change directory           |
| retry  | `true`  | `false` | Capture exec error         |
| custom | `true`  | `false` | User-defined event         |

By default, both `$.verbose` and `$.quiet` options are `false`, so only `stderr` events are written. Any output goes to the `process.stderr` stream.

You may control this flow globally or in-place

```js
// Global debug mode on
$.verbose = true;
await $`echo hello`;

// Suppress the particular command
await $`echo fobar`.quiet();

// Suppress everything
$.quiet = true;
await $`echo world`;

// Turn on in-place debug
await $`echo foo`.verbose();
```

You can also override the default logger with your own:

```js
// globally
$.log = (entry) => {
    switch (entry.kind) {
        case 'cmd':
            console.log('Command:', entry.cmd);
            break;
        default:
            console.warn(entry);
    }
};
// or in-place
$({ log: () => {} })`echo hello`;
```

## Canary / Beta / RC builds

Impatient early adopters can try the experimental zx versions.
But keep in mind: these builds are ⚠️️**beta** in every sense.

```bash
npm i zx@dev
npx zx@dev --install --quiet <<< 'import _ from "lodash" /* 4.17.15 */; console.log(_.VERSION)'
```
