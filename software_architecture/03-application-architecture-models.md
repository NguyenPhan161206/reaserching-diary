# Application Architecture Models - Monolithic, Multi-Tier, Microservices, Serverless

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Architecture`, `#Monolith`, `#MultiTier`, `#Microservices`, `#Serverless`, `#SystemArchitecture`, `#Express`, `#PostgreSQL`
**References:** [martinfowler.com - Microservices](https://martinfowler.com/articles/microservices.html), [martinfowler.com - MonolithFirst](https://martinfowler.com/bliki/MonolithFirst.html); Kleppmann, *Designing Data-Intensive Applications* (O'Reilly); [AWS Serverless documentation](https://aws.amazon.com/serverless/); the course syllabus 1.4 (Monolithic, Multi-tier, Microservices, Serverless)
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`02-modular-monolith-vs-microservices.md`](02-modular-monolith-vs-microservices.md) (boundaries and contracts); [`../dev_foundation/web_server/01-static-dynamic-ssr-vs-csr.md`](../dev_foundation/web_server/01-static-dynamic-ssr-vs-csr.md) (how a request is rendered)

---

## Key Takeaways (TL;DR)

- **Architecture is the division of responsibilities and how the parts communicate.** The four syllabus models are points on a *spectrum*, not a ladder to climb - more parts does not mean better.
- **A monolith is one deployable containing UI, logic, and data access.** It is the correct default: one transaction, no network hops, easy debugging. The course project is a monolith.
- **Multi-tier separates presentation / application / data.** The course app *is already 3-tier*: browser + Nginx -> Express -> PostgreSQL. You can scale or replace one tier without rewriting the others.
- **Microservices are many independently deployable services, each owning a data slice.** They buy independent scaling and deployment at the price of distributed transactions, network failure, and heavy operations.
- **Serverless (FaaS) runs functions, auto-scales, and bills per invocation - but it is stateless, suffers cold starts, and creates a database-connection nightmare.** Choose it for bursty, glue, and inference endpoints, not for session-heavy work.

---

## Detailed Notes

### 1. Core Concepts - Responsibilities and Communication

Architecture is not the list of technologies you use. It is the answer to two questions:

1. **What are the responsibilities, and who owns each one?** (UI, business logic, data access, persistence.)
2. **How do the parts communicate?** (in-process function calls, HTTP, events.)

The four course models differ only in how they answer those questions. Draw them as one picture, from "everything in one process" to "everything is a function":

```
 MONOLITH            MULTI-TIER              MICROSERVICES           SERVERLESS
                     
  UI               Presentation        Auth User Post    fn fn fn
  logic            (nginx, HTML)               
  data          Application                                  
                   (Express)                   
        Data                DB  DB  DB     DB  DB  DB 
 one process         (PostgreSQL)                
                          one DB per service     managed runtime
 1 deploy           3 logical tiers         many deployables        event-triggered
```

Read the picture left to right: each step moves from **in-process calls** (free, fast) to **network calls** (slow, failure-prone, independently deployable). You are trading simplicity for independence. That trade is the whole subject.

> **Why it matters:** The syllabus offers these as a taxonomy to remember, but the real skill is judgement: knowing that the *course project* should be a multi-tier monolith, and that "let us use microservices" is usually a decision someone regrets before they have a scaling problem.

---

### 2. Monolithic - One Deployable, All Responsibilities

A monolithic application packages the UI, business logic, and data access into **one deployable unit** (one Express process, one artifact, one deploy).

```
 
              THE MONOLITH                   
   routes  controllers  services       
                                            
                                            
              templates      data access      
                                             
 
                                  
                            PostgreSQL
```

**Pros - and they are real, not consolation prizes:**

- **Simplicity.** One repo, one build, one deploy, one log stream. A new developer runs `npm install && npm start` and has the whole system.
- **One transaction.** The classic checkout flow - create order -> decrement stock -> record payment - is a single ACID transaction because everything touches one database from one process. In a distributed system this becomes a saga with compensations.
- **No network hops between internal calls.** Calling your `orders` service is a function call, measured in microseconds, not an HTTP round-trip that can fail.
- **Easy debugging.** A stack trace runs from the route to the query in one place. There is no correlation-id hunting across services.

**Cons:**

- **Scale as a unit.** You cannot scale just the expensive report endpoint - you scale the whole process.
- **Team coupling.** Everyone commits to the same deployable, so teams step on each other.
- **Fear of deployment.** One broken module blocks the release of every other module.

**When it is the RIGHT call:** small teams, an early product, a course project, or any system where the domain fits in one process. The course app is a textbook monolith, and that is correct.

