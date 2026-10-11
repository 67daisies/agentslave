# What durable tool builders teach us

This note compares HashiCorp, SST, Xe Iaso's Anubis, the T3 ecosystem, and
tinygrad. The goal is not to copy their products. It is to understand how they
choose a wedge, make a technical abstraction credible, distribute it, and earn
the right to build a larger system.

The conclusion is narrow: 67daisies should not begin by building another
self-hosted PaaS. It should build a family of small, local tools that turns raw
machines into reproducible infrastructure. Deployment is part of that
workflow, alongside imaging, hardware discovery, networking, CI/CD,
diagnostics, backup, restore, upgrade, and rollback. There is no 67daisies
account or hosted control plane.

## Comparative summary

| Builder | Initial wedge | Durable abstraction | Distribution | Business layer | Lesson for 67daisies |
| --- | --- | --- | --- | --- | --- |
| HashiCorp | Painful operator workflows | Consistent CLI, declarative state, providers | Separate open tools and an integration ecosystem | Enterprise and hosted operation | Build workflows, not a technology showcase; keep each tool bounded |
| SST | Deploying web apps to AWS was too low-level | Type-safe components over provider primitives | Project-local CLI, examples, framework integrations | Optional hosted console | Hide incidental complexity while preserving full escape hatches |
| Anubis | Small sites were overwhelmed by AI crawlers | A lightweight reverse-proxy policy boundary | One memorable tool, packages, strong voice, honest defaults | Sponsors and paid support/community | Acute pain and identity can outrun a broad platform roadmap |
| T3 | Starting a good TypeScript app required too many decisions | Opinionated, optional modules and explicit axioms | One-command scaffold plus creator community | Products built for the same audience | Encode judgment, do not accumulate features; community is a product surface |
| tinygrad | ML stacks and accelerator support became opaque | Tiny visible core and a narrow backend interface | Familiar API, source-first development, bounties | Reference hardware | Complexity is a cost; reference hardware can fund and continuously test open software |

## HashiCorp: a family of workflows

