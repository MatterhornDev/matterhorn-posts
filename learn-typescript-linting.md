# How to Set Up ESLint for TypeScript

Written by: Ethan Arrowood

This guide will show you how to set up [ESLint](https://eslint.org/) with a [TypeScript](https://www.typescriptlang.org/) project. The guide is broken up into five sections. The first two are about setting up ESLint and configuring it to work with TypeScript. The third and forth sections are for integrating popular styling formatters [Standard](https://standardjs.com/) and [Prettier](https://prettier.io/). The last section contains additional context and a list of resources for those interested in learning more. Make use of the table of contents and the `[toc]` shortcuts to better navigate this article.

### Table of Contents
- [1 Getting Started](#1-getting-started)
- [2 Adding ESLint](#2-adding-eslint)
  - [2.1 Initializing .eslintrc](#21-initializing-eslintrc)
  - [2.2 Specifying environments](#22-specifying-environments)
  - [2.3 Specifying ecmaVersion](#23-specifying-ecmaVersion)
  - [2.4 Creating ESLint npm script](#24-Creating-ESLint-npm-script)
  - [2.5 Executing ESLint](#25-executing-eslint)
  - [2.6 Fixing an ESLint warning](#26-fixing-an-eslint-warning)
  - [2.7 Configuring ESLint](#27-configuring-eslint)
  - [2.8 Additional ESLint rule configuration](#28-additional-eslint-rule-configuration)
  - [2.9 Fixing unused variable definition error from type import](#29-fixing-unused-variable-definition-error-from-type-import)
- [3 Adding Standard Style Formatter](#3-adding-standard-style-formatter)
  - [3.1 Installing Standard](#31-installing-standard)
  - [3.2 Evaluating new errors](#32-evaluating-new-errors)
  - [3.3 Configuring Standard specific rules](#33-configuring-standard-specific-rules)
- [4 Adding Prettier Style Formatter](#4-adding-prettier-style-formatter)
  - [4.1 Installing Prettier](#41-installing-prettier)
  - [4.2 Executing Prettier](#42-executing-prettier)
  - [4.3 Configuring Prettier specific rules](#43-configuring-prettier-specific-rules)
- [5 Additional Resources and Documentation](#5-additional-resources-and-documentation)

## 1 Getting Started
[`[toc]`](#table-of-contents)

**Note:*** Developers without an existing TypeScript project should start here at section 1; developers with an existing project can skip ahead to section [2 Adding ESLint](#2-adding-eslint). This guide works best if you follow along with the GitHub repository.

View the GitHub repository [`learn-typescript-linting`](https://github.com/MatterhornDev/learn-typescript-linting). Copy, paste and execute the following command to clone it to your machine:

```bash
git clone git@github.com:MatterhornDev/learn-typescript-linting.git
git checkout init
# After cloning
cd learn-typescript-linting
npm install
```

The repository comes with multiple branches for different points in the guide.
- `init`: a baseline repo without ESLint installed so you can follow along (section [2](#2-adding-eslint))
- `master`: a complete example of TypeScript with ESLint (section [2](#2-adding-eslint))
- `unused-variable`: an example of a common TypeScript + ESLint error (section [2.9](#29-fixing-unused-variable-definition-error-from-type-import))
- `standard-style`: a complete example of with Standard (section [3](#3-adding-standard-style-formatter))
- `prettier-style`: a complete example of with Prettier (section [4](#4-adding-prettier-style-formatter))

The project comes with a single developer dependency, `typescript`, and two npm scripts, `compile` and `start`. The `compile` command is `tsc -p tsconfig.json`. The project is configured for `es5` in `strict` mode and includes all `.ts` files under the `src` directory. The compiled output can be found in the `lib` directory. The `start` command runs the compiled `.js` output via `node lib/index.js`. Try them out by running:

```bash
npm run compile
npm run start
```

It is recommended you do not modify the `.ts` files as they are specifically set up to show off unique aspects of linting typescript projects.

## 2 Adding ESLint
[`[toc]`](#table-of-contents)

Install `eslint`, `@typescript-eslint/eslint-plugin`, `@typescript-eslint/parser` as developer dependencies. Initialize an empty eslint configuration file. I prefer to use `.json` for configuration files.

```bash
npm i -D eslint @typescript-eslint/{eslint-plugin,parser}
touch .eslintrc.json
```

### 2.1 Initializing .eslintrc
[`[toc]`](#table-of-contents)

Add the following to `.eslintrc.json`
```json
{
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "env": { "node": true },
  "parserOptions": {
    "ecmaVersion": 5,
    "sourceType": "module"
  }
}
```

### 2.2 Specifying environments
[`[toc]`](#table-of-contents)

The `env` object is used for defining global variables in a project that are not explicitly imported. A great example is the `console` object. This is considered globally available in both browser and Node.js environments. In the `learn-typescript-linting` example, the code will be executed in a Node.js terminal, thus the `node` attributes is enabled. Some other common attributes include `jest`, `mocha`, `amd`, `commonjs`, and `es6`. There is no easy way to know which attributes need to be enabled; it is recommended to consult [ESLint's Specifying Environments](https://eslint.org/docs/user-guide/configuring#specifying-environments) documentation to find out what each environment attribute provides.

### 2.3 Specifying ecmaVersion
[`[toc]`](#table-of-contents)

The `parserOptions.ecmaVersion` value is based on the `target` value found in the `tsconfig.json`. A `tsconfig.json` value of `{ "target": "es5" }` is equivalent to `{ "ecmaVersion": 5 }`. Use the table below for additional mappings.

| tsconfig.json `target` | .eslintrc.json `ecmaVersion` |
| ------ | ----------- |
| `es3` | 3 |
| `es5` | 5 |
| `es6` or `ES2015` | 6 or 2015 |
| `ES2016` | 7 or 2016 |
| `ES2017` | 8 or 2017 |
| `ES2018` | 9 or 2018 |
| `ES2019` | 10 or 2019 |
| `ESNext` | 10 or 2019 |

Take a look at TypeScript's `--lib` [compiler options](https://www.typescriptlang.org/docs/handbook/compiler-options.html) to learn how to include unique library files in the compilation. By setting `target` to either `es5` or `es6`, TypeScript will automatically import a set of libraries (i.e. `{ target: es5 } = { lib: ['DOM', 'ES5', 'ScriptHost']}`).

### 2.4 Creating ESLint npm script
[`[toc]`](#table-of-contents)

Now that ESLint is configured, create a new npm script in `package.json`:

```json
{
  "scripts": {
    "lint": "eslint 'src/**/*.ts'"
  }
}
```

This command will run ESLint on all `.ts` files within the `src` directory. The `/**/*` glob pattern will reach all files within subdirectories of `src`. If you have multiple directories to run the linter on (such as a `test` directory), use a pattern such as: `{src,test}/**/*.ts`.

### 2.5 Executing ESLint
[`[toc]`](#table-of-contents)

Run this command via `npm run lint`, you should get an output similar to:

```bash
/learn-typescript-linting/src/bar.ts
  4:1  error  Expected indentation of 4 spaces but found 2  @typescript-eslint/indent
  4:19  warning  Missing return type on function            @typescript-eslint/explicit-function-return-type

/learn-typescript-linting/src/foo.ts
  4:1  error  Expected indentation of 4 spaces but found 2  @typescript-eslint/indent
  5:1  error  Expected indentation of 4 spaces but found 2  @typescript-eslint/indent

/learn-typescript-linting/src/index.ts
   9:1  error  Unexpected console statement  no-console
  10:1  error  Unexpected console statement  no-console

✖ 6 problems (5 errors, 1 warnings)
  3 errors and 0 warnings potentially fixable with the `--fix` option.
```

This output shows two main things. First, it outputs linting errors and warnings (i.e. `@typescript-eslint/indent` and `no-console`). These will come in handy for further configuring ESLint. Second, it says _" 3 errors and 0 warnings potentially fixable with the `--fix` option."_. ESLint comes with a great CLI option `--fix` to automatically fix certain linting errors and warnings.

Run `npm run lint -- --fix` to pass the `--fix` option down to the `eslint` command. After it completes there should be a new output:

```bash
/learn-typescript-linting/src/bar.ts
  4:21  warning  Missing return type on function  @typescript-eslint/explicit-function-return-type

/learn-typescript-linting/src/index.ts
   9:1  error  Unexpected console statement  no-console
  10:1  error  Unexpected console statement  no-console

✖ 3 problems (2 errors, 1 warning)
```

The source files have now been updated to use 4-spaces per indent rather than 2. Section [2.8](#28-additional-eslint-rule-configuration), will describe how to change this configuration back to 2-spaces as well as how to enable semicolons.

### 2.6 Fixing an ESLint warning
[`[toc]`](#table-of-contents)

TypeScript exists to help developers write better and safer code. The typescript-eslint package helps accomplish this by warning of missing explicit types. The warning:

```bash
/learn-typescript-linting/src/bar.ts
  4:21  warning  Missing return type on function  @typescript-eslint/explicit-function-return-type
```

refers to the missing return type from the arrow function inside the `.reduce` method in the `bar.ts` file. Modify the code by specifying a return type:

```diff
import { CustomType } from './foo'

export function bar (a: CustomType, b: CustomType[]): CustomType {
-    return b.reduce((c, v) => c+=v, a)
+    return b.reduce((c, v): CustomType => c+=v, a)
}
```

Run the linter (`npm run lint`), observe how the previous warning no longer exists!

### 2.7 Configuring ESLint
[`[toc]`](#table-of-contents)

To start, configure the `no-console` rule by adding the following to the `.eslintrc.json`

```json
{
  "rules": {
    "no-console": "warn"
  }
}
```

The `no-console` rule can be configured to one of three values: `"error"` (default), `"warn"`, or `"off"`. Specify the rule to whichever value best serves the project at hand. In my opinion, `no-console` should be enabled as a warning because in a production application it is considered best-practice not to log to console, but to instead use a legitimite logger such as [Pino](http://getpino.io/#/).

### 2.8 Additional ESLint rule configuration
[`[toc]`](#table-of-contents)

Some other opinionated rules are indent spacing and semicolon use. Start by specifying the `@typescript-eslint/indent` rule with the value `["error", 2]`. Make sure to include `@typescript-eslint` in the rule name; otherwise, the configuration will target ESLint's base rule `indent` and not the typescript-eslint's rule `indent`. The value for this rule is different to that of `no-console`. It is an array of parameters. The first value refers to the lint _level_. It can have the value of `"error"` (default), `"warn"`, or `"off"`. Awfully familiar right? The second value is the number of spaces for space-based indentation or the string `"tab"` for tab based indentation. The third and optional value is an object of language specific configurations. Visit the `@typescript-eslint/eslint-plugin` [indent rule documentation](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin/docs/rules/indent.md) for more details.

Based on that documentation, Node.js standard development uses 2-space indentation. Configure the rule and run `npm run lint` to see the indentation errors in the `learn-typescript-linting` project. Run the command with the fix option `npm run lint -- --fix` to automatically fix the errors.

Next, configuring semicolons requires specifying the `semi` rule. Similar to the previous two rules it can be set to one of three values, `"error"`, `"warn"`, or `"off"` (default). My preference is _sans_ semicolons, but to each their own (do not try to debate me on this 😊).

### 2.9 Fixing unused variable definition error from type import
[`[toc]`](#table-of-contents)

This final section covers a common problem case when linting TypeScript with ESLint. If you __do not__ specify `plugin:@typescript-eslint/recommended` in the `.eslintrc.json` configuration `"extends"` list, then a troubling error will be returned from the `learn-typescript-linting` project. Modify the code by removing the second value in the `"extends"` list, or checkout the `learn-typescript-linting/unused-variable` branch to see the error.

```bash
/learn-typescript-linting/src/bar.ts
  1:10  error  'CustomType' is defined but never used  no-unused-vars

/learn-typescript-linting/src/index.ts
   1:10  error    'CustomType' is defined but never used  no-unused-vars
   9:1   warning  Unexpected console statement            no-console
  10:1   warning  Unexpected console statement            no-console

✖ 4 problems (2 errors, 2 warnings)
```

ESLint thinks that `CustomType` is never used; however, the source code is definitely using it, just not in the way ESLint expects it to. This error happens because of how ESLint works. ESLint parses JavaScript into an AST and analyzes that AST using the configured rule set. ESLint does not natively understand how to parse TypeScript; thus, the use of the `typescript-eslint/parser`. The linter then utilizes the `typescript-eslint/eslint-plugin` to enable a rule set to analyze the parsed TypeScrtipt code. For more information on the inner workings of `typescript-eslint` read their documentation on [_How does `typescript-eslint` work and why do you have multiple packages_](https://github.com/typescript-eslint/typescript-eslint#how-does-typescript-eslint-work-and-why-do-you-have-multiple-packages). 

To fix the above `no-unused-vars` error, set two rule configurations.

```json
{
  "rules": {
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": ["error"]
  }
}
```

This configuration turns off the base ESLint rule and enables the typescript-eslint rule instead. The typescript-eslint rule understands how to analyze TypeScript source code and will still catch normal JavaScript based unused variables. The `plugin:@typescript-eslint/recommended` specification creates this specification automatically which is why the error didn't appear earlier in this guide. Similar to the previous rules, additional configuration is available for the [no-unused-vars rule](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin/docs/rules/no-unused-vars.md).

## 3 Adding Standard Style Formatter
[`[toc]`](#table-of-contents)

[Standard](https://standardjs.com) is a JavaScript style guide, linter, and formatter. It is opinionated and does not require any special configuration. A part of the appeal to using Standard is that it _just works_. The Standard website has documentation on how to set it up standalone with TypeScript; however, with the deprecation of `eslint-plugin-typescript` I have not found a way to configure Standard with `@typescript-eslint/eslint-plugin`. If this changes I will make sure to update this post! Checkout the [`learn-typescript-linting/standard-style`](https://github.com/MatterhornDev/learn-typescript-linting/tree/standard-style) branch for a completed example.

### 3.1 Installing Standard
[`[toc]`](#table-of-contents)

To get started, run the following command:

```bash
npm i -D eslint-config-standard eslint-plugin-{standard,promise,import,node}
```

The additional eslint-plugin's are peer dependencies of `eslint-config-standard` and are required.

Add it to the config by prepending it to the end of the `"extends"` list.

```json
{
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended", "standard"]
}
```

### 3.2 Evaluating new errors
[`[toc]`](#table-of-contents)

Executing `npm run lint` on the `learn-typescript-linting` repository should result in the following output:

```bash
/learn-typescript-linting/src/bar.ts
  4:19  error  Arrow function should not return assignment    no-return-assign
  4:42  error  Operator '+=' must be spaced                   space-infix-ops
  5:2   error  Newline required at end of file but not found  eol-last

/learn-typescript-linting/src/foo.ts
  3:20  error  Missing space before function parentheses      space-before-function-paren
  6:2   error  Newline required at end of file but not found  eol-last

/learn-typescript-linting/src/index.ts
   9:1   warning  Unexpected console statement                   no-console
  10:1   warning  Unexpected console statement                   no-console
  10:23  error    Newline required at end of file but not found  eol-last

✖ 8 problems (6 errors, 2 warnings)
  5 errors and 0 warnings potentially fixable with the `--fix` option.
```

Now, there are a couple new errors that need fixing. When using an opinionated formatter such as Standard it is best practice not to modify the rule configuration much. Looking at the rules currently throwing errors, many are easy fixes. Run `npm run lint -- --fix` to have ESLint make the changes automatically. The remaining error is a basic code fix:

```diff
import { CustomType } from './foo'

export function bar (a: CustomType, b: CustomType[]): CustomType {
-  return b.reduce((c, v): CustomType => c += v, a)
+  return b.reduce((c, v): CustomType => c + v, a)
}

```

The previous code snippet was actually considered computationally wasteful as the `+=` assignment was an unnecessary operation, even though it has the same runtime output of just `+`. In larger scale applications catching errors like this can make a great performance difference. This is why using linters are important! They can alert developers of potential issues in their code base.

Run `npm run lint` one last time to verify everything is passing as expected. Remember that the warnings for `no-console` are expected based on the `.eslintrc.json` configuration.

### 3.3 Configuring Standard specific rules
[`[toc]`](#table-of-contents)

If you really need to modify the Standard formatting rules then override them in the `.eslintrc.json` file by adding the ESLint rules to the `rules` object. Unlike `typescript-eslint`, Standard does not add any additional rules, so everything can be overwritten using the ESLint style. For example, to turn off the `semicolon` rule add `"semi": "off"` to the `"rules"` object.

Take a moment to read Standard's FAQ answer to the [_can I configure_](https://standardjs.com/index.html#i-disagree-with-rule-x-can-you-change-it) question:

> No. The whole point of standard is to save you time by avoiding [bikeshedding](https://www.freebsd.org/doc/en/books/faq/misc.html#bikeshed-painting) about code style. There are lots of debates online about tabs vs. spaces, etc. that will never be resolved. These debates just distract from getting stuff done. At the end of the day you have to 'just pick something', and that's the whole philosophy of standard -- its a bunch of sensible 'just pick something' opinions. Hopefully, users see the value in that over defending their own opinions.

## 4 Adding Prettier Style Formatter
[`[toc]`](#table-of-contents)

[Prettier](https://prettier.io/) is an opinionated code formatter that brilliantly integrates with many languages and editors. It has limited configuration options to simplify code formatting. It works seemlessly with ESLint and will transform your coding workflow if used properly.

### 4.1 Installing Prettier
[`[toc]`](#table-of-contents)

Just like Standard, Prettier is easy to get up and running with the existing ESLint configuration. Start by running the following command:

```bash
npm i -D prettier eslint-config-prettier eslint-plugin-prettier
```

Modify the `.eslintrc.json` configuration to use the newely installed configuration and plugin.

```json
{
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended", "plugin:prettier/recommended"],
  "plugins": ["@typescript-eslint", "prettier"]
}
```

### 4.2 Executing Prettier
[`[toc]`](#table-of-contents)

Running `npm run lint` on the `learn-typescript-linting` project should result in the following output:

```bash
/learn-typescript-linting/src/bar.ts
  1:28  error  Replace `'./foo'` with `"./foo";`        prettier/prettier
  3:20  error  Delete `·`                               prettier/prettier
  4:41  error  Replace `c+=v,·a)` with `(c·+=·v),·a);`  prettier/prettier
  5:2   error  Insert `⏎`                               prettier/prettier

/learn-typescript-linting/src/foo.ts
  1:32  error  Insert `;`  prettier/prettier
  4:19  error  Insert `;`  prettier/prettier
  5:31  error  Insert `;`  prettier/prettier
  6:2   error  Insert `⏎`  prettier/prettier

/learn-typescript-linting/src/index.ts
   1:33  error    Replace `'./foo'` with `"./foo";`  prettier/prettier
   2:21  error    Replace `'./bar'` with `"./bar";`  prettier/prettier
   4:25  error    Insert `;`                         prettier/prettier
   5:40  error    Insert `;`                         prettier/prettier
   6:24  error    Insert `;`                         prettier/prettier
   7:24  error    Insert `;`                         prettier/prettier
   9:1   warning  Unexpected console statement       no-console
   9:23  error    Insert `;`                         prettier/prettier
  10:1   warning  Unexpected console statement       no-console
  10:23  error    Insert `;⏎`                        prettier/prettier

✖ 18 problems (16 errors, 2 warnings)
  16 errors and 0 warnings potentially fixable with the `--fix` option.
```

Similar to Standard, there are plenty of easily fixable errors. Run `npm run lint -- --fix` to automatically fix all of the listed errors.

### 4.3 Configuring Prettier specific rules
[`[toc]`](#table-of-contents)

As mentioned previously, the purpose of using a formatter such as prettier is so that there is no configuration needed. Nevertheless, understanding how to modify rules is important to comply to an organization's code formatting practices. Lets use the classic `semicolon` rule as an example.

To modify Prettier rules you must create either a new `.prettierrc.json` file or add a `"prettier"` section to the `package.json`. In order to eliminate maintaining additional files, I prefer to add it directly to the package.json.

```json
{
  "prettier": {
    "semi": false
  }
}
```

Run `npm run lint` and see how the linter now errors on all the semicolons added in the previous section. Fix them automatically or remove the rule to go back to using semicolons.

## 5 Additional Resources and Documentation
[`[toc]`](#table-of-contents)

- [Learn TypeScript Linting post repository](https://github.com/MatterhornDev/matterhorn-posts/blob/master/learn-typescript-linting.md): This post is open sourced! Check it out at the link and open issues/pull requests if you'd like to contribute to it.
- [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint): Monorepo for all the tooling which enables ESLint to support TypeScript.
  - [typesciprt-eslint/parser](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/parser)
  - [typescript-eslint/eslint-plugin](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin)
- [eslint-config-standard](https://github.com/standard/eslint-config-standard): The ESLint configurtion for integrating the JavaScript Standard Style formatter.
- [Prettier documentation](https://prettier.io/docs/en/index.html): Prettier code formatter documentation. Relevant sections of the documentation listed below.
  - [Integrating with Linters](https://prettier.io/docs/en/integrating-with-linters.html)
  - [Prettier vs. Linters](https://prettier.io/docs/en/comparison.html)
- [Write Perfect Code With Standard and ESLint](https://www.youtube.com/watch?v=kuHfMw8j4xk): a great talk on linters and formatters by Feross Aboukhadijeh at JSConf.Asia 2018

---

Thank you for reading! If you enjoyed this article follow [@MatterhornDev](https://twitter.com/matterhorndev) on Twitter for notifications on all future posts. This article was written by Ethan Arrowood, share you support on Twitter by following him ([@ArrowoodTech](https://twitter.com/ArrowoodTech)) and [sharing this article](https://twitter.com/intent/tweet?text=Learn%20TypeScript%20Linting%20by%20@ArrowoodTech&url=https://github.com/MatterhornDev/matterhorn-posts/blob/master/learn-typescript-linting.md&hashtags=typescript,eslint,standardjs,prettier&via=MatterhornDev&related=ArrowoodTech,MatterhornDev).
