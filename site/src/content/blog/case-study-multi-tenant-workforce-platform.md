---
title: "Case study: building a multi-tenant workforce management platform"
description: "How we took an enterprise workforce and project-management platform from greenfield to production — the architecture, the delivery process, and the numbers behind it."
date: 2026-08-04
---

One of our longest-running engagements has been building a workforce and project-management platform for an enterprise client — the kind of system that touches almost every part of an organization: people, projects, reporting, and access control. This is a look at how we approached it, what we built, and what the platform looks like a year in.

## The problem

Our client needed to replace fragmented, manual processes for managing their workforce with a single platform that could:

- Track employees, roles, and organizational structure in one place
- Manage projects end-to-end — creation, resource allocation, timelines, and task categorization
- Collect and process recurring status submissions on a configurable reporting cycle, with grace periods and business-day handling built in
- Turn all of that into real-time dashboards — resource utilization, submission trends, training compliance, risk and issue tracking
- Enforce strict role-based access control, scoped per organization, since the platform needed to serve multiple tenants rather than a single company

That last point shaped almost every architectural decision: this wasn't a single-tenant internal tool, it was a platform meant to onboard and isolate multiple organizations, each with their own users, roles, and data boundaries — administered centrally by our client's own operations team through a separate admin portal.

## Architecture & stack

We split the system into three applications sharing one backend, plus the infrastructure to run all of it:

**Tenant-facing web app** — React 19, TypeScript, and Vite, with Redux Toolkit and RTK Query handling state and data fetching, React Hook Form and Zod for forms and validation, Tailwind CSS and Radix UI for the component layer, and Recharts for the dashboards. Authentication runs through AWS Cognito's hosted login with the authorization-code-plus-PKCE flow.

**HQ admin portal** — a second React application built on the same foundations, purpose-built for platform-level administrators to manage tenants, cross-organization users, and role/permission definitions from a dedicated interface, separate from the tenant-facing app.

**Backend platform** — NestJS on Node.js, with PostgreSQL via Drizzle ORM for typed schemas and migrations. The API is organized into modules — Auth, Organization, Profile, Role, Support (a full ticketing system with threaded messages and attachments), and Uploads (presigned S3 upload/download with lifecycle tracking) — each backed by the same database and fronted by an API gateway. CASL handles fine-grained permission rules on top of Cognito-issued JWTs.

**Infrastructure** — Terraform-managed AWS, environment-scoped across dev, QA, stage, and prod. The stack runs on EKS with Fargate profiles behind an application load balancer, RDS for Postgres, API Gateway with a VPC link, Cognito for identity, S3 and Lambda for file processing, and CloudWatch and SES rounding out observability and email. Two Helm charts — one for the frontend, one for the platform — get built, versioned, and deployed per environment.

It's a stack chosen for boring reliability over novelty: managed services wherever they made sense, typed schemas end to end, and a clean module boundary in the backend so new features (a support desk, presigned uploads, subscription billing) could be added without destabilizing the core.

## Process & delivery

The engagement started with the frontend and infrastructure work in parallel, with the backend following within days — a deliberate choice so the API contract and the UI could be shaped together rather than one blocking the other. The HQ admin portal, a dedicated load-testing suite, and a shared API collection were added several months in, once the core platform had stabilized enough to need dedicated tooling around it.

Every repository follows the same delivery discipline:

- **Unit tests gate every deploy.** No merge reaches an environment without its test suite passing in CI.
- **Environment-scoped deployments** — dev, QA, stage, and prod are distinct pipelines, not a single branch-and-pray flow.
- **Automatic semantic versioning and GitHub releases** on the backend, so every deploy is traceable to an exact, tagged build.
- **Signed commits required** across the codebase — a small thing that keeps the provenance of every change verifiable.

A small, cross-functional team has carried this from greenfield to a live, multi-environment platform still under active development a year later — the same people working across frontend, backend, and infrastructure rather than siloed hand-offs between them.

## Outcomes

We ran structured load tests against the platform's core read paths — the endpoints tenants hit hardest in daily use — at 100 concurrent virtual users:

| Journey | Error rate | p95 latency | Throughput |
|---|---|---|---|
| Baseline read | 0% | 467 ms | 15,060 req/min |
| Project read | 0% | 974 ms | 12,360 req/min |
| Status report read | 0% | 801–920 ms | 10,080–13,980 req/min |

Every journey held a 0% error rate under load, and all stayed within a 1-second p95 latency budget — with the heavier read paths (projects, status reports) identified early as the ones to keep optimizing as usage grows. That's the kind of result we look for: not a flashy number, just a system that stays boring and predictable under real load, with the headroom clearly mapped out before it becomes a problem.

The platform is still growing — new modules ship regularly, and the infrastructure and test tooling built alongside it mean each addition gets measured the same way the last one did.

---

Want something built with the same discipline? [Get in touch](/#contact).