```js
// app.js - a monolith: routing, logic, and data access in one process
const express = require('express');
const knex = require('./db/knex');
const app = express();

app.get('/orders/:id', async (req, res) => {
  // logic and data access are in-process; one transaction would span both
  const order = await knex('orders').where({ id: req.params.id }).first();
  res.render('orders/show', { order });
});

app.listen(3000);
```

---

### 3. Multi-Tier - The Course Project, Drawn Exactly

**Multi-tier** splits responsibilities into layers: **presentation**, **application**, and **data**. The tiers can be logical (folders in one process) or physical (separate processes/machines). The course project is physically three-tier:

```
 
  PRESENTATION TIER              browser (HTML/CSS/JS)
   browser + Nginx               + nginx: static files, TLS, proxy
 
                 HTTP (proxy_pass)
                
 
  APPLICATION TIER               Express: routes, controllers,
   Express (SSR)                 sessions, auth, template rendering
 
                 SQL (pg / Knex pool)
                
 
  DATA TIER                      PostgreSQL: tables, indexes,
   PostgreSQL                    transactions, constraints
 
```

**The value:** each tier evolves and scales independently.

- Need more requests per second? Run more Express instances behind nginx (scale the application tier).
- Need faster asset delivery? Tune nginx and its cache (presentation tier).
- Need faster queries? Index the database or add a replica (data tier).

The interface between tiers is a **contract**: HTTP between presentation and application, SQL between application and data. Because the contracts are narrow, you can replace a tier - swap nginx for a CDN, PostgreSQL for another store - without rewriting the others. This is the same "boundary plus contract" discipline behind the modular monolith. [-> `02-modular-monolith-vs-microservices.md`](02-modular-monolith-vs-microservices.md)

**Where it lives in the repo:** the deploy topology - nginx, Express, Postgres wired by Docker Compose - is exactly a 3-tier deployment in files. [-> `../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md`](../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md)

---

### 4. Microservices - Independence at an Operational Price

**Microservices** split the system into many small services, each **independently deployable and scalable**, each **owning its own data slice**. Services communicate over the network (HTTP/gRPC) or asynchronously via events.

```
        client
          
    
               
       HTTP/events
 Auth     User    
                      Post  
                                    
                         
 authDB   userDB                  
                      postDB
                                      
```

**What you buy:** independent deployment (ship the auth service without touching posts), independent scaling (scale only the hot service), fault isolation (one service can fail without taking everything down), and team autonomy (a team owns a service end to end).

**What you pay - do not skip this column:**

- **Distributed transactions.** The single ACID transaction of the monolith becomes a saga: multi-step workflows with compensations when a step fails. Correctness is now your problem, not the database's.
- **Network failure.** Every internal call can time out, retry, or partially apply. You need retries, idempotency, timeouts, and circuit breakers.
- **Operations.** Each service needs its own deploys, logs, metrics, tracing, and secrets. Monitoring cost scales with service count.
- **Contract versioning.** Changing an API breaks consumers; you need SemVer, backward compatibility, and a migration window.
- **Distributed debugging.** The stack trace is gone - you chase a request across services with correlation IDs.

This note is deliberately the *map*; the deep decision framework - modular monolith vs microservices, bounded contexts, fitness functions, and the distributed-monolith anti-pattern - lives in note 02. **Read that note for the "when and how to split."** [-> `02-modular-monolith-vs-microservices.md`](02-modular-monolith-vs-microservices.md)

**The monolith-first rule (Martin Fowler):** almost every successful microservices system began as a monolith that outgrew its shape, then extracted services one at a time. Starting with microservices means paying the distributed-systems tax *before* you have revenue, users, or a team large enough to need it. **Premature splitting is the most expensive mistake in this space.**

---

### 5. Serverless (FaaS) - Functions as the Unit of Deployment

In **serverless / Function-as-a-Service**, you deploy individual functions, not servers. The provider (AWS Lambda, Cloudflare Workers, Google Cloud Functions) runs them on demand, scales automatically, and bills per invocation and execution time.

```
 event (HTTP request / queue msg / file upload)
        
        
    spin up (or reuse) a container,
   your function    run to completion, return, scale to zero
 
         
   managed runtime - you never see the server
```

**Properties:**

- **Auto-scaling to zero.** No traffic, no cost; a spike, and the platform spins up thousands of instances. You pay per invocation, not for idle capacity.
- **Stateless.** A function instance may be created or destroyed at any time. Session state cannot live in memory - it goes in a database, Redis, or an external store.
- **Cold starts.** The first invocation after idle must boot the runtime and load your code, adding tens to hundreds of milliseconds (worse for large dependencies and JVM/.NET runtimes).

**When it suits:** bursty or unpredictable traffic, glue code (resize an image on upload, send a webhook, transform a record), scheduled jobs, and **ML inference endpoints** where the model server scales to zero between bursts.

**When it bites:**

