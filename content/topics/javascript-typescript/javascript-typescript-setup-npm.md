+++
title = "Modern TypeScript Project Setup with NPM, Part 1"
description = "Write, Run, and Compile TypeScript with Node.js, `tsx`, and `tsc`"

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

We will:

* Initialize and configure a basic Node.js project with NPM
* Utilize existing TypeScript types from Node packages and define our own custom types
* Run the TypeScript application with `tsx`
* Configure the `tsc` compiler to check and compile our TypeScript code

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

While not strictly necessary, we can import the type definitions which are already using from the Node `http` package.
Declaring the types explicitly can help with editor tooling for autocomplete and jumping to type documentation,
as well as make the code easier to understand.

First, we must install the `@types/node` package to expose the definitions.
This can be a bit tricky - I installed the latest LTS Node version with NVM and got version `24.12.0`,
but if I just run `npm install @types/node`, I will get the latest `@types/node` package which is on version 25.
Further, most type definitions are not updated for every single minor or patch version of the associated package.

Instead, just restrict the types package to the same major version:

```shell
npm install --save-dev @types/node@24
```

Now we can add types to our Node server handler function.
The types for the `req` and `res` variables in our handler function are `IncomingMessage` and `ServerResponse`.

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

```json
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

Note that we still have not _compiled_ the code - that is next!

## 4. Install, Configure, and Run the TypeScript Compiler

### 4.1 Install TypeScript

Run:

```shell
npm install --save-dev typescript
```

Check that it is working with `npx` again - use `--noEmit` to skip actually writing the compiler output to files:

```shell
npx tsc --noEmit src/server.ts
```

This should not produce any output, as the project has no errors.
We can check how the errors would look by commenting out the first line of code with our type imports,
then running again the `tsc` command again.
This time we should see errors related to those missing types:

```console
src/server.ts:12:16 - error TS2304: Cannot find name 'createServer'.

12 const server = createServer(
                  ~~~~~~~~~~~~

src/server.ts:13:11 - error TS2304: Cannot find name 'IncomingMessage'.

13     (req: IncomingMessage, res: ServerResponse) => {
             ~~~~~~~~~~~~~~~

src/server.ts:13:33 - error TS2304: Cannot find name 'ServerResponse'.

13     (req: IncomingMessage, res: ServerResponse) => {
                                   ~~~~~~~~~~~~~~
```

### 4.2 Initialize TypeScript Compiler Config

#### 4.2.1 Init Default Config

We need a `tsconfing.json` to unlock all the power of the compiler.

Run:
```shell
tsc --init
```

This creates a `tsconfig.json` file with some sensible defaults and hints on other config options we may want.
Note that this is a JSONC file, meaning a flavor of JSON which supports comments.

#### 4.2.2 Configure Input and Output Directories

The first few lines offer us a hint of how to target files for compilation:
```jsonc
{
  // Visit https://aka.ms/tsconfig to read more about this file
  "compilerOptions": {
    // File Layout
    // "rootDir": "./src",
    // "outDir": "./dist",
// ...
```

We can uncomment both the `rootDir` and `outDir` keys to enable those settings:
```jsonc
{
  // Visit https://aka.ms/tsconfig to read more about this file
  "compilerOptions": {
    // File Layout
    "rootDir": "./src",
    "outDir": "./dist",
// ...
```

As noted in the [`tsconfig` reference](https://www.typescriptlang.org/tsconfig/):

> Importantly, rootDir does not affect which files become part of the compilation.
> It has no interaction with the `include`, `exclude`, or `files` `tsconfig.json` settings.

If we have any other TypeScript files which we do not want to include (likely config files like `eslint.config.ts`),
then we can use the `exclude` key to skip them.
We also want to exclude the compiler from looking at code that it previously built,
meaning we need to exclude the `dist`  directory we just told it to write to.
The `exclude` key is _not_ a compiler option, so do not nest it in where we just worked on `rootDir` and `outDir`.

It can just go at the beginning or end of the file for now:

```jsonc
{
  // Visit https://aka.ms/tsconfig to read more about this file
  "exclude": [
    "./dist",
    "eslint.config.ts"
  ],
  "compilerOptions": {
// ...
```

#### 4.2.3 Configure Type Discovery

While TypeScript attempts to include any types found in `node_modules/@types`,
this does not always work by default due to oddities in how packages declare their types.
The default `tsconfig` offers a hint about this:

```jsonc
    "types": [],
    // For nodejs:
    // "lib": ["esnext"],
    // "types": ["node"],
    // and npm install -D @types/node
```

We already installed `@types/node` before, so we can just uncomment the suggestion:

```jsonc
    // For nodejs:
    // "lib": ["esnext"],
    "types": [
      "node"
    ],
```

We are not using TypeScript's built-in types yet, but we would want to uncomment the `lib` key as well if we did.
It is worth taking a look at the config definitions for
[`lib`](https://www.typescriptlang.org/tsconfig/#lib),
[`types`](https://www.typescriptlang.org/tsconfig/#types),
and [`typeRoots`](https://www.typescriptlang.org/tsconfig/#typeRoots).

Now we are ready to compile!

### 4.3 Compile!

Now we can drop the `--noEmit` flag and actually produce the output:

```shell
npx tsc
```

We should get no errors in the console and see the files appear in the `dist` directory -
run `tree dist` or `ls dist` to check:

```console
dist
├── server.d.ts
├── server.d.ts.map
├── server.js
└── server.js.map
```

More complex projects or those building for frontend components often require capabilities beyond what `tsc` offers -
this is the realm of "bundlers" like Babel, Webpack, and Vite.
Each of these has its own strengths and idiosyncrasies which are beyond the scope of this guide.

In any case, `tsc --noEmit` is still often used as a quick type _checker_.

We can add this to our `package.json`:

```json
"scripts": {
    "dev": "tsx ./src/server.ts",
    "tsc": "tsc --noEmit"
  },
```
and run it as we have before:

```shell
npm run tsc
```

## 5. Review Config Files

Our final `package.json` looks something like:
```json
{
  "name": "js-ts-npm-starter-demo",
  "version": "0.0.0-alpha.0",
  "license": "MIT",
  "type": "module",
  "scripts": {
    "dev": "tsx ./src/server.ts",
    "tsc": "tsc --noEmit"
  },
  "dependencies": {
    "node": "^24.12.0"
  },
  "devDependencies": {
    "@types/node": "^24.10.4",
    "tsx": "^4.21.0",
    "typescript": "^5.9.3"
  }
}
```

And `tsconfig.json` looks like:
```jsonc
{
  // Visit https://aka.ms/tsconfig to read more about this file
  "exclude": [
    "./dist",
    "eslint.config.ts"
  ],
  "compilerOptions": {
    // File Layout
    "rootDir": "./src",
    "outDir": "./dist",
    // Environment Settings
    // See also https://aka.ms/tsconfig/module
    "module": "nodenext",
    "target": "esnext",
    // For nodejs:
    // "lib": ["esnext"],
    "types": [
      "node"
    ],
    // and npm install -D @types/node

    // ...other default values from tsc --init
  }
} // end of file
```

## Conclusion

We have entered the anomalous zone and somehow made it back,
with a working Node.js app written in TypeScript - checked, and compiled, and run!

This of course is only the beginning, and the terrors of the JavaScript ecosystem still stretch before us endlessly.

I can only hope that by building a simple project from the ground up
we have gained some understanding of the fundamental structure and tools at play and how they all fit together.
