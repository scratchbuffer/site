+++
type = "article"
title = "Modern TypeScript Project Setup with NPM, Part 1"
description = "Lint and Format TypeScript with ESLint and Prettier"

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
The ecosystem for JavaScript, which was created in two weeks and has cursed us ever since,
has certainly only grown larger and more terrifying since we first encountered in it Part 1.
But this time, we will use some new equipment to help protect us: linting and formatting.

While static type checking provides some protection against the runtime type errors which plague dynamic languages,
it does little to help with enforcing general good coding practices for any given language.

Our goal here within the JavaScript ecosystem