- **Stateful / session-heavy work.** In-memory sessions are impossible; you pay a round-trip to external state on every request.
- **Long-running jobs.** Platform execution time limits (e.g. ~15 minutes) kill batch work; you need a queue plus workers.
- **The database-connection nightmare.** Each function instance opens its own DB connection. Under load, thousands of instances try to connect at once and exceed Postgres's `max_connections` - the pool-exhaustion failure from the database notes. The fix is almost always a **connection proxy (PgBouncer, RDS Proxy)** between functions and the database. [-> `../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md`](../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md)

```js
// a serverless handler - stateless by contract; state lives elsewhere
export const handler = async (event) => {
  const db = await getPooledClient();        // proxied, not a fresh connection
  const rows = await db.query('SELECT ...');
  return { statusCode: 200, body: JSON.stringify(rows) };
};
```

> **Why it matters:** Serverless is often the fastest path to shipping an AI inference endpoint - but the connection-pooling trap means it is only fast if the database is proxied. That single interaction between two "independent" decisions is the kind of thing senior engineers check automatically.

---

### 6. Comparison Table - One Axis at a Time

| Criterion | Monolithic | Multi-tier | Microservices | Serverless (FaaS) |
|-----------|-----------|------------|---------------|-------------------|
| Deployment unit | one process | one process (layered) | many services | one function |
| Scaling | scale the whole app | scale each tier | scale each service | automatic, per invocation |
| State | in-process, simple | app tier may hold sessions | per-service state, external | stateless by contract |
| Cost profile | pay for always-on server | pay per tier | pay per service host | pay per invocation (scale to zero) |
| Operational complexity | low | lowmedium | high | low ops, hidden limits |
| Team-size fit | 110 | 320 | 20+ (teams per service) | 150 (event-driven) |
| Independent deploy | no | tiers yes, app no | yes | yes (per function) |
| Transactions | single ACID tx | single ACID tx (same DB) | distributed (saga) | often per-invocation |
| Network between parts | none | one hop per tier | many hops | platform-invoked |
| Failure blast radius | whole app | whole app tier | isolated service | isolated invocation |
| Cold start | none | none | none | yes |
| Testing | easy end-to-end | easy end-to-end | contract + integration | per-function |
| Best for | prototypes, course project | web apps, SSR | large orgs, hot services | bursty glue, inference |

**Decision table - which model answers which question:**

| Your situation | Reach for | Because |
|----------------|-----------|---------|
| "We are 3 people and shipping v1" | Monolith (multi-tier) | Independence is not yet a need; ops cost is real |
| "Our SSR web app needs to hold sessions" | Multi-tier monolith | Presentation/app/data split is enough |
| "One service has 100x the traffic of the rest" | Extract that service | Scale the hotspot, keep the rest simple |
| "Two teams block each other on every release" | Split by bounded context | Team autonomy is the actual driver |
| "Traffic is spiky and mostly idle" | Serverless | Pay per invocation; scale to zero |
| "We need a model-inference HTTP endpoint" | Serverless + connection proxy | Bursty, stateless; proxy the DB |
| "We share one database across 'services'" | **Stop** - that is a distributed monolith | Split by data ownership, not just by process |

---

### 7. Choosing - The Method

Do not choose a model by taste. Answer these questions, in order, honestly:

1. **Team size and structure.** One builder or a small team? The monolith's simplicity wins. Multiple teams that step on each other? Boundaries matter more, so consider services - but only around the friction. (Conway's Law: the system will mirror the team.)
2. **Traffic shape.** Steady load favors always-on servers; bursty or mostly-idle load favors serverless. A single hotspot favors extracting *that* service, not all of them.
3. **Statefulness.** Sessions and in-process state say monolith or multi-tier. Serverless forces external state.
4. **Operational capability.** Who runs this at 3 a.m.? If you cannot staff monitoring, tracing, and on-call, do not adopt microservices. Complexity you cannot operate is not architecture, it is a liability.
5. **Regulatory / transaction needs.** Strong multi-step transactional integrity favors a shared database in one process. Distributed transactions are a real cost, not a buzzword.
6. **Time-to-first-value.** The fastest correct thing that can evolve is almost always a multi-tier monolith.

**The course-relevant takeaway:** build the 3-tier monolith first (browser+nginx -> Express -> Postgres). Enforce clean module boundaries *inside* it, so each module owns its tables and exposes an interface. When a specific, measurable pain appears - a hotspot that needs independent scaling, or a team that blocks its neighbour - extract exactly that module, using the strangler pattern. Every extraction is a small, reversible step, never a rewrite. [-> `02-modular-monolith-vs-microservices.md`](02-modular-monolith-vs-microservices.md)

```
  START: multi-tier monolith with clean internal boundaries
        
          pain appears (evidence, not vibes)
        
  Which pain?
    one hotspot needs scaling ......... extract THAT bounded context
    two teams block each other ........ split along their boundary
    bursty / idle glue work ........... move that piece to serverless
    genuine multi-step transactions ... keep it together, do not "split"
        
        
  still shipping? you chose correctly.
```

