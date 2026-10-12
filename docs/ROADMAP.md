# Roadmap

This roadmap orders risk. Dates come after each milestone has an executable
acceptance test.

## Milestone 0: make AgentSlave boring

AgentSlave is the reference workload and the place to prove operational
conventions.

- Idempotent intake, retries, and pull-request lifecycle handling.
- An operator-facing `doctor` command with human and JSON output.
- Backup and restore commands for PostgreSQL, the object store, configuration,
  and Codex authentication state.
- A documented upgrade and rollback exercise using NixOS generations.
- Resource ceilings, disk-pressure checks, and actionable failure messages.
- Integration tests covering Slack report to draft PR on a disposable host.
- A release artifact, changelog, compatibility policy, and signed provenance.

Acceptance: a new operator can install AgentSlave on one clean NixOS host,
process a sample bug, restore it onto a second host, and diagnose a deliberately
broken dependency using only checked-in documentation.

## Milestone 1: host bootstrap and inspection

Build the smallest separate CLI that can manage one existing Linux server.

- Support Hetzner Cloud creation and Hetzner Robot rescue as the first provider
  adapters.
- Initialize a NixOS host over SSH without retaining a central daemon.
- Compose pinned `nixos-anywhere` and `disko` rather than reimplementing their
  installation and partitioning engines.
- Inventory CPU, memory, disks, network exposure, generations, and workloads.
- Produce a deterministic plan before changing the host.
- Apply, verify, roll back, and emit a local audit record.
- Reuse AgentSlave's process, health, object-store, and Nix conventions where
  they are genuinely generic; extract packages instead of importing the app.

Acceptance: install and remove a stateless sample service with a previewed plan,
health verification, and successful rollback. Replace a Hetzner test host from
checked-in configuration without using the web console.

## Milestone 1.5: raw machine to managed host

- Discover hardware through local boot facts, Redfish, or IPMI.
- Build a reproducible NixOS image for a declared hardware class.
- Preview disk destruction, partitioning, networking, and boot changes.
- Install through ISO, PXE, or an out-of-band management adapter.
- Verify the resulting host and add its facts to a plain inventory file.
- Match the host to a hardware class and run topology, performance, and thermal
  qualification before marking it available.
- Retain a structured qualification report and quarantine failed machines.
- Make the entire flow non-interactive and suitable for CI.

Acceptance: take one blank physical or virtual machine from power-on to a
verified, SSH-reachable NixOS host using checked-in configuration and no
67daisies account.

## Milestone 2: service package contract

- Define a versioned manifest for ports, domains, secrets, volumes, databases,
  health checks, backup hooks, restore hooks, and upgrade notes.
- Package three real services: one stateless web service, one PostgreSQL-backed
  application, and AgentSlave itself.
- Add validation that rejects undeclared persistence or network exposure.
- Keep generated Nix readable and exportable.

Acceptance: deploy all three packages to one host and recover them from only
the exported configuration and backup set.

## Milestone 3: essential operations

- Encrypted secret material with host-scoped decryption.
- Scheduled local and off-site backups with restore drills.
- Certificate and domain automation with explicit DNS boundaries.
- Upgrade channels, maintenance windows, notifications, and rollback policy.
- Disk, memory, service, certificate, and backup health reporting.

Acceptance: survive a failed upgrade, a full host loss, an expired credential,
and a nearly-full disk without undocumented intervention.

## Milestone 4: rack and fleet

- Manage a handful of hosts or a rack without introducing a consensus system.
- Select hosts by declared hardware facts, role, accelerator, rack, or health.
- Acquire time-bounded host and accelerator leases with cleanup and quarantine
  behavior.
- Apply bounded concurrency and preserve a per-host plan and audit trail.
- Model rack, power, network, BMC, and storage failure domains for placement and
  canary selection.
- Explicit placement and failover policy rather than implicit scheduling.
- Migration tooling between supported hosts and providers.

Acceptance: move a stateful service between two providers and prove that loss
of the operator workstation does not stop either host. Rebuild the inventory
from the machines and checked-in configuration alone.

## Immediate engineering slice

The infrastructure family starts outside AgentSlave. Specify and prototype a
single raw-machine workflow:

1. inspect a host through SSH or Redfish and emit versioned JSON facts;
2. compare those facts with a checked-in NixOS host declaration;
3. produce a plan covering boot mode, disks, partitions, network, image, and
   destructive actions;
4. install the machine and wait for it to return over SSH;
5. verify the installed generation and write a local JSONL audit event.

The first acceptance fixture can be a VM, but the interface must model BMC
identity and physical disks from the start. AgentSlave's future `doctor`
command can consume the diagnostics contract; it is not the source of the
bare-metal architecture.
