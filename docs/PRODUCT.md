# Product direction

## Mission

Give a small business or small engineering team a credible way to run essential
software on one or a few ordinary Linux servers without making a public cloud,
Kubernetes, or a hosted control plane part of the application runtime.

The experience should feel closer to installing an appliance than assembling a
platform. Operators declare what should run, preview the change, apply it, and
retain enough state and documentation to recover the system without 67daisies.

AgentSlave is the first product in this direction. It automates a bounded
software-maintenance workflow and proves the underlying operating model:
declarative host configuration, isolated workloads, durable state, auditable
events, explicit human approval, and replaceable external integrations.

## Who this is for

- A company with one to twenty engineers and no dedicated platform team.
- An agency operating a small number of client applications.
- A local business that needs internal tools, automations, or data services.
- A technical founder who can operate Linux but does not want infrastructure to
  become the product.

It is not initially designed for untrusted public multi-tenancy, thousands of
hosts, or organizations that already need Kubernetes-scale scheduling.

## Product principles

1. **The server keeps working without us.** A hosted 67daisies account may add
   convenience later, but it must never be required for an installed workload
   to boot, serve traffic, restore a backup, or be administered locally.
2. **One server is a complete topology.** Multi-host operation is an extension,
   not a prerequisite disguised as an entry-level architecture.
3. **Boring state wins.** Prefer files, PostgreSQL, systemd, NixOS, Incus, SSH,
   and content-addressed objects over bespoke distributed systems.
4. **Plans precede mutations.** Operators can see the services, ports, storage,
   migrations, restarts, and destructive actions before applying a change.
5. **Every operation is recoverable.** Upgrades retain a rollback generation;
   deletion is explicit; backups include tested restore instructions.
6. **Isolation has a replaceable boundary.** Incus is the first workload
   runtime. Workflows depend on a small runner contract rather than Incus calls
   scattered throughout product logic.
7. **No hostage data.** Configuration is text, artifacts use documented
   formats, and state can be exported with ordinary tools.
8. **Secure defaults without security theatre.** Minimize exposed ports, scope
   credentials, encrypt transport, record privileged actions, and make unsafe
   development modes obvious.
9. **No surprise metering.** Core self-hosted operation is useful without a
   usage-based 67daisies bill. Optional paid services must have visible limits.
10. **Small tools, shared conventions.** Follow the HashiCorp product lesson,
    not its historical complexity: separate tools with a consistent CLI,
    configuration language, release process, and interoperability contract.

## Product boundaries

The ecosystem should not become a monolith.

### AgentSlave

Owns intake-to-change automation: reports, evidence, isolated coding runs,
verification, audit history, and draft pull requests. It may consume the common
host/runtime contracts but should not become a general deployment platform.

### Host and deployment tool

A future, separately versioned CLI should own server bootstrap, plans, NixOS
generation changes, service installation, health checks, upgrades, and
rollbacks. Its first supported target is one NixOS server reachable over SSH.
The name is intentionally undecided until its command and package contracts are
proven.

### Service catalog

Deployable services should be ordinary, reviewable packages with declared
ports, volumes, secrets, health checks, backup hooks, restore hooks, and upgrade
notes. A catalog is a distribution mechanism, not an opaque marketplace.

### Optional hosted coordination

Remote inventory, notifications, encrypted backup storage, and team approvals
may be offered later. Installed systems must degrade to local operation when
that service is absent.

## Architecture shape

```text
operator CLI
    -> plan and policy engine
        -> SSH to one or more hosts
            -> NixOS generation and systemd services
            -> Incus workload boundary
            -> PostgreSQL and content-addressed storage
            -> local audit log, backup and restore hooks

Slack / GitHub / future integrations
    -> AgentSlave
        -> the same host, runner and audit contracts
```

The operator CLI is not a permanently connected control plane. It produces a
plan, applies it through explicit host APIs, records the result, and can
disconnect. The host owns its desired configuration and can be operated with
standard NixOS, systemd, Incus, PostgreSQL, and SSH tools if the CLI disappears.

## Commercial posture

The open-source tools must be enough to run the system. Sustainable paid work
can come from managed updates, encrypted off-site backups, fleet visibility,
support, certified service packages, and a hosted approval/integration layer.
The business should sell reduced operational burden, not artificial lock-in.

## Naming

`AgentSlave` is an existing working name, not automatically the umbrella brand.
Before a broad public launch, evaluate a clearer product name that communicates
an isolated worker or steward without carrying avoidable social baggage. A
rename must include migration aliases and must not delay proving the product.
