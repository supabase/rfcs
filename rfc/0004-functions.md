- Feature Name: `functions`
- Start Date: 2021-12-03
- RFC PR: [supabase/rfcs#0004](https://github.com/supabase/rfcs/pull/0000)

# Summary
[summary]: #summary

Outlines various options for implementing Supabase Functions.

# Motivation
[motivation]: #motivation

Firebase offers [Cloud Functions](https://firebase.google.com/docs/functions/), a serverless framework that lets you automatically run backend code in response to events triggered by Firebase features and HTTPS requests. As a Firebase alternative we aim to provide an offering which is as good, or better. Functions is one of our most requested features.

While we could copy the design of Firebase Functions exactly, we also have an opportunity to provide a better experience, and a more flexible implementation - similar to our choice of PostgreSQL over a NoSQL database. 

This RFC will function more as a Design Document - exploring various potential options, and the pros and cons of each.

# Roadmap / Blockers

For all of these options, there are a few blockers that we need to clear before proceeding.

- Need to have
  - API + CLI (in progress). Developers need to authenticate their local machine to deploy their functions to the cloud.
  - Async Triggers / Function Hooks (in progress) - Ideally we would sign the payload before triggering an external function.
  - Secrets management / vault (in progress) - a system to store keys/secrets, used to 
  - Builders on the cloud (not started). Developers should be able to push code to the cloud to be built before running.
- Nice to have
  - Realtime logs (in progress). Developers should be able to debug their Functions on the platform.
  - Domains (not started) - there may be some cases where developers need their functions on their own domains. 
  - Git Integration (not started). Developers should be able to associate repositories with Supabase projects, and there should be a "function deployment per-branch".

# Option 1: Don't do Functions (partner instead)

Worth considering in any RFC.

Developer platforms like Vercel and Netlify already provide a phenomenal developer experience around Functions. There has been a lot of investment put into these platforms, and we could work with any or all of these - building tight integrations. This is great for the whole ecosystem.

### Pros 

- We can focus on what we're good at (Databases) and make faster progress on some of the other outstanding features like database branching.
- Cost. Much cheaper for Supabase :)

### Cons

- Supabase developers need to use third-party tools.

# Option 2: Isolates

The ability to run very isolated pieces of code on th edge. This is the most common design for functions.

### Prior Art

[Firebase Functions](https://firebase.google.com/docs/functions/), [Cloudflare workers](https://workers.cloudflare.com/), [Deno Deploy](https://deno.com/deploy/docs), [Vercel Edge Functions](https://vercel.com/features/edge-functions), [Netlify Functions](https://www.netlify.com/products/functions/), [AWS Lambda](https://aws.amazon.com/lambda/).

### Pros

- Cheap. 
- Easiest of all the options in terms of design, since we can follow the design of any of the other systems.

### Cons

- Hard to run locally. 
- No portability. There seems to be a lot of platform lock in for each of these providers.
- Not multi-lingual - some cloud providers only support Node.js. Ideally we can support all languages.

# Option 3: Containers

Run functions inside full Docker containers. We are leaning heavily towards this option, and have a separate RFC here with the details of how we would implement this.

## Prior Art

Open FaaS, Google Cloud Run, Fly.io

### Pros

- Extremely extensible: this can be used for _almost anything_. While this model could start as just a function set up, it could be used for data loaders.
- Extremely portable: docker is supported everywhere.
- Immediately open source: can run on Kubernetes, Docker, etc.

### Cons

- We probably need to build a framework which "feels" like developers can write.
- Cost. Depending on the design/provider, this could be very expensive, especially if we are running containers on every branch. We would need a way to "pause" containers, at least for non-production use-cases.
- Local DX is hard - developing inside docker images isn't great.

# Option 4: Extend Postgres Functions

PostgreSQL already supports Functions, which Supabase developers can call using `rpc()`. We could focus on improving this option to the point where the developer experience is on-par with Firebase Functions.

### Prior Art

[Postgres Functions](https://supabase.com/docs/guides/database/functions), [SupaScript](https://github.com/burggraf/SupaScript) - [Demo](https://www.youtube.com/watch?v=ywdRAjPhTbg&t=3s), [postgres-deno](https://github.com/supabase/postgres-deno/issues/1)


### Pros 

- We already manage a database, so we have the infrastucture deployed
- No cold-starts.

### Cons

- Hard to support multiple languages. At the moment we only have `plv8` for running Node.js functions, and even that has limitations.
- Not on the edge. The function runs inside the database, so latency is high unless the database itself is running at the edge.
- Database load. Functions are often used for computationally heavy actions, which might be better to run outside of the database itself. 

# Option 5: Workflows

We originally intended to solve this with [workflows](https://supabase.com/blog/2021/04/02/supabase-workflows), but internally we decided to clear 2 blockers first: Realtime Row Level Security (now done), and turning Realtime into a multi-tenant system to save costs (not completed).

Workflows are like "smart" functions. They can be used as queues or cron jobs, they can sleep and wake up, and they can be expressed in a declarative language.

### Prior Art

[Supabase Workflows PR](https://github.com/supabase/realtime/pull/161#issuecomment-859611164), [Step Functions](https://aws.amazon.com/step-functions/), [Temporal](https://temporal.io/), [Zapier](https://zapier.com/).


### Pros

- Can be used by non-developers. It's easy for us to produce a no/low-code UI for this.
- This is an extremely under-served part of the market for our segment. While there are a lot of platforms focused on Functions, not many are focused on queues and workflows. 
- A lot of integration opportunities. Think "open source zapier".

### Cons

- The workflow engine is a designed for _coordinating_ functions, not for _executing_ them. Although, in theory, we could design some sort of execution engine into the workflow engine, perhaps using [WASM](https://github.com/tessi/wasmex)
- Also expensive - we would need to finish our work on multi-tenant Realtime first.


