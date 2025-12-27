+++
title = "Structured Eleventy, Part 1"
description = "Project Structure and Tool Selection"

date = 2025-12-13

+++

## Background

Eleventy is billed as a "simple static site generator".
Simplicity, of course, is in the eye of the beholder.

Eleventy's advantages are [well-enumerated](https://www.11ty.dev/#why-should-you-use-eleventy)
and the list starts off with promise: fast, stable, and zero-config!
Then it gets tricky -
it supports the use of multiple template languages at once,
and allows us to use any project structure you want.

If you are comfortable with more opinionated SSGs such as Hugo, Jekyll, or Zola,
this endless flexibility can be both a blessing and curse.
This series seeks to strike a balance -
we can take advantage of the flexibility while establishing some structure to keep our sanity intact.

## Goals

We will:

* Initialize a project structure with clear separation of layouts and content
* Select the features and tools we will use (or avoid) to maintain a clear structure
* Create basic layouts and render our first content!

## 0. Prerequisites

### 0.1 Make Tool Selections

#### ECMAScript Modules (ESM), Not CommonJS

For compatibility reasons, CommonJS is still the default selection when initializing an NPM project,
but ECMAScript module syntax is the new standard and is [recommended](https://www.11ty.dev/docs/cjs-esm/) by Eleventy.

#### TypeScript, Not JavaScript

This one should be an easy choice!
We are looking for more structure and less implicit behavior,
so we can jump through a few configuration hoops to enable Eleventy's TypeScript support.

To keep it simple at the start, we will not have any client-side JavaScript in our site,
so the TypeScript will be used only in the Eleventy config file.

#### Liquid, Not... All the Other Template Languages

Eleventy [supports numerous template languages](https://www.11ty.dev/docs/languages/) out of the box,
or even lets you add your own.

This choice is a bit personal, but I made a selection based on a few criteria:
* Layout authors should not need to write JavaScript to create HTML (no MDX, EJS, JSX, or TypeScript)
* The templates should generally _look_ like HTML (no Pug or HAML)
* The template language should be maintained, with recent releases (No Handlebars, Mustache, or Nunjucks)

This leaves... Liquid!
Liquid is straightforward, actively maintained, popular, and has just the right amount of features
to enable any layout we could want without complex logic.

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

This will create the default `package.json`:
```json
{
  "name": "eleventy-demo",
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

You may also want to update other keys:
```shell
npm pkg set license="MIT"
npm pkg set version="0.0.0-alpha.0"
```

Or delete them altogether - we do not have an entrypoint script,
so you can drop the "main" key and anything else you are uninterested in.
```shell
npm pkg delete main
npm pkg delete description keywords author
```

## 2. Install and Run Eleventy

### 2.1 Install

Install as a regular or dev dependency -
the Eleventy package only generates the static site and is not required at runtime:
```shell
npm install @11ty/eleventy
```
or:
```shell
npm install --save-dev @11ty/eleventy
```
### 2.2 Run

Now we can run the Eleventy build command:

```shell
npx @11ty/eleventy
```

We have no templates, content, layouts, or input of any kind so we should see that it successfully did nothing:
```console
[11ty] Wrote 0 files in 0.03 seconds (v3.1.2)
```

By default, Eleventy writes the output to `<project root>/_site`,
but without any files to it does not bother to create the `_site` directory.

We can even run the development server:
```shell
npx @11ty/eleventy --serve
```
and see that it also produces nothing:
```console
[11ty] Wrote 0 files in 0.04 seconds (v3.1.2)
[11ty] Watching…
[11ty] Server at http://localhost:8080/
```

Navigating to any `localhost:8080` URL in the browser will get the same result:
* [http://localhost:8080/](http://localhost:8080/) shows `Cannot GET /`
* [http://localhost:8080/hello](http://localhost:8080/hello) shows `Cannot GET /hello`
* ...etc.

To tidy up, we can place the build and run commands in `package.json`'s `scripts` key:
```json
"scripts": {
    "build": "npx @11ty/eleventy",
    "serve": "npx @11ty/eleventy --serve"
  },
```
and run them with `npm run build` or `npm run serve`.

### 2.3 Stop

Do not do anything else!
The official Eleventy tutorial will tell you to go ahead and create some "templates" and start serving pages,
but we are not following that guide for a reason.

The magic "no-config" default configurations for "templates", "includes", and "layouts",
obscure quite a bit of how the system works and will be unfamiliar for users of other static site generators.

In order to understand Eleventy better, we will have to delay gratification just a bit longer.

## Initialize Eleventy Config with TypeScript

We will assume for now that 