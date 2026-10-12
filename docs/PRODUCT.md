# Product direction

## Mission

Build a family of small tools that turns raw servers into usable infrastructure
without requiring a public cloud, Kubernetes, a hosted control plane, or a
67daisies account. The same primitives should work for a small business with
one rented machine and an expert operator bringing up a rack of bare-metal or
GPU servers.

The experience should feel like a well-designed Unix toolchain rather than a
platform. An operator can discover hardware, install an operating system,
configure networking and storage, attach CI/CD, deploy workloads, diagnose the
machine, and recover it. Each tool works alone, composes through documented
files and commands, and leaves the machine understandable without 67daisies.

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
- A datacenter or lab operator provisioning racks of heterogeneous bare-metal
  and accelerator machines.
- An open-source project that needs reproducible CI workers on hardware it owns.

The initial implementation will still prove itself on one machine. It is not
initially designed for untrusted public multi-tenancy or hyperscaler-scale
scheduling, but its file formats and host contracts must not assume that a
fleet has only one node.

The first provider target is Hetzner Cloud and Hetzner dedicated servers. This
is a product wedge, not a permanent dependency: provider behavior lives behind
capability-oriented adapters, while host configuration, plans, evidence, and
recovery remain provider-neutral.

## Product principles

1. **There is no account.** Installation, provisioning, deployment, CI/CD,
   backup, restore, and administration work with local files and operator-owned
   machines. Network services may be integrated, but none belongs to the
   product's trust boundary.
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
9. **No metering.** The tools do not count hosts, cores, builds, deployments,
   or users and do not phone home to enforce a commercial boundary.
10. **Small tools, shared conventions.** Follow the HashiCorp product lesson,
    not its historical complexity: separate tools with a consistent CLI,
    configuration language, release process, and interoperability contract.
11. **One machine and one rack use the same primitives.** Fleet operation adds
    inventory, selection, and concurrency; it does not replace the single-host
    execution model.
12. **Expert operators keep the escape hatch.** Every operation has a verbose
    mode, a machine-readable result, and an ordinary command or file that can
    be inspected and reproduced by hand.

## Product boundaries

The ecosystem should not become a monolith.

### AgentSlave

Owns intake-to-change automation: reports, evidence, isolated coding runs,
verification, audit history, and draft pull requests. It may consume the common
host/runtime contracts but should not become a general deployment platform.

### Host and deployment tool

A future, separately versioned CLI should own host discovery, bootstrap, plans,
NixOS generation changes, service installation, health checks, upgrades, and
rollbacks. Its first supported target is one NixOS server reachable over SSH;
the same interface should later drive machines discovered through BMC,
Redfish, IPMI, or PXE. The name is intentionally undecided until its command
and package contracts are proven.

### Image and bare-metal tool

A small image tool should build, inspect, sign, and install reproducible host
images. It should understand disks, boot mode, firmware facts, hardware class,
and the boundary between destructive installation and reversible
configuration. Provider APIs are adapters; the core target is a machine the
operator can power cycle and reinstall.

### CI/CD tool

A repository-local tool should turn owned machines into CI workers and deploy
artifacts to declared targets. GitHub, GitLab, and local Git are event sources,
not control planes. Jobs use the same runner contract as AgentSlave, emit the
same audit format, and can execute without a 67daisies service.

### Inventory and diagnostics tool

Inventory is a signed or versioned set of host facts: identity, BMC address,
hardware, disks, accelerators, network, installed generation, workloads, and
health. Diagnostics should work against one host, an inventory selection, or
stdin and should always support structured output.

### Service catalog

Deployable services should be ordinary, reviewable packages with declared
ports, volumes, secrets, health checks, backup hooks, restore hooks, and upgrade
notes. A catalog is a distribution mechanism, not an opaque marketplace.

### Integration adapters

DNS providers, object stores, Git forges, secret stores, notification systems,
and hardware management protocols are replaceable adapters. Operators supply
their own endpoints and credentials. No adapter may silently introduce a
67daisies account or vendor-owned state.

## Architecture shape

```text
inventory + repository configuration
    -> small command-line tools
        -> inspect / image / provision / plan / apply / verify
        -> build / test / release / deploy / rollback
            -> SSH, Redfish/IPMI, PXE and Git adapters
                -> NixOS generations and systemd services
                -> Incus workload boundary
                -> disks, networks and accelerator runtimes
                -> local audit log, backup and restore hooks

Slack / GitHub / future integrations
    -> AgentSlave
        -> the same host, runner and audit contracts
```

The tools are not a permanently connected control plane. They read inventory
and desired state, produce plans, apply through explicit host APIs, record
results, and exit. Hosts remain operable with standard NixOS, systemd, Incus,
PostgreSQL, SSH, and hardware-management tools if every 67daisies binary
disappears.

## Commercial posture

The open-source tools are the product, not a client for a future account.
Sustainable work can come from support, integration engineering, certified
hardware and service packages, training, compatibility testing, and reference
server images. The business should sell expertise and reliability, not control
of the operator's machines or metadata.

The sustainability and provider strategy is documented in
[ENGINEERING-INDEPENDENCE.md](./ENGINEERING-INDEPENDENCE.md).

## Naming

`AgentSlave` is an existing working name, not automatically the umbrella brand.
Before a broad public launch, evaluate a clearer product name that communicates
an isolated worker or steward without carrying avoidable social baggage. A
rename must include migration aliases and must not delay proving the product.
