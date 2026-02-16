+++
type = "article"
title = "Modern TypeScript Project Setup with NPM, Part 2"
description = "Lint and Format TypeScript with JavaScript Standard Style"

slug = "typescript-npm-setup-part-2"

tags = ['JavaScript', 'TypeScript', 'NPM']

date = 2025-12-15
+++

## Preface

We are back!

In [Part 1](/topics/web-development/typescript-npm-setup-part-1/),
we entered the anomalous zone and came back with a basic Node server project with TypeScript,
which can be type-checked, compiled and run with `tsx` and `tsc`.

Right back into the Shimmer we go - not because it is easy, but because for some reason we have to... or want to.
The JavaScript ecosystem has certainly only grown larger and more terrifying since we first encountered in it Part 1.
But this time, we will use some new equipment to help protect us: linting and formatting.

While static type checking provides some protection against the runtime type errors which plague dynamic languages,
it does little to help with enforcing general good coding practices for any given language.

Our goal here is to minimize the detrimental effects of endless optionality -
to stop fiddling with configuration knobs or different function syntaxes and just produce code!

## Goals

We will:

* Initialize and run the JavaScript Standard Style linter and formatter
* Fix the TypeScript style mistakes we made in Part 1

## 0. Prerequisites

### 0.1 Understand Tool Selection: JavaScript Standard Style

While there is no one official linter or formatter for the JavaScript ecosystem,
[StandardJS](https://standardjs.com) comes pretty close.

It is the formatter used by Node/NPM, GitHub, Express, Electron, and many other foundational projects,
has a large, active community of developers and users,
and has an officially-supported TypeScript variant which we will use here.

Most importantly it has zero config!
No options, no bikeshedding, just run it and move on.

If you do not like the formatting choices chosen by StandardJS, keep it to yourself.
Wasting energy on debates and configuration for linters and formatters
is far worse than any particular style oddity could ever be.
God help you if you introduce alternatives like ESLint and Prettier to your team,
with their broken install scripts and configuration black holes
which can only be viewed as intentionally hostile to actually getting work done.