> **Why it matters (exam and career):** The four models are an exam taxonomy, but for an AI Engineer they are a recurring product decision. Most ML/AI products ship as **a monolith service (API + UI) plus a model-serving endpoint** - often serverless - because that is the shape that gets value to users fastest. Being fluent in the spectrum, and in the cost of each step to the right, is the mark of someone who designs systems rather than assembles them.

---

## Tech Cross-References

- **Modular monolith vs microservices - the decision framework.** This note names the four models; note 02 gives the boundaries, contracts, and "monolith-first" mechanics behind the choice. [-> `02-modular-monolith-vs-microservices.md`](02-modular-monolith-vs-microservices.md)
- **Event-driven architecture - how microservices and serverless communicate without tight coupling.** Events replace synchronous chains between services and trigger functions. [-> `01-event-driven-architecture-task-orchestration.md`](01-event-driven-architecture-task-orchestration.md)
- **SSR vs CSR - what each tier actually renders.** The presentation/application split is concrete in how a page is built and where. [-> `../dev_foundation/web_server/01-static-dynamic-ssr-vs-csr.md`](../dev_foundation/web_server/01-static-dynamic-ssr-vs-csr.md)
- **Docker Compose - the 3-tier topology written down.** nginx, Express, and Postgres wired as separate containers is the multi-tier model in a file. [-> `../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md`](../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md)

---

## Research Journey

- **Why:** I kept hearing "microservices" as if it were the destination and "monolith" as if it were an insult. I wanted an honest map: what are these four models *for*, and which one does my Web Application Development project actually use? Reading the syllabus taxonomy forced me to name what I was already building - a 3-tier monolith - and to be proud of that choice instead of apologising for it.
- **Struggle:** The hardest thing to accept was that **more parts is a cost, not a badge**. My instinct was that splitting into services is "more advanced." Working through the price - distributed transactions, network failure, ops, versioning - flipped the default: dividing a working system must *earn* its complexity, and the burden of proof is on the split, not the whole.
- **Aha moment:** Drawing my own course app as tiers and seeing that it is *already three-tier*: browser+nginx, Express, PostgreSQL. Nothing needed to change; I just had to name the interfaces (HTTP, SQL). The second aha was serverless: it looks like the simplest model, but its statelessness collides directly with Postgres connection limits - a reminder that architecture choices interact, they do not stack.
- **Career link:** As an AI Engineer in economics/business, I will constantly decide how to package a model + data + UI for a client. The realistic answer is almost always "a monolith service with a model endpoint," and knowing *why* - and when it stops being enough - is exactly the judgment that separates an engineer who builds products from one who follows trends.

## Related Topics

- [`02-modular-monolith-vs-microservices.md`](02-modular-monolith-vs-microservices.md) - the deep dive behind the monolith/microservices choice
- [`01-event-driven-architecture-task-orchestration.md`](01-event-driven-architecture-task-orchestration.md) - the async contract layer for distributed models
- [`04-web-performance-caching-and-bottlenecks.md`](04-web-performance-caching-and-bottlenecks.md) - where the cache and tiers physically sit, and how performance follows the topology
- [`../dev_foundation/web_server/01-static-dynamic-ssr-vs-csr.md`](../dev_foundation/web_server/01-static-dynamic-ssr-vs-csr.md) - how the presentation tier produces a page
- [`../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md`](../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md) - the 3-tier topology as a deploy file
- [SUGGESTED] `bridge-serverless-and-model-serving.md` - *why it's important:* serverless is the natural deployment for bursty ML inference, and the connection-proxy interaction deserves its own note tying architecture to MLOps. *Folder:* `software_architecture/` or `ai_ml/`.
- [SUGGESTED] `05-application-architecture-case-study-joblink.md` - *why it's important:* applying the four models to the real Joblink stack would turn this taxonomy into evidence about which model fits an actual full-stack project. *Folder:* `software_architecture/`.

## Open Questions

- [ ] Is my course project's tier boundary drawn in the right place - should nginx be considered part of "presentation," or is it infrastructure?
- [ ] At what concrete signal (team size, release frequency, a hot endpoint) would extracting one module from my monolith actually pay for its operations?
- [ ] For a serverless ML inference endpoint, when is a connection proxy enough, and when is a queue plus persistent workers the better architecture?
- [ ] How do multi-tier and event-driven (note 01) combine - is a tiered monolith with a message bus the sweet spot for a one-builder business system?
- [ ] Where exactly does the "distributed monolith" line fall - how do I detect it early instead of after it is painful?

---
*Models are not a ladder. They are a set of trades - pick the one whose costs you are willing to pay.*
