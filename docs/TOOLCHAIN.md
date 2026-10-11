# Bare-metal toolchain shape

67daisies should resemble early HashiCorp in product shape: several focused
binaries, common interaction patterns, and stable interchange contracts. It
should not resemble a cloud console split into command-line clients.

Names below describe responsibilities and are not product names.

## The tools

### Facts

Discovers what a machine is.

- CPU, memory, accelerators, disks, NICs, boot mode, firmware, BMC, rack and
  power facts.
- Reads locally, over SSH, or through Redfish/IPMI adapters.
- Emits a versioned document and never mutates the host.
- Can rebuild inventory from the machines when the operator loses a laptop.

```text
facts inspect ssh://root@node-01 --json
facts inspect redfish://bmc-01 --output inventory/node-01.json
```

### Image

Builds and inspects reproducible host images.

- Resolves a Nix flake and hardware class to a bootable artifact.
- Produces checksums, provenance, package inventory, and compatibility facts.
- Supports ISO first, then netboot and provider-specific formats.
- Signs artifacts using operator-owned keys.

```text
image build ./hosts/gpu-worker.nix --format iso
image inspect ./result/worker.iso
```

### Metal

Takes a raw or replaceable machine to a verified operating system.

- Plans disk, boot, firmware, network, and image changes.
- Installs through an attached ISO, PXE, rescue environment, or BMC virtual
  media adapter.
- Treats installation as destructive and requires an explicit reviewed plan.
- Finishes by verifying SSH and recording the resulting generation.

```text
metal plan node-01 --host ./hosts/node-01.nix --out plan.json
metal apply plan.json
metal verify node-01
```

### Run

Executes builds and tests on owned machines.

- Selects workers from inventory by architecture, accelerator, memory, labels,
  and health.
- Creates an ephemeral Incus or native process boundary.
- Streams logs and emits a versioned result document.
- Accepts events from GitHub, GitLab, or local Git without making any forge the
  scheduler or source of machine state.
- Begins with explicit leases and bounded concurrency, not a distributed
  scheduler.

```text
run exec --select 'gpu=7900xtx' -- ./test-model.sh
run ci --repo . --event ./fixtures/push.json
```

### Ship

Moves a tested artifact to declared machines.

- Plans service, port, secret, volume, migration, and restart changes.
- Deploys a NixOS generation, Nix closure, OCI artifact, or service package.
- Verifies health, retains the previous generation, and rolls back on failure.
- Supports batches and canaries by repeatedly invoking the single-host path.

```text
ship plan --artifact ./result --select 'role=api'
ship apply ./deploy-plan.json --max-parallel 2
ship rollback node-03
```

### Doctor

Explains why machines and workloads are unhealthy.

- Runs local or remote checks without needing a dashboard.
- Understands disks, memory, network, certificates, generations, systemd,
  Incus, databases, backups, accelerator drivers, temperatures, and power.
- Produces both concise remediation and stable structured output.
- Can compare a failing machine with a healthy peer of the same hardware class.

```text
doctor host node-01
doctor fleet --select 'rack=r1' --json
```

## Shared contracts

The family becomes a toolchain through files and process interfaces, not a
shared control-plane database.

### Inventory document

Operator-owned host identity, connection methods, observed hardware facts,
roles, labels, and the last verified generation. Secrets are referenced, never
embedded.

### Plan document

Immutable description of intended mutations, prerequisites, destructive
actions, expected restarts, verification steps, rollback boundary, tool
version, and input hashes. `apply` consumes the exact reviewed plan.

### Result and audit event

Every operation emits timestamps, target, plan hash, actor, outcome, produced
artifacts, verification evidence, and rollback status as appendable JSONL.

### Adapter protocol

SSH, Redfish, IPMI, PXE, Git forge, DNS, object storage, and secret backends sit
behind capability-oriented interfaces. An adapter declares its capabilities;
the core never assumes every provider can perform every operation.

### Runner protocol

A job is inputs, command, isolation requirements, resource requirements,
timeout, outputs, and cleanup policy. AgentSlave, CI, image builds, and restore
tests can share this contract without sharing their application state.

### Hardware class and qualification report

A hardware class declares expected device topology, firmware ranges,
performance floors, thermal limits, compatible images, and qualification
tests. A qualification report combines observed facts with evidence and a
`pass`, `degraded`, or `quarantined` decision.

### Lease and failure domain

A lease is a time-bounded resource claim with an owner, cleanup policy, and
recovery behavior. Failure domains describe correlated rack, power, switch,
BMC-network, and storage risks so CI allocation and fleet rollout do not place
every copy behind the same failure.

## The datacenter test

An expert operating a rack should be able to:

1. clone the repositories and build every binary from source;
2. inventory machines without registering an account;
3. install ten identical hosts concurrently with bounded failure domains;
4. select machines by real hardware facts, including accelerators;
5. attach existing Git and CI workflows;
6. see the native command and file behind every abstraction;
7. replace one tool while retaining inventory and desired state;
8. reconstruct operations from Git, machine facts, artifacts, and audit logs.

If the suite requires a dashboard, proprietary service, opaque state database,
or always-connected coordinator to pass that test, the architecture is wrong.

## First repository boundary

The first new repository should contain the shared schemas plus `facts` and
`metal`. They form one vertical slice from an unknown machine to a verified
NixOS host. Split `image`, `run`, `ship`, and `doctor` into independent tools
only after the schemas are exercised on real hardware.

AgentSlave remains its own application. It can later use `run` for isolated
jobs, `doctor` for host evidence, and `ship` for deployment without owning any
of those tools.

The machine lifecycle and CI practices observed in tiny corp's public
repositories are analyzed in
[TINYCORP-OPERATIONS.md](./TINYCORP-OPERATIONS.md).
