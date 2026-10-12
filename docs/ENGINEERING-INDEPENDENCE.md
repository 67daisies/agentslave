# Engineering independence and sustainable free software

The objective is not infinite scale as a slogan. The objective is a toolchain
whose architecture does not impose an early ceiling or require a different
operating model at each order of magnitude.

One Hetzner server should be a complete system. A hundred servers should be the
same host operations with inventory selection and bounded concurrency. Ten
thousand servers may require sharded coordinators and lease services, but those
services must implement the same public contracts and must not become part of
the workload runtime.

The project can prioritize engineering excellence for years without pretending
that sustainability is irrelevant. Sustainability is itself a reliability
property: users should know that releases, security work, testing, and
maintenance can continue without selling the project to a hyperscaler.

## Projects and engineers worth studying

### SourceHut and Drew DeVault: the business is a tool for free software

SourceHut is the closest cultural reference. Its platform is entirely free
software. Revenue comes from subscriptions for operating the service and from
consulting that is required to produce free software
([business model](https://sourcehut.org/blog/2022-01-09-how-does-our-business-work/)).
The hosted forge supplies predictable base revenue; consulting funds larger or
more experimental work. SourceHut has published financial reports and treats
profitability as an operational requirement rather than the purpose of the
project.

What to copy:

- The company exists to sustain the engineering mission.
- Paid consulting improves public software; it does not create private forks.
- Operational transparency includes finances, staffing, reliability, and
  infrastructure costs.
- Simple tools, plain protocols, email and Git remain first-class.
- Revenue is reinvested into a broader free-software ecosystem.

What not to copy blindly:

- A hosted forge naturally creates accounts and recurring service operations.
  Our core tools should not require either.
- Low salaries and founder sacrifice are not a durable systems architecture.

### SQLite and D. Richard Hipp: sell certainty around an uncompromised core

SQLite may be the strongest sustainability reference for pure engineering. The
core remains public domain while the developers sell private support,
guaranteed response, custom testing, specialized extensions, and consortium
membership. Consortium members fund continuing independence and receive direct
engineering access and tests for their exact configurations
([support model](https://sqlite.org/prosupport.html),
[consortium](https://www.sqlite.org/consortium.html)).

The project explicitly designs for continuity through 2050. The commercial
offering is not "unlock more database." It is expert attention, assurance,
testing, compatibility, and institutional continuity.

What to copy:

- Keep the essential artifact permanently useful and independent.
- Sell response time, custom qualification, long-term support, and engineering
  attention.
- Let major users fund the project's independence without controlling its
  technical direction.
- Treat exhaustive tests and compatibility promises as valuable products.

For 67daisies, a future infrastructure consortium could fund qualification of
specific servers, NICs, accelerators, firmware, and workload profiles while all
resulting generic fixes remain public.

### Caddy and Matt Holt: no feature gates, professional sponsorship

Caddy keeps all functionality open and asks professional users to sponsor the
project in exchange for support and development assurances
([sponsorship model](https://caddyserver.com/sponsor)). This is simple and
aligned: companies relying on the software fund its maintenance without buying
permission to run more instances.

What to copy:

- Never count machines, cores, users, or deployments.
- Charge for assurance and a relationship with maintainers.
- Set the expectation that professional dependence deserves professional
  support.

### Proxmox: complete software plus paid stability and support

Proxmox ships a complete open infrastructure product. Subscriptions buy access
to the most heavily tested package repository and different support response
levels; features are not divided into community and enterprise editions
([subscription plans](https://www.proxmox.com/en/products/proxmox-virtual-environment/pricing)).

What to copy:

- The free version remains operationally complete.
- Paid value is conservative release engineering, security updates, support,
  and predictable response.
- Pricing maps to support burden rather than artificial feature gates.

What to improve:

- A production-quality stable channel should eventually be broadly available;
  payment should fund stronger guarantees, longer maintenance, mirrors, and
  direct assistance rather than make safe packages feel withheld.

### Oxide and Bryan Cantrill: sell the computer, open the software

Oxide designs the rack as a computer: hardware, firmware, networking, storage,
control plane, API, CLI, and console are one engineered product. Its software
is open because the company sells the integrated machine, with no separate
software licensing
([Cloud Computer](https://oxide.computer/blog/the-cloud-computer)). It also
publishes design reasoning through RFDs, preserving the context behind systems
decisions rather than only releasing code
([RFD workflow](https://oxide.computer/blog/a-tool-for-discussion)).

Oxide is not a near-term business template—it required substantial capital—but
it is the long-term engineering standard:

- design hardware and software failure handling together;
- make API and CLI behavior the contract, with the UI as another client;
- publish architectural reasoning;
- include all system software with the machine;
- give operators hyperscaler ergonomics without a hyperscaler dependency.

### tiny corp and George Hotz: reference hardware closes the test loop

tiny corp sells the machine, keeps the software open, uses the same hardware in
CI, and pushes down the stack when drivers or firmware block the result. The
lesson is not to manufacture hardware immediately. It is to maintain a
reference machine class that continuously tests the claimed experience.

### 37signals and Kamal: fund tools through your own operational need

Kamal is open source and explicitly unrelated to a commercial control plane.
37signals built it to move its own products from public cloud infrastructure to
owned hardware, and the same tool accepts ordinary Ubuntu servers reachable by
SSH
([Kamal vision](https://kamal-deploy.org/)).

What to copy:

- Be the demanding first user.
- Keep the abstraction thin enough that operators can see the Docker, SSH, and
  proxy commands underneath it.
- Prefer an imperative, understandable deployment workflow over a permanently
  reconciling platform when the problem does not require one.
- Let avoided cloud cost fund the tool even before external revenue exists.

AgentSlave and Aeomatic can play this role: every operational problem on our
Hetzner machine becomes a missing capability or test in the public toolchain.

### Nix community and Numtide: build the missing primitives, sell expertise

[`nixos-anywhere`](https://github.com/nix-community/nixos-anywhere) installs
NixOS remotely over SSH and uses
[`disko`](https://github.com/nix-community/disko) for declarative disk layout.
The tools are MIT licensed and supported through community work and commercial
Nix consulting. Their maintainers sell integration, support, training, and
custom development rather than a restricted binary.

What to copy:

- Compose existing excellent primitives instead of absorbing them.
- Upstream fixes whenever the general-purpose layer is missing something.
- Make dangerous operations explicit and reproducible.
- Sell the expertise required to integrate the tools into real organizations.

### Sidero Labs and Talos: study the API, reject the mandatory coordinator

Talos demonstrates an immutable, API-managed machine operating system and
bare-metal registration through ISO or PXE. Omni demonstrates a polished fleet
workflow, including discovering BMC details through the installed machine
([bare-metal provider](https://omni.siderolabs.com/tutorials/setting-up-the-bare-metal-infrastructure-provider)).

What to study:

- machine APIs designed for automation rather than interactive shell mutation;
- immutable hosts and reproducible reset;
- image factories and bare-metal enrollment.

What not to inherit:

- Kubernetes as the assumed destination;
- an identity-backed hosted coordinator as the easiest path;
- a control plane required for ordinary machine operation.

### Plausible: useful economics, wrong primary shape for this project

Plausible demonstrates that an AGPL self-hosted product can be funded by a
managed service and remain independent and profitable
([Community Edition](https://plausible.io/blog/community-edition)). That model
is legitimate, but it should not drive our architecture. Our mission is to
remove required accounts and hosted runtime dependencies, not operate the most
convenient cloud version of our tools.

## The synthesis for 67daisies

For the first several years:

1. **Kamal:** build tools we need to leave expensive and opaque cloud layers.
2. **SourceHut/Numtide:** accept only consulting and integrations that improve
   the public toolchain.
3. **Caddy:** offer sponsorship and direct maintainer support with no feature
   gates or machine counting.
4. **SQLite:** eventually sell qualification, compatibility, long-term support,
   and consortium membership to organizations that depend on the tools.
5. **Proxmox:** maintain conservative release channels and backports.
6. **tiny corp/Oxide:** later sell certified images, known-good servers, or an
   integrated box while keeping all essential software open.

Do not begin with:

- venture-scale growth targets;
- a hosted dashboard;
- usage billing;
- enterprise-only operational features;
- marketplace revenue;
- relicensable source-available terms;
- a sales organization ahead of an exceptional tool.

## Hetzner-first, not Hetzner-shaped

Hetzner is an excellent proving ground because it exposes two useful ends of the
market:

- quickly created virtual machines through Hetzner Cloud;
- dedicated servers managed through Robot, rescue mode, reset facilities, and
  optional remote console workflows.

Hetzner documents custom installation through its rescue system and
`installimage`, or through KVM virtual media for arbitrary systems
([custom image documentation](https://docs.hetzner.com/robot/dedicated-server/operating-systems/installing-custom-images/)).
`nixos-anywhere` already supports remote installation to Hetzner bare metal and
other SSH-accessible machines.

Our provider interface should describe capabilities rather than pretend every
provider is identical:

```text
allocate?
destroy?
power-cycle
enter-rescue?
attach-image?
serial-console?
set-rdns?
configure-firewall?
list-network-facts
```

A Hetzner Cloud adapter implements ephemeral allocation. A Robot adapter
implements rescue, reset, and server metadata. A Redfish adapter implements
BMC operations. A local adapter may only know SSH and require the operator to
perform power actions.

The core lifecycle remains provider-neutral:

```text
acquire machine
  -> inspect facts
  -> create reviewed disk and network plan
  -> enter installer or rescue environment
  -> invoke nixos-anywhere + disko
  -> reboot and verify
  -> qualify hardware and workload
  -> write inventory and evidence
```

### First supported path

Support one excellent path before broad adapters:

1. Hetzner Cloud x86 VM for fast integration tests.
2. Hetzner dedicated server through Robot rescue mode.
3. NixOS using pinned `nixos-anywhere` and `disko` inputs.
4. One public IPv4/IPv6 network profile and one private-network profile.
5. Local encrypted secrets and operator-owned SSH keys.
6. Host qualification, backup target, CI worker attachment, application
   deployment, rollback, and replacement.

The acceptance test is not "the machine answers SSH." It is:

> Starting with only the Git repository, Hetzner credentials, and recovery
> keys, replace a failed production machine and restore its declared workloads
> without opening the Hetzner web console.

## Scaling without an infinite-scale fantasy

Scale should be introduced as independent layers.

### One host

- Files and Git are inventory.
- The CLI connects directly over SSH.
- Plans and JSONL events are local.
- There is no daemon.

### Tens to hundreds of hosts

- Inventory selectors choose hosts.
- Commands run with bounded concurrency.
- Failure domains determine canaries and batch limits.
- Time-bounded leases protect scarce machines and accelerators.
- Events can be indexed, but files remain the recovery source.

### Thousands of hosts

- Inventory and events are partitioned by site or failure domain.
- Lease coordinators may be replicated per site.
- Artifact and Nix caches are regional.
- Work is queued close to target machines.
- No operation needs global consensus across the fleet.
- Losing a coordinator stops new work but does not stop running workloads.

The invariant is more important than the implementation:

> Scale adds indexes, caches, queues, and coordinators around the host
> contract. It never replaces the host contract or moves the only copy of
> desired state into a service.

## Governance and licensing posture

- Use an OSI-approved license and publish a no-surprise-relicensing pledge.
- Keep protocols, schemas, CLI behavior, and conformance tests public.
- Accept a DCO rather than requiring contributors to grant broad relicensing
  rights.
- Publish design proposals and important rejected alternatives.
- Keep trademarks available for compatibility descriptions while protecting
  users from misleading commercial distributions.
- Never require a contributor agreement that exists primarily to enable an
  open-core or source-available pivot.

The exact license should be decided before accepting outside contributions.
MPL-2.0 is a reasonable default for the tools because it preserves changes to
core files while permitting broad integration. Apache-2.0 is reasonable for
schemas and SDKs that should spread with minimal friction. AGPL should be used
only if a network service later has a clear need for network copyleft; it should
not be used as a substitute for a sustainable engineering model.

## Decision

Build for Hetzner first. Make the provider experience exceptional enough that a
small team can leave higher-level cloud products entirely. Preserve capability-
based adapters so Hetzner never becomes a hidden control-plane dependency.

Optimize for ten years of excellent engineering, not infinite scale or an
immediate business. The sustainable outcome is a complete free toolchain funded
by the people who need assurance, integration, qualification, and long-term
maintenance—not by withholding the ability to operate their own machines.
