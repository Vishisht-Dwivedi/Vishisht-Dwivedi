<h1 align="center">Vishisht Dwivedi</h1>

<p align="center"><img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&weight=600&size=21&duration=2600&pause=800&color=36BCF7&center=true&vCenter=true&width=700&lines=Reactor+Pattern+Networking+in+C;Distributed+Systems+with+gRPC+and+Queues;Linux+Internals+and+Systems+Programming;Shipping+Production+Grade+Backend+Infra"/></p>

<p align="center">
<a href="https://github.com/Vishisht-Dwivedi"><img src="https://img.shields.io/github/followers/Vishisht-Dwivedi?style=for-the-badge&logo=github&label=Follow&color=181717"/></a>
<a href="mailto:public.vishisht.dwivedi@gmail.com"><img src="https://img.shields.io/badge/Email-D44638?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/vishisht-dwivedi-066004311"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
<img src="https://img.shields.io/badge/Open_to-Internships-2ea44f?style=for-the-badge"/>
<img src="https://komarev.com/ghpvc/?username=Vishisht-Dwivedi&style=for-the-badge&color=36BCF7&label=VIEWS"/>
</p>

CS undergrad @ **IIIT Bhopal** who builds the layer underneath the app — hand-rolled TCP reactors, multithreaded filesystem daemons, gRPC/queue-driven microservice meshes. **Seeking Systems / Backend / Infra internships.**

<p align="center">
<img src="https://img.shields.io/badge/C-00599C?logo=c&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/C++-00599C?logo=cplusplus&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/Go-00ADD8?logo=go&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black&style=flat-square"/>&nbsp;<img src="https://img.shields.io/badge/Node.js-339933?logo=nodedotjs&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/Fastify-000000?logo=fastify&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/GraphQL-E10098?logo=graphql&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/gRPC-4285F4?style=flat-square"/><img src="https://img.shields.io/badge/BullMQ-CC0000?style=flat-square"/>&nbsp;<img src="https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/Redis-DC382D?logo=redis&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/Prisma-2D3748?logo=prisma&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/nginx-009639?logo=nginx&logoColor=white&style=flat-square"/>&nbsp;<img src="https://img.shields.io/badge/Next.js-000000?logo=nextdotjs&logoColor=white&style=flat-square"/><img src="https://img.shields.io/badge/React-61DAFB?logo=react&logoColor=black&style=flat-square"/><img src="https://img.shields.io/badge/TailwindCSS-06B6D4?logo=tailwindcss&logoColor=white&style=flat-square"/>
</p>

### ⚙️ Systems Engineering

