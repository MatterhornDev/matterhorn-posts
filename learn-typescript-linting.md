# How to Set Up ESLint for TypeScript

Written by: Ethan Arrowood

This guide will show you how to set up ESLint with a TypeScript project. The guide includes additional details on how to include formatting libraries such as [prettier]() and [standard](). It is broken up into 2 main sections. First, is a short, step-by-step walkthrough. The second, is a longer more detailed write up on linters, formatters, and how all the pieces work together. Make use of the table of contents and the `[toc]` shortcuts to better navigate this article.

### Table of Contents
- [1 Set Up Guide](#1-set-up-guide)
  - [1.1 Getting Started](#11-getting-started)
  - [1.2 Adding ESLint](#12-adding-eslint)
    - [1.2.1 Initializing .eslintrc](#121-initializing-.eslintrc)
    - [1.2.2 Specifying environments](#122-specifying-environments)
    - [1.2.3 Specifying ecmaVersion](#123-specifying-ecmaVersion)
    - [1.2.4 Creating ESLint npm script](#124-Creating-ESLint-npm-script)
    - [1.2.5 Executing ESLint](#125-executing-eslint)
    - [1.2.6 Fixing an ESLint warning](#126-fixing-an-eslint-warning)
    - [1.2.7 Configuring ESLint](#127-configuring-eslint)
    - [1.2.8 Additional ESLint rule configuration](#128-additional-eslint-rule-configuration)
    - [1.2.9 Fixing unused variable definition error from type import](#129-fixing-unused-variable-definition-error-from-type-import)
  - [1.3 Adding Standard Style Formatter](#13-adding-standard-style-formatter)
  - [1.4 Adding Prettier Style Formatter](#14-adding-prettier-style-formatter)
- [2 Additional Details](#2-additional-details)
  - [Additional Resources and Documentation](#additional-resources-and-documentation)

## 1 Set Up Guide

Section 1 is a step-by-step guide to configuring ESLint in a TypeScript project. Developers without an existing TypeScript project should start at section [1.1 Getting Started](#1.1-getting-started); developers with an existing project can skip ahead to section [1.2 Adding ESLint](#1.2-adding-eslint).

### 1.1 Getting Started
[`[toc]`](#table-of-contents)

I've created a basic TypeScript project to help you get started. Check out the GitHub repository, [`learn-typescript-linting`](https://github.com/MatterhornDev/learn-typescript-linting), and clone the repository to your local development environment. Copy, paste and execute the following command to clone it directly:
```bash
git clone git@github.com:MatterhornDev/learn-typescript-linting.git
git checkout init
# After cloning
cd learn-typescript-linting
npm install
```

The project comes with a single developer dependency, `typescript`, and two npm scripts, `compile` and `start`. The `compile` command is `tsc -p tsconfig.json`. The project is configured for `es5` in `strict` mode and includes all `.ts` files under the `src` directory. The compiled output can be found in the `lib` directory. The `start` command runs the compiled `.js` output via `node lib/index.js`. Try them out by running:
```bash
npm run compile
npm run start
```

It is recommended you do not modify the `.ts` files as they are specifically set up to show off unique aspects of linting typescript projects.

### 1.2 Adding ESLint
[`[toc]`](#table-of-contents)

Install `eslint`, `@typescript-eslint/eslint-plugin`, `@typescript-eslint/parser` as developer dependencies. Initialize an empty eslint configuration file. I prefer to use `.json` for configuration files.

```bash
npm i -D eslint @typescript-eslint/{eslint-plugin,parser}
touch .eslintrc.json
```

#### 1.2.1 Initializing .eslintrc
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

#### 1.2.2 Specifying environments
[`[toc]`](#table-of-contents)

The `env` object is used for defining global variables in a project that are not explicitly imported. A great example is the `console` object. This is considered globally available in both browser and Node.js environments. In the `learn-typescript-linting` example, the code will be executed in a Node.js terminal, thus the `node` attributes is enabled. Some other common attributes include `jest`, `mocha`, `amd`, `commonjs`, and `es6`. There is no easy way to know which attributes need to be enabled; it is recommended to consult [ESLint's Specifying Environments](https://eslint.org/docs/user-guide/configuring#specifying-environments) documentation to find out what each environment attribute provides.

#### 1.2.3 Specifying ecmaVersion
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

#### 1.2.4 Creating ESLint npm script
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

#### 1.2.5 Executing ESLint
[`[toc]`](#table-of-contents)

Run this command via `npm run lint`, you should get an output similar to:

```bash
> learn-typescript-linting@0.1.0 lint /learn-typescript-linting
> eslint 'src/**/*.ts'


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

npm ERR! code ELIFECYCLE
...
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

The source files have now been updated to use 4-spaces per indent rather than 2. Section [](), will describe how to change this configuration back to 2-spaces as well as how to enable semicolons.

#### 1.2.6 Fixing an ESLint warning
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

#### 1.2.7 Configuring ESLint
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

#### 1.2.8 Additional ESLint rule configuration
[`[toc]`](#table-of-contents)

Some other opinionated rules are indent spacing and semicolon use. Start by specifying the `@typescript-eslint/indent` rule with the value `["error", 2]`. Make sure to include `@typescript-eslint` in the rule name; otherwise, the configuration will target ESLint's base rule `indent` and not the typescript-eslint's rule `indent`. The value for this rule is different to that of `no-console`. It is an array of parameters. The first value refers to the lint _level_. It can have the value of `"error"` (default), `"warn"`, or `"off"`. Awfully familiar right? The second value is the number of spaces for space-based indentation or the string `"tab"` for tab based indentation. The third and optional value is an object of language specific configurations. Visit the `@typescript-eslint/eslint-plugin` [indent rule documentation](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin/docs/rules/indent.md) for more details.

Based on that documentation, Node.js standard development uses 2-space indentation. Configure the rule and run `npm run lint` to see the indentation errors in the `learn-typescript-linting` project. Run the command with the fix option `npm run lint -- --fix` to automatically fix the errors.

Next, configuring semicolons requires specifying the `semi` rule. Similar to the previous two rules it can be set to one of three values, `"error"`, `"warn"`, or `"off"` (default). My preference is _sans_ semicolons, but to each their own.

#### 1.2.9 Fixing unused variable definition error from type import
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

ESLint thinks that `CustomType` is never used; however, the source code is definitely using it, just not in the way ESLint expects it to. This error happens because of how ESLint works. ESLint parses JavaScript into an AST and analyzes that AST using the configured rule set. ESLint does not natively understand how to parse TypeScript; thus, the use of the `typescript-eslint/parser`. The linter then utilizes the `typescript-eslint/eslint-plugin` to enable a rule set to analyze the parsed TypeScrtipt code. To fix the above `no-unused-vars` error two rule configurations need to be set.

```json
{
  "rules": {
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": ["error"]
  }
}
```

This configuration turns off the base ESLint rule and enables the typescript-eslint rule instead. The typescript-eslint rule understands how to analyze TypeScript source code and will still catch normal JavaScript based unused variables. The `plugin:@typescript-eslint/recommended` specification creates this specification automatically which is why the error didn't appear earlier in this guide. Similar to the previous rules, additional configuration is available for the [no-unused-vars rule](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin/docs/rules/no-unused-vars.md).

### 1.3 Adding Standard Style Formatter
[`[toc]`](#table-of-contents)

### 1.4 Adding Prettier Style Formatter
[`[toc]`](#table-of-contents)

## 2 Additional Details
[`[toc]`](#table-of-contents)

### Additional Resources and Documentation
[`[toc]`](#table-of-contents)

- [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint): Monorepo for all the tooling which enables ESLint to support TypeScript.
  - [typesciprt-eslint/parser](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/parser)
  - [typescript-eslint/eslint-plugin](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin)
- [eslint-config-standard](https://github.com/standard/eslint-config-standard): The ESLint configurtion for integrating the JavaScript Standard Style formatter.
- [prettier](https://prettier.io/docs/en/index.html): Prettier code formatter documentation. Relevant sections of the documentation listed below.
  - [Integrating with Linters](https://prettier.io/docs/en/integrating-with-linters.html)
  - [Prettier vs. Linters](https://prettier.io/docs/en/comparison.html)
