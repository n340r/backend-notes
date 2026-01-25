# Fundraise Up (Full-stack)

## Arrow functions vs regular functions.

## What happens if you try using `.bind()`, `.call()`, `.apply()` on an arrow function ?

Nothing, it will just fail with an error.

## Is it true that async function's body always runs asynchronously ?

- **No** unless we have `await` to wait for some asynchronous task.
- If we for instance have only `console.log()` inside async function, they will be executed synchronously.
- If we put `await` between those `console.log()` then it will run like so:

```ts
async function example() {
  console.log("1: start");
  await Promise.resolve(); // async pause here
  console.log("2: after await"); // this runs later
}
```

## Ways of running code asynchronously

- **setTimeout(function, delay)**;
- **setInterval(function, delay)**
- **new Promise((res, rej) => {});**
- **queueMicrotask()**
- **WebWorkers**, **Worker Threads**

## Types of workers in a browser

- Web Worker
- Shared Worker
- Service Worker

## Workers vs Worker threads in NodeJS

## Micro-tasks, Macro-tasks

## Heavy computationon a client

We need to process billions of rows of data. How do we optimmize ?  
**Solution:**
Break it into chunks and process one at a time.  
**setTimeout(() => {})** with pieces inside. We cannot spam microtask queue because ui will freeze.  
But we can add stuff to macrotask quueue so that event loop can process things in the meantime

## Storages in a browser

## Cookies and their flags

## LibUV and what it does in Node.js

## CSRF, CSRF Tokens

## Race condition

here is a race conditions task

context:
we are making microservice that does user's finance data. Microservice works on http
When high load, multiple requests change **same use balance** (because race condition)
why this happens and how do we fix.

**Solution (DB level)**

- Isolation levels in PostgreSQL
- Atomic updates with $inc

**Solution (App level)**
async-mutex

## Indexes in mongodb, PostgresSQL

## Heavy api endpoint

lets say we have a server, fastify

this server uses one thread (dont use pm2)

We only do this:
node index.js and it runs

on fastify we have two endpoints
one **POST is easy** (takes less time)
second **GET has complicated** synchronous logic (count things, CSV process) - it takes lots of time

how do we fix

will post wait ?
are thay parallel

**Solution:**

- App will freze because node is single threaded
- use `pipe stream` to read in one place and send to user in chunks.
- Stream works with GET requests !

What if we hae a zipped archive ?

**Solution:**
Then we will not be able to send it because of zipped file structure.