| Project | Under the hood | Stack |
|---|---|---|
| **[Velora](https://github.com/Vishisht-Dwivedi/Velora)** | Hand-written TCP engine — single-threaded **epoll reactor**, ring-buffered per-connection I/O, custom 8-byte binary frame (magic·version·type·flags·len) driving CONNECT/PING/STREAM_OPEN/PUBLISH opcodes, **256 multiplexed streams per socket**, connection table sized for **65,536 concurrent sockets**. Ships its own load-test rig — churn, concurrent-churn & Slowloris scripts to prove it survives. | `C` `epoll` `Linux` |
| **[chronofs](https://github.com/Vishisht-Dwivedi/chronofs)** | Filesystem-watch **daemon** — recursive `inotify` trees, **thread-per-watcher** POSIX model bridged over **Unix Domain Socket IPC** to a CLI client, dynamic watcher (un)registration with auto-expansion into new subdirs, snapshot-commit versioning (rollback in progress). | `C` `pthreads` `inotify` |

<sub>Root of it all: **[low-level-server](https://github.com/Vishisht-Dwivedi/low-level-server)** — raw TCP/UDP/WebRTC chatrooms + bare C sockets, the sandbox Velora grew out of.</sub>

### 🕸️ Sermocino — distributed messaging platform
**[github.com/Vishisht-Dwivedi/Sermocino](https://github.com/Vishisht-Dwivedi/Sermocino)** · pnpm monorepo, independently deployable services, polyglot-style comms:

```mermaid
flowchart LR
  B([Browser]) --> N[nginx Gateway]
  N -->|/api/auth| A[Auth Service]
  N -->|/*| WEB[Next.js App]
  A <-->|gRPC CreateProfile| U[User Service]
  A --> PG1[(Postgres: auth)]
  U --> PG2[(Postgres: user)]
  A -->|enqueue| R[(Redis)]
  M[Media Service] -->|enqueue| R
  R --> W[Media Worker]
  W -->|worker_threads + sharp| O[(Storage)]
```
- **Fastify** services (`auth` · `user` · `media`) — independently deployable, each with auto-generated Swagger/OpenAPI docs
- **gRPC + Protobuf** for internal RPC (auth service calls user service's `CreateProfile`), typed via a shared workspace package
- **BullMQ over Redis** decouples slow work from the request path; `media-worker` further offloads CPU-bound media-processing to Node **worker_threads** running `sharp`
- **nginx** gateway doing path-based routing across services + the Next.js frontend
- **Postgres via Prisma**, schema-per-service (`auth`, `user`) — real data-ownership boundaries, not a shared blob
- Auth hardened: bcrypt hashes, JWT + **rotating refresh tokens hashed at rest and revocable**, per-session device fingerprinting (IP/OS/browser/UA), OTP email verification
- Dockerfile per service, root `compose.yaml` with healthchecks; shared Zod schemas + gRPC types give compile-time safety across service boundaries

### 🌐 Product Engineering

| Project | Details | Stack |
|---|---|---|
| **[Academia Tempore](https://github.com/Vishisht-Dwivedi/Academia_Tempore)** | Full-stack scheduling platform for **IIIT Bhopal** — Apollo **GraphQL** API over MongoDB/Mongoose, JWT auth with **RBAC**, a collision-detection engine for clash-free timetables, Zustand state. | `Next.js 15` `Express 5` `GraphQL` `MongoDB` |
| **Freelance** | [Laxmi Finsec](https://www.laxmifinsec.com/) · [Prof. Rahul Chaurasia](https://www.rahulchaurasia.com/) · [Manthan Rehab](https://www.manthanrehab.org/) — production sites shipped end-to-end for real clients. | `React` `Next.js` `Tailwind` |

<sub>Also shipped: <a href="https://vishisht-dwivedi.github.io/waves-simulation/">waves-simulation</a> — sine-wave field renderer · <a href="https://github.com/Vishisht-Dwivedi/diurnal-chronique">diurnal-chronique</a> — retro news portal with live D3 crypto charts · <a href="https://github.com/Vishisht-Dwivedi/IIITB-IEEE-SBC">IEEE CS SBC site</a> — animated SPA w/ Nodemailer backend · <a href="https://github.com/Vishisht-Dwivedi/Blob-Simulations">Blob-Simulations</a> / <a href="https://github.com/Vishisht-Dwivedi/Physics-simulation-using-balls">Physics-Balls</a> — canvas physics, zero libraries</sub>

### 🏆 Open Source & Campus
<img src="https://img.shields.io/badge/GSSoC-Contributor-success?style=flat-square"/> <img src="https://img.shields.io/badge/SSOC-Contributor-success?style=flat-square"/> <img src="https://img.shields.io/badge/GitHub-Pull_Shark_x2-8957e5?style=flat-square&logo=github&logoColor=white"/> <img src="https://img.shields.io/badge/IEEE_CS-Webmaster-00629B?style=flat-square"/> <img src="https://img.shields.io/badge/IIIT_Bhopal-Website_Cell%2C_Asst._Web_Lead-6f42c1?style=flat-square"/>

<p align="center"><i>Coffee, rain, music and a black/blue wall of code</i></p>
