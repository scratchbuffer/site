+++
title = "Modern TypeScript Project Setup with NPM"
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

## 2. Create a Basic Node.js Application in JavaScript

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

### 2.2 Create a Node Echo Server

Add the following code (expanded upon from the Node.js docs [example](https://nodejs.org/docs/latest/api/synopsis.html#example))
to `src/server.ts` - Node will still run the code regardless of the `.ts` extension:

```js
import {createServer} from 'node:http';

const hostname = '127.0.0.1';
const port = 8080;

const server = createServer(
    (req, res) => {
        let reqBody = '';

        req.on('data', (chunk) => {
            reqBody += chunk.toString();
        });

        req.on('end', () => {
            res.setHeader('Content-Type', 'application/json');

            const response = {
                body: reqBody,
            };

            res.end(JSON.stringify(response));
        });

        req.on('error', (error) => {
            res.statusCode = 500;
            res.end('Internal Server Error');
        });
    }
);

server.listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`);
});
```

### 2.3 Run the Echo Server

Run:
```shell
node src/server.ts
```

The console should output:
```console
Server running at http://127.0.0.1:8080/
```

In another terminal window, use curl to talk to the echo server:
```shell
curl http://localhost:8080 -d 'hello, world'
```

And check the response:
```console
{"body":"hello, world"}
```

### 2.4 Add the Node Run Command to `package.json` Scripts

Fill in the `scripts` key with the command we just used -
I chose `dev-node` to differentiate with Typescript commands we will add later:
```json
"scripts": {
  "dev-js": "node src/server.ts"
},
```

While this command is short, using the `scripts` will help us keep track of all the commands for the project.
Now run:
```shell
npm run dev-js
```

and Node will echo the command before it runs:
```console
> js-ts-npm-starter-demo@0.0.0-alpha.0 dev-js
> node src/server.ts

Server running at http://127.0.0.1:8080/
```

Now we can convert to TypeScript!

## 3. Convert the Application to TypeScript

### 3.1 Install a TypeScript Execution Engine

We can choose between [`ts-node`](https://typestrong.org/ts-node/) and [`tsx`](https://tsx.is/).

The differences are minimal for our current needs - `ts-node` compiles the TypeScript code before running,
while `tsx` skips this step and just runs it.
The compilation may be slow on a large project and may developers prefer the faster startup,
as they already have an IDE plugin checking types as they work.

We can just choose `tsx` for now:
```shell
npm install --save-dev tsx
```

### 3.2 Add Types to the Echo Server Code

### 3.2.1 Import Types Already In Use

While not strictly necessary, we can import the Node request and response types `IncomingMessage` and `ServerResponse`.
We are already using these as the `req` and `res` variables in our handler function, we just could not see them!
Importing the types can help with editor tooling for autocomplete and jumping to type documentation.

First we must install the `@types/node` package to expose the definitions.
It can be a bit tricky - I installed the latest LTS Node version with NVM and got version `24.12.0`,
but if I just run `npm install @types/node`, I will get the latest `@types/node` package which is on version 25.
Further, the `@types/node` package does not release a version for every single minor or patch version of `node`.

Instead, just restrict it to the same major version:

```shell
npm install --save-dev @types/node@24
```

Now we can add types to our Node server handler function.
This line:
```js
(req, res) => {
```
becomes:
```ts
(req: IncomingMessage, res: ServerResponse) => {
```

### 3.2.2 Define Custom Types

We can also add our own types to make the code a bit cleaner -
I am not a fan of having important strings sitting inline in code,
like the `'data'` `'end'` and `'error'` events that we listen for in the handler with `req.on('data', ...)`, etc.
These event definitions from the parent class of `IncomingMessage`, `stream.Readable`,
but a quick scan of the type docs show us that there are no exported constants we can use in place of inline strings.

We can convert these to a TypeScript enum:
```ts
enum StreamEvent {
    DATA = 'data',
    END = 'end',
    ERROR = 'error',
}
```

and replace bits like `req.on('end', ...)` with `req.on(StreamEvent.END, ...)` - and we will get autocomplete!

The full updated code is:
```ts
import {createServer, IncomingMessage, ServerResponse} from 'node:http';

enum StreamEvent {
    DATA = 'data',
    END = 'end',
    ERROR = 'error',
}

const hostname = '127.0.0.1';
const port = 8080;

const server = createServer(
    (req: IncomingMessage, res: ServerResponse) => {
        let reqBody = '';

        req.on(StreamEvent.DATA, (chunk) => {
            reqBody += chunk.toString();
        });

        req.on(StreamEvent.END, () => {
            res.setHeader('Content-Type', 'application/json');

            const response = {
                msg: reqBody,
            };

            res.end(JSON.stringify(response));
        });

        req.on(StreamEvent.ERROR, (error) => {
            res.statusCode = 500;
            res.end('Internal Server Error');
        });
    }
);

server.listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`);
});
```

### 3.3 Run the Echo Server with `tsx`

If `tsx` is not installed globally, the command may not be available directly from the command line.
Instead we can use Node's built-in `npx` which runs arbitrary commands from an npm package:

```shell
npx tsx src/server.ts
```

That will run the server, which we can check again with `curl`:
```shell
curl http://localhost:8080 -d 'hello, world'
```

Finally, we can add the command to the `package.json` scripts as we did before.
When using `npm run` with these scripts, we do not need `npx` as long as the package we are calling is installed:

```ts
"scripts": {
    "dev-js": "node src/server.ts",
    "dev-ts": "tsx ./src/server.ts"
  },
```

As before, `npm run dev-ts` will echo the command:
```console
> js-ts-npm-starter-demo@0.0.0-alpha.0 dev-ts
> tsx ./src/server.ts

Server running at http://127.0.0.1:8080/
```