HashiCorp's strongest idea is not HCL or Terraform. It is the decision to make
each operational workflow a separate product while giving the family a common
mental model. The [Tao of HashiCorp](https://www.hashicorp.com/en/resources/what-is-the-tao-of-hashicorp)
names three especially useful principles: workflows rather than technologies,
infrastructure represented as code, and pragmatism. A user could learn the
shape of one tool and recognize the others without accepting a single giant
platform.

Terraform also demonstrates how an extension boundary becomes distribution.
Providers let external vendors and users add breadth while the core team owns
the planning and state model. That is a better moat than a long built-in
service list.

The caution is trust. HashiCorp's 2023
[move from MPL to the Business Source License](https://www.hashicorp.com/en/blog/hashicorp-adopts-business-source-license)
protected its commercial investment, but it also showed that infrastructure
users treat licensing and the ability to run locally as part of the product's
reliability. A technically replaceable tool can still create organizational
dependence.

Copy:

- Give every tool the same verbs and lifecycle: `init`, `doctor`, `plan`,
  `apply`, `status`, `backup`, `restore`, `upgrade`, and `rollback`.
- Store desired configuration as reviewable text and produce a plan before a
  privileged change.
- Put integrations behind a small, versioned contract.
- Split products when their state, failure domain, or operator is different.

Avoid:

- Expanding to every provider before the central workflow is excellent.
- Making a hosted service the canonical holder of state.
- Treating the open-source license as a temporary acquisition channel.

## SST: compress complexity for one audience

SST succeeds by being explicit about its user: an application developer, not a
platform engineer. Its [documentation](https://sst.dev/docs) explains that
high-level components turn many low-level cloud resources into concepts such
as a site, function, or database. The configuration is TypeScript, the CLI is
installed with the project, and users can transform the underlying resources
when the abstraction is insufficient.

SST also reuses rather than recreates the provider layer. It uses Pulumi's
engine and bridges Terraform providers, then spends its effort on the developer
workflow. Its [components model](https://sst.dev/docs/components/) is a useful
example of a small curated layer over a much larger ecosystem. The optional
[SST Console](https://sst.dev/docs/console/) adds team visibility, logs,
deployments, and alerts, but the CLI and deployed application do not require
it.

SST's migration away from CDK is another lesson: an excellent interface cannot
fully insulate a product from deep dependencies. We need to own the lifecycle
contract and keep the generated host configuration readable.

Copy:

- Design for one operator persona and use their familiar language.
- Offer high-level components with sensible defaults and a documented escape
  hatch.
- Pin the tool version with the project so configuration and behavior travel
  together.
- Make hosted collaboration genuinely optional.

Avoid:

- Stacking several engines until failure messages become archaeology.
- Claiming portability while testing one provider well and the rest lightly.
- Hiding generated configuration or the native recovery path.

## Anubis: sharp pain, cheap adoption, memorable identity

Xe Iaso's Anubis is intentionally not a general security platform. The
[project](https://github.com/TecharoHQ/anubis) protects small websites from
aggressive AI crawlers with a lightweight challenge and policy layer. Its
README calls the mechanism a "nuclear response," warns that good bots may be
blocked, and says Cloudflare may be enough for many users. That honesty improves
trust because the tool describes its boundary instead of promising perfect bot
detection.

Anubis couples technical focus with memorable language, art, and a moral
position: protect the small internet. The package is cheap to try, useful
without an account, and support can be funded without placing a vendor service
in the request path. Its rapid adoption is evidence that a severe, current
problem plus a legible identity can distribute a small utility more effectively
than a broad feature matrix.

Copy:

- Lead with one sentence describing the pain and the protected user.
- Ship a useful binary or package that does not need registration.
- State false positives, limits, and failure modes in the first page of docs.
- Give the project a human voice and a constituency worth protecting.

Avoid:

- Turning a successful utility into an unrelated suite too early.
- Pretending policy defaults are universally correct.
- Letting the mascot or story substitute for recovery tests.

## T3: opinionated assembly and a media flywheel

The T3 ecosystem shows the value of publishing taste. `create-t3-app` is not a
large framework; it is an interactive assembler for optional pieces. Its
[axioms](https://github.com/t3-oss/create-t3-app) are unusually concrete:
solve actual problems, "bleed responsibly" by using riskier technology only
where replacement is cheap, and preserve end-to-end type safety. These rules
let maintainers reject attractive additions that weaken the product.

The one-command entry point is important, but Theo Browne's community and media
work are also part of the system. Explanations, strong opinions, live product
building, and feedback create a distribution loop for open tools and adjacent
commercial products. The current product family listed on
[Theo's GitHub profile](https://github.com/t3dotgg) includes T3 Chat, T3 Code,
and UploadThing: different products aimed at an audience already gathered
around the same style of building.

Copy:

- Publish a short set of axioms that can reject features.
- Make every optional component removable.
- Let users reach a useful result with one command, then teach what happened.
- Build in public and make incident analysis and tradeoffs good content.

Avoid:

- Copying creator-led launch energy without the operational discipline that
  infrastructure requires.
- Betting the durable state layer on fashionable technology.
- Measuring attention as if it were successful restores or safe upgrades.

## tinygrad: a small core and a real machine

tinygrad's developer documentation reduces the framework to
[four pieces](https://docs.tinygrad.org/developer/developer/): frontend,
scheduler, lowering, and execution. Its repository says a new accelerator
needs roughly 25 low-level operations and explicitly rejects large, complex
diffs. Bug fixes require regression tests, performance claims require
benchmarks, and features face a usefulness-versus-lines-added test. This is not
minimalism as branding; it is a maintenance strategy.

The commercial move is particularly relevant. The
[tinybox](https://docs.tinygrad.org/tinybox/) is hardware sold by tiny corp,
used heavily in tinygrad CI, and presented without a cloud account or tracking.
The reference product creates revenue while forcing the open software to work
on a complete, real system. Customers own the machine and can run other
frameworks on it.

Copy:

- Define a tiny runtime contract and make every backend conform to it.
- Require small changes, regression tests, and measured performance claims.
- Use the reference production setup in CI and destructive recovery tests.
- Eventually sell a known-good box or server image without restricting the
  software to it.

Avoid:

- Confusing low line count with readable simplicity.
- Supporting many backends before the contract is stable.
- Selling hardware before installation, replacement, and support logistics are
  understood.

## The market gap

The broad self-hosted PaaS is already a competitive category. Coolify describes
itself as a control plane that connects over SSH and runs standard Docker
workloads, with self-hosted and managed options
([documentation](https://coolify.io/docs/core/what-is-coolify)). Dokploy offers
the same basic promise—applications and databases on customer-owned servers,
without Kubernetes—and also sells a managed control plane
([overview](https://dokploy.com/self-hosted-paas)). Competing with their
dashboards, Git deployments, templates, domains, and container management would
put 67daisies into a feature-count race.

The less well-served question is what happens after the easy deployment:

- Can the owner prove that backups can restore onto a clean replacement box?
- Can an upgrade show its state migrations and roll back safely?
- Can the system explain a full disk, expired certificate, broken DNS record,
  failed health check, or abandoned workload?
- Can the business operate when the vendor account, dashboard, or original
  engineer is unavailable?
- Can a service move between a rented VPS and owned hardware using the same
  declared contract?

That is the wedge. We should make ownership operationally credible, not merely
make a container appear on a VPS.

## Proposed product thesis

**67daisies builds the operator toolbox for boxes you control: from one small
business server to a rack of bare-metal or accelerator machines.**

The first tool should be a local CLI for one existing NixOS host. A repository
contains the desired services; the CLI connects over SSH, explains a plan,
applies a NixOS generation, verifies service contracts, records an audit event,
and leaves the host operable through standard tools. Later tools should image
raw machines, manage inventory, attach CI workers, and operate a rack using the
same host contract rather than introducing a central platform.

The smallest complete workflow is:

```text
init -> doctor -> plan -> apply -> verify
                          |          |
                          |          +-> backup -> restore drill
                          +-> failed apply -> rollback
```

The first supported topology is deliberately constrained:

- one NixOS server on Hetzner or ordinary x86 hardware;
- systemd for supervision;
- Incus for workloads that need an isolation boundary;
- PostgreSQL and filesystem/object storage for durable state;
- Caddy or another replaceable reverse proxy for TLS and routing;
- SSH as the remote administration transport;
- readable generated Nix committed beside the service manifest.

Normal operators should use a small declarative manifest. TypeScript should be
available for package authors and advanced composition, but recovery must not
depend on evaluating an arbitrary application program. The generated Nix and
service contract are the durable escape hatch.

## Product axioms

1. **There is no account.** No 67daisies service is required to discover,
   image, boot, serve, build, deploy, back up, restore, or administer a box.
2. **A restore is the proof of a backup.** Backup success without a periodic
   clean-host restore is an unverified claim.
3. **Plans precede privilege.** Every mutation names restarts, migrations,
   network exposure, storage changes, and rollback limits.
4. **One box is production.** Clustering is not a prerequisite disguised as
   reliability.
5. **Generated state is inspectable.** Users can read, version, export, and
   operate the native NixOS configuration.
6. **AI advises; deterministic code mutates.** Agents can investigate and
   propose plans, but lifecycle correctness lives in typed, tested operations.
7. **Risk belongs at replaceable edges.** Durable data uses boring formats and
   tools; experiments live in adapters and user interfaces.
8. **Small tools share verbs, not databases.** AgentSlave and future tools may
   share contracts without becoming a monolith.

## What to build first

Do not turn AgentSlave into the platform. It is one future consumer of the same
runner and host contracts. The next sequence should be:

1. Specify plain inventory, plan, result, and audit formats independent of any
   product or daemon.
2. Build one small bare-metal tool that discovers a machine, previews disk and
   network changes, installs NixOS, verifies SSH, and writes inventory.
3. Exercise the same flow against a virtual machine, a rented bare-metal
   server, and a BMC-controlled physical host.
4. Build CI execution and deployment as separate tools consuming the inventory
   and result formats.
5. Adapt AgentSlave to consume the runner and diagnostics contracts rather than
   extracting the product family from AgentSlave internals.

## Ninety-day validation

The target is not a polished dashboard. It is evidence that five small teams
can own a production box without calling its original installer.

Success measures:

- clean server to healthy first service in under 15 minutes;
- a human-readable plan before every privileged change;
- full restore to a replacement host in under 30 minutes;
- one-command rollback after a deliberately broken upgrade;
- no required 67daisies account, telemetry endpoint, or runtime connection;
- a second operator can diagnose five seeded failures from `doctor` output;
- generated Nix remains sufficient to operate the host without the CLI.

Interview and pilot questions should focus on costly incidents rather than
feature requests: the last failed restore, surprise bill, expired certificate,
abandoned VPS, dependency upgrade, or provider migration. If teams will not run
a restore drill with us, they are not validating the product's core claim.

## Sustainable work without hostage-taking

Keep the tools and contracts under a durable open-source license. If a business
forms around them, sell labor and confidence around the owned system:

- managed update channels and compatibility testing;
- encrypted off-site backup storage and restore-drill reports;
- certified service packages and migration support;
- a supported reference server image, and later a known-good physical box;
- response-time support contracts.

These are services around the tools, not an account required by the tools.
Desired state, keys, inventory, and recovery instructions remain entirely with
the operator.

## Decision

Continue hardening AgentSlave as the first reference workload, but frame the
larger company around **software ownership with tested recovery**. Do not enter
the generic self-hosted PaaS race. Earn expansion one workflow at a time:

1. make one box understandable;
2. make one service recoverable;
3. make upgrades reversible;
4. make packages composable;
5. only then make several boxes coordinated.

That direction combines HashiCorp's workflow discipline, SST's compression of
provider complexity, Anubis's sharp and human wedge, T3's explicit taste, and
tinygrad's tiny core plus reference machine—without inheriting their least
useful incentives.
