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
and has an officially-supported [TypeScript variant](https://standardjs.com/#typescript) which we will use here.

Most importantly it has zero config!
No options, no bikeshedding, just run it and move on.

If you do not like the formatting choices chosen by StandardJS, keep it to yourself.
Wasting energy on debates and configuration for linters and formatters
is far worse than any particular style oddity could ever be.
Please do not introduce alternatives like ESLint and Prettier to your team,
with their broken install scripts and configuration black holes
which can only be viewed as intentionally hostile to actually getting work done.

### 0.2 A TypeScript Project with NPM

We will use the project initialized in Part 1 -
a simple Node echo server written in TypeScript,
with standard NPM Package management.

Jump to the final [server code](/topics/web-development/typescript-npm-setup-part-1/#322-define-custom-types)
and the final [`package.json` and `tsconfig.json`](http://localhost:1313/topics/web-development/typescript-npm-setup-part-1/#5-review-config-files)
to see our starting point.

## 1. Install and Run the StandardJS TypeScript Variant

### 1.1 Install `ts-standard`

Run:
```shell
npm install --save-dev ts-standard
```

### 1.2 Run `ts-standard` to View Issues

From the project root or the same directory as the `tsconfig.json` file, run:
```shell
npx ts-standard
```

If you are starting with the exact code from Part 1,
this will spit out a _ton_ of messages - mostly regarding formatting issues.
The original code used 4 spaces for indentation,
among other choices which do not match the StandardJS formatting spec.

The last lines of the output will look something like this:
```console
<project root>/src/server.ts:35:2: Extra semicolon. (@typescript-eslint/semi)
<project root>/src/server.ts:38:1: Expected indentation of 2 spaces but found 4. (@typescript-eslint/indent)
<project root>/src/server.ts:38:65: Extra semicolon. (@typescript-eslint/semi)
<project root>/src/server.ts:39:3: Extra semicolon. (@typescript-eslint/semi)
<project root>/src/server.ts:39:4: Newline required at end of file but not found. (eol-last)
```

Most of the issues detected can be fixed automatically!

### 1.3 Run `ts-standard --fix` to Auto-Fix Issues

This will fix all the formatting issues:
```shell
ts-standard --fix
```

Now the output is left only with those issues which cannot be fixed automatically:
```console
<project root>/src/server.ts:17:18: Invalid operand for a '+' operation. Operands must each be a number or string. Got `any`. (@typescript-eslint/restrict-plus-operands)
<project root>/src/server.ts:30:31: Expected error to be handled. (n/handle-callback-err)
```

These will be up to us to fix manually.

### 1.4 Add Lint Commands to `package.json` Scripts

Before we move on, we can add these commands for easier access later:
```json
  "scripts": {
    "dev": "tsx ./src/server.ts",
    "tsc": "tsc --noEmit",
    "lint": "npx ts-standard",
    "lint-fix": "npx ts-standard --fix"
  },
```

## 2. Fix TypeScript Lint Issues

### 2.1 Handle the Unhandled TypeScript Error

The simpler of the two remaining lint issues is `server.ts:30:31: Expected error to be handled.`.
This refers to the lines:
```ts
    req.on(StreamEvent.ERROR, (error) => {
      res.statusCode = 500
      res.end('Internal Server Error')
    })
```

We returned an HTTP error code and a generic message to the caller,
but did not do anything with the `error` the server received.

We do not have any experience yet with what form these internal server error details may take,
so we should avoid sending those gory details (and potentially sensitive information) to the client.

Instead we can log it:
```ts
    req.on(StreamEvent.ERROR, (error) => {
      console.error(
        'Request error:',
        `err=${error.name}`,
        `msg=${error.message}`
      )

      res.statusCode = 500
      res.end('Internal Server Error')
    })
  }
```

Run `ts-standard` again and the lint message will have disappeared.

### 2.1 Handle the Operand Type Error

The final lint issue is ``server.ts:17:18: Invalid operand for a '+' operation.
Operands must each be a number or string. Got `any`.``

This refers to the lines:
```ts
    req.on(StreamEvent.DATA, (chunk) => {
      reqBody += chunk.toString()
    })
```

JavaScript will allow you to use the `+` operator on many different types
but the resulting behavior can be quite unexpected,
so these lints try to prevent this usage from happening at all.

Despite the fact that we call `toString` on the `chunk` object,
the linter is still not certain if we are correctly adding a string to a string.

Do we wish that the Node type definitions had given `chunk` a more specific type than `any`?
Do we wish the lint rules recognized that `toString` does return a string even though `chunk` is `any`?
Do we wish there was only one correct way to fix this?

Well too bad, this is JavaScript baby.
This can be fixed a few different ways.

**Option 1: Use `concat` instead of `+`**

This is probably the simplest and most direct.
The `String.concat` method does not have the oddities of the `+` operator so the linter will not complain:
```ts
    req.on(StreamEvent.DATA, (chunk: any) => {
      reqBody.concat(chunk.toString())
    })
```

**Option 2: Specify `chunk` type**

If we poke around the Node type definitions a bit,
we can find that it also uses the type `Buffer` for the `req.on('data')` argument.
Indicate that in the callback signature:
```ts
    req.on(StreamEvent.DATA, (chunk: Buffer) => {
      reqBody += chunk.toString()
    })
```

**Option 2: Declare variable for `chunk.toString()`**

If we first assign the `toString` output to a variable and declare its type,
the linter will recognize it as a string when appending it to `reqBody`:
```ts
    req.on(StreamEvent.DATA, (chunk: any) => {
      const bodyStr: string = chunk.toString()
      reqBody += bodyStr
    })
```

**Option 4: Specify `chunk.toString()` type in-line**

```ts
    req.on(StreamEvent.DATA, (chunk: any) => {
      reqBody += (chunk.toString() as string)
    })
```

Take your pick and run the linter again - all issues should be resolved!

## Conclusion

We have survived another adventure into the anomalous zone,
this time with some extra protective gear in the form of a standardized linter and formatter.

With no config files and autofixes for most of our issues,
all we had to do was learn a bit about the types we are using to fix up our code.

Our teammates, git history, and future selves will thank us for setting this system up,
and now we can finally go forth and focus on actually producing working software.
