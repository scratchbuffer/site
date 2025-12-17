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

## Prerequisites

### Install Node and NPM

Using [`nvm`](https://github.com/nvm-sh/nvm) is recommended.
After NVM itself is set up, keep it simple and install an LTS version of Node and NPM:

```shell
nvm install --lts
nvm use --lts
```

### Make Tool Selections

#### ECMAScript Modules, Not CommonJS

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