- Feature Name: `functions_with_containers`
- Start Date: 2021-12-03)
- RFC PR: [supabase/rfcs#0005](https://github.com/supabase/rfcs/pull/0005)

# Summary
[summary]: #summary

Outlines our design of Functions with containers.

Watch a demonstration:

# Motivation
[motivation]: #motivation

Firebase offers Cloud Functions, a serverless framework that lets you automatically run backend code in response to events triggered by Firebase features and HTTPS requests. As a Firebase alternative we aim to provide an offering which is as good, or better. Functions is one of our most requested features.

While we could copy the design of Firebase Functions exactly, we also have an opportunity to provide a better experience, and a more flexible implementation - similar to our choice of PostgreSQL over a NoSQL database.

This RFC will function as a Design Document for one of the implementations we are considering using containers. You can find the other options we're considering in [this RFC](https://github.com/supabase/rfcs/blob/rfc/functions/rfc/0004-functions.md).

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Even though we will be explaining how to implement "Functions" in containers, what we will actually be designing is something akin to [Service oriented architecture](https://en.wikipedia.org/wiki/Service-oriented_architecture). For the beginner developer, they don't need to know that they can do anything beyond simple Functions. But as their application grows, we will have "prepared' them use some best practices that Supabase, as developers ourselves, have used for years.

The design:

- is multi-lingual and polyglot - each service can be written in any language.
- has good separation of concerns - you can split logic into separate services.
- can be used for anything that can be containerized. It has application beyond simple functions.


# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Init a new project

Start a new Supabase project

```bash
supabase init
```

This will create a folder structure like this

```bash
supabase/
├── config.json
├── migrations
│   └── 20211021190022_init.sql
└── seed.sql
```

## Creating new Services

Then they can generate "services"

```bash
supabase service new --template deno my-deno-example
# and
supabase service new --template fastify my-fastify-example
```

This command generates a new service inside `supabase/services`

```bash
supabase/
├── services
│   ├── my-deno-example     # HERE
│   └── my-fastify-example  # HERE (expanded)
│   │   ├── docker.base.yaml   ## Base image 
│   │   ├── docker.prod.yaml   ## Production build
│   │   └── docker.dev.yaml    ## Has hot reloading
├── config.json
├── migrations
│   └── 20211021190022_init.sql
└── seed.sql
```

Each folder is an entire application, wrapped inside a single docker container


## Default Service

In order to make life easy for developers, when they `init` a project it should add a default app called `service` called `api`. 

```bash
supabase/
├── service
│   └── api  # HERE
│   │   ├── src 
│   │   ├── /routes 
│   │   ├── /routes/hello-world.js 
│   │   ├── docker.base.yaml   ## Base image 
│   │   ├── docker.prod.yaml   ## Production build
│   │   └── docker.dev.yaml    ## Has hot reloading
├── config.json
├── migrations
│   └── 20211021190022_init.sql
└── seed.sql
```
## Running locally

The docker container is added to the `docker-compose` so that when they run `supabase start`, it picks up the application in dev mode (code refreshes automatically)


## Deploy

When developers have completed development they can run

```bash
supabase deploy
# or 
supabase deploy --service my-fastify-example
```

Now we can add the app to their service routing (in this case the app is called `api`)

`https://{project_ref}.supabase.co/services/api`

So if they have some fastify routes like `/countries` and `/cities` they can hit:

- `https://{project_ref}.supabase.co/services/api/countries`
- `https://{project_ref}.supabase.co/services/api/cities`



## Invoking

We can create a new client library for invoking functions inside an services, and for communicating across services.

```js
// REST: get
supabase
.service('api')
.get('/customers?id=1' { headers })

// REST: post
supabase
.service('api')
.post('/customers', { headers, data: { name: 'Fred' } ))

// REST: put
supabase
.service('api')
.put('/customers', { headers, data: { id: 1, name: 'Wilma' } })

// REST: patch
supabase
.service('api')
.patch('/customers?id=1', { data: { name: 'Barney' } })

// REST: delete
supabase
.service('api')
.delete('/customers?id=1', { })

// call a function within a secondary app "stripe"
supabase
.service('stripe')
.get('/customers', { })

// websocket ?
supabase
.service('api')
.subscribe('/customers', { })  // connect?


supabase.service.list() // Get a list of apps
```

## Dashboard

- If the App exposes a JSON Schema at the root, then we could also provide auto-generated documentation, generate types etc
- We should add logging per-route (in this case it's simply an HTTP log)


# Unresolved questions
[unresolved-questions]: #unresolved-questions

- How long should we allow these to run? Should we pause after no activity?
- Should we deploy globally?
- How to we handle periodic jobs/cron etc if we scale to Zero?
- Should the apps be allowed to return HTML? (most likely not, as we have been burned with abuse of this in our Storage product)
- what is the authorization model? will the anon key be able to execute any function?

# Future possibilities
[future-possibilities]: #future-possibilities

Because these are just Docker containers, there are plenty of possibilities:

Developers could:

- Run the Stripe Sync Engine: https://github.com/supabase/stripe-sync-engine
- Run pgloader to load data into their projects: https://github.com/dimitri/pgloader
- Run services like Redis for custom caching

Supabase could provide a marketplace-type product which would make it easy to deploy these types of products.