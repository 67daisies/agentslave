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

- Initialize a NixOS host over SSH without retaining a central daemon.
- Inventory CPU, memory, disks, network exposure, generations, and workloads.
- Produce a deterministic plan before changing the host.
- Apply, verify, roll back, and emit a local audit record.
- Reuse AgentSlave's process, health, object-store, and Nix conventions where
  they are genuinely generic; extract packages instead of importing the app.

Acceptance: install and remove a stateless sample service with a previewed plan,
health verification, and successful rollback.

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

## Milestone 4: small fleet

- Manage a handful of hosts without introducing a consensus system.
- Explicit placement and failover policy rather than implicit scheduling.
- Optional hosted inventory and approval UI that is never on the runtime path.
- Migration tooling between supported hosts and providers.

Acceptance: move a stateful service between two providers and prove that loss
of hosted coordination does not stop either host.

## Immediate engineering slice

The next code change should implement `agentslave doctor` as the first stable
operator contract. It should check:

- PostgreSQL connectivity and migration level;
- Incus availability, project, image, bridge, storage, and reaper timer;
- free disk and inode headroom;
- Slack and GitHub configuration without printing secrets;
- Codex authentication status and configured model;
- stale jobs, orphaned instances, and recent backup evidence.

Human output must explain remediation. JSON output must be versioned so a future
host CLI can consume it without scraping logs.
