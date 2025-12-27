+++
title = "Modern JavaScript & TypeScript Project Setup with NPM"
description = "Initialize and Configure Packages, Type Checkers, Linters, and Formatters"

date = 2025-12-13

+++

## Preface

Despite the best efforts of many talented engineers,
we have limped through the first quarter-century of the new millennium and
JavaScript is still the lingua franca of the browser and web development in general.

This terrifying ecosystem grows and mutates faster than the Shimmer in Annihilation,
but much like film representation of the science-fiction-horror "anomalous zone",
we gain nothing by ignoring it or hoping it disappears.

So here we go!

In an effort to stick to common, standardized tooling and keep it simple for newcomers,
I have restricted the problem area in two primary ways:

1. Choosing NPM - While many tried-and-true (Yarn) and upstart (Deno) alternatives have their virtues,
    Node.js and its ecosystem dominate the experience of JavaScript development.

2. _Not_ choosing a bundler - JavaScript bundling and distribution are topics of their own,
    and does not fit into our focus here of getting up and running with a sensible dev environment.

## Goals

We will initialize and configure:

* A basic NPM project, with development and production dependencies
* TypeScript, as a type checker
* ESLint, to lint for modern JavaScript (ECMAScript) standards
* Prettier, to maintain consistent formatting

## 0. Prerequisites

### 0.1 Understand Tool Selections

#### 0.1.1 ECMAScript Modules, Not CommonJS

For compatibility reasons, CommonJS (CJS) syntax is still the default for new NPM projects,
but ECMAScript module (ESM) syntax is the official standard for JavaScript and is better-supported in TypeScript.
Node first shipped with module support in 2019 with Node version 12.

#### 0.1.2 TypeScript, Not (just) JavaScript

As lamented above, we cannot opt out of the JavaScript ecosystem completely without becoming monks
(writing Rust to compile to WebAssembly).
We _can_ however be good citizens of this apocalyptic ecosystem and use types in our software.

#### 0.1.3 ESLint, Not the Other Linters

ESLint is the standard, TSLint is deprecated.
There are certainly others, but you can explore them on your own time.
This is my blog!

#### 0.1.4 Prettier, Not the Other Formatters

Prettier is a highly opinionated formatter,
and has become an industry standard despite the frustrations that opinionation can cause.
I do not love some of Prettier's formatting myself, but I dislike fiddling and bikeshedding over formatting even more.
Best to accept it and move on!

### 0.2 Install Node and NPM

Using [`nvm`](https://github.com/nvm-sh/nvm) is recommended.
After NVM itself is set up, keep it simple and install an LTS version of Node and NPM:
```shell
nvm install --lts
nvm use --lts
```

## 1. Initialize NPM Project

Initialize NPM from the project root directory.
To start with the most basic settings, we can just accept the NPM defaults.
```shell
npm init -y
```

This will create the default `package.json`, with the name based on the project directory:
```json
{
  "name": "js-ts-npm-starter-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs"
}
```

As we decided on using ESM we do not want `"type": "commonjs"`.
Fix this:

```shell
npm pkg set type="module"
```

We may also want to update other keys:
```shell
npm pkg set license="MIT"
npm pkg set version="0.0.0-alpha.0"
```

Or delete them altogether - the `main` key is unneeded until we distribute our code,
and is superceded by the more modern [`exports`](https://docs.npmjs.com/cli/v11/configuring-npm/package-json#exports) key.
We can drop it and any other keys we are uninterested in for now.
```shell
npm pkg delete main
npm pkg delete description keywords author
```

## 2. Create a Basic Node Application in JavaScript

No TypeScript yet!
Focus on vanilla Node package usage first,
_then_  we can appreciate the addition of Typescript.

### 2.1 Install Node as Dependency

The Node CLI we installed with `nvm` lives outside the project.
While it will allow us to run Node code without declaring it as a dependency,
this can trip us up later when building and packaging for production environments.

Installing Node into our project also declares explicitly in our `package.json` which version our project is built for.

Run:
```shell
npm install node
```

And note the new `dependencies` declaration in `package.json`:

```json
"dependencies": {
    "node": "^24.12.0"
  },
```



