# tiny corp operations: public evidence and workflow implications

This note examines the public tiny corp repositories as an operator would. It
does not claim access to tiny corp's private datacenter configuration. The
useful distinction is between mechanisms visible in source, reasonable
inferences from those mechanisms, and capabilities for which there is no public
evidence.

## Executive summary

tiny corp publicly demonstrates a strong **appliance lifecycle**, not a broad
datacenter platform:

1. build hardware-specific Ubuntu images;
2. boot and identify the precise machine variant;
3. configure disks, drivers, services, BMC, and BIOS in numbered stages;
4. validate physical topology and performance rather than merely checking that
   devices exist;
5. burn in the complete machine with real training work while observing
   temperatures;
6. expose the validated machines as hardware-labeled GitHub Actions runners;
7. update deployed boxes through small, ordered migration scripts.

This is refreshingly direct. The source is the runbook, shell commands remain
visible, and the shipped machine is also a CI reference platform. However, the
public stack leaves the fleet layer largely unspecified. That is an opportunity
for 67daisies: retain tiny corp's machine-level honesty while adding explicit
inventory, plans, leases, rollout policy, evidence, and recovery across a rack.

## What is visibly in use

### 1. Hardware-specific Ubuntu images

[`tinyos`](https://github.com/tinygrad/tinyos) builds images with Canonical's
`ubuntu-image`. The Makefile selects red, green, or core variants and stamps
the hardware family into the image. The current template is based on Ubuntu
24.04 and combines an Ubuntu seed, cloud-init, copied setup code, drivers,
developer packages, and systemd services into a raw disk image
([image template](https://github.com/tinygrad/tinyos/blob/master/tinyos.template.yaml)).

The image includes ordinary operator tools such as `ipmitool`, `lshw`,
`lm-sensors`, `jq`, `jc`, `rclone`, `rsync`, RDMA utilities, and
`systemd-journal-remote`. Nix is installed through the Determinate installer,
but the host lifecycle visible here is Ubuntu image plus staged shell scripts,
not NixOS generations.

Operational lesson: image construction and post-boot qualification are one
product workflow. An image is not considered correct just because it boots.

### 2. Staged first boot and staged updates

The setup service sorts numbered scripts and executes every stage newer than
the value stored in `/etc/tinybox-setup-stage`. A successful stage advances the
marker; a special exit code indicates that a reboot is underway. The update
service uses the same pattern with `/etc/tinybox-update-stage`
([repository explanation](https://github.com/tinygrad/tinyos)).

The update mechanism pulls its Git checkout, resets it to the remote branch,
then runs outstanding migration scripts
([implementation](https://github.com/tinygrad/tinyos/blob/master/service/autoupdate-tinybox.sh)).
This is deliberately smaller than a package-management control plane.

Operational lesson: ordered, resumable steps are a good primitive. We should
keep that clarity but make each step declare inputs, mutations, evidence,
rollback limits, and idempotence. Pulling a mutable branch directly should not
be the integrity boundary for our tools.

### 3. BMC, IPMI, and Redfish are normal paths

The customer documentation treats BMC serial-over-LAN and SSH as primary
access paths, not enterprise add-ons
([tinybox docs](https://docs.tinygrad.org/tinybox/)). The OS creates a BMC
password, can update BMC firmware, stages BIOS updates, and uses Redfish to
issue the restart required to complete a BIOS flash. Lower-level fan and power
commands use `ipmitool` directly.

Operational lesson: out-of-band management belongs in the first version of a
bare-metal tool. Provisioning that only works after SSH is available begins too
late in the machine lifecycle.

### 4. Qualification is hardware-class specific

The system-check stage detects the GPU PCI ID and maps it to an expected
machine class. It checks:

- homogeneous GPU models and expected device count;
- expected PCIe link speed and lane width;
- stable bus locations for some variants;
- Resizable BAR support;
- CPU core count and installed RAM;
- NVMe count, PCIe link characteristics, and other system facts;
- multi-GPU all-reduce bandwidth against a minimum threshold.

These checks live in the
[`00012-system-check` stage](https://github.com/tinygrad/tinyos/blob/master/setup/stages/00012-system-check.sh).
The critical idea is that inventory and acceptance are different. Six visible
GPUs can still describe a bad machine if one negotiated the wrong lane width or
the interconnect is too slow.

Operational lesson: our inventory schema needs both **observed facts** and
**declared acceptance constraints**. `doctor` must understand degraded hardware,
not just missing hardware.

### 5. Burn-in uses a real workload and thermal evidence

Provisioning runs matrix tests and a ResNet training workload, retries missing
checkpoints, records CPU and per-GPU temperatures, and rejects a machine that
crosses thermal thresholds
([provisioning stage](https://github.com/tinygrad/tinyos/blob/master/setup/stages/01002-provision__user.sh)).
Older standalone scripts also expose GPU burn, bandwidth, RAID, reset, fan, and
power controls
([tinybox scripts](https://github.com/tinygrad/tinybox)).

Operational lesson: a server is ready only after a workload representative of
its intended role succeeds. Generic CPU, disk, and port checks are necessary
but insufficient for accelerator systems.

### 6. Local fast storage is part of the execution environment

The setup scripts create a four-device RAID0 array and mount it at `/raid`.
Hardware CI jobs reuse `/raid/weights` and `/raid/datasets` rather than fetching
large inputs for every run. The storage is treated as a high-speed cache and
working set, not implicitly as the only durable copy.

Operational lesson: the runner contract needs named local caches with declared
durability and eviction policy. Dataset locality is a scheduling fact for GPU
CI and training workloads.

### 7. GitHub Actions is the visible CI scheduler

Generic tests use GitHub-hosted runners. Hardware benchmarks use self-hosted
runners labeled by machine capability, including `tinybox`, `tinyboxgreen`,
and macOS. Jobs clean up existing GPU processes, link persistent datasets and
weights, run backend-specific tests, assert performance thresholds, and upload
artifacts
([benchmark workflow](https://github.com/tinygrad/tinygrad/blob/master/.github/workflows/benchmark.yml)).
Scheduled MLPerf jobs also target self-hosted tinyboxes.

Operational lesson: tiny corp does not publicly require a bespoke CI language
to exploit its hardware. We should integrate with existing Git forges while
owning worker preparation, isolation, leasing, cleanup, evidence, and recovery.

### 8. The physical product and CI target are the same system

tinyboxes are heavily used in tinygrad CI and described as the best-tested
tinygrad platform
([documentation](https://docs.tinygrad.org/tinybox/)). This closes an unusually
valuable loop:

```text
hardware shipped to users
    -> identical class runs project CI
        -> regressions reveal hardware/software mismatch
            -> fixes improve both product and reference platform
```

Operational lesson: reference hardware should continuously test every layer we
claim to support. Compatibility cannot be a spreadsheet maintained after the
fact.

## What is not established by public evidence

The public repositories do not establish that tiny corp uses any particular
system for:

- rack-level source of truth or asset inventory;
- switch configuration, VLANs, routing, DHCP, DNS, or IPAM;
- power distribution units, circuit capacity, or rack thermal budgets;
- PXE orchestration or concurrent bare-metal installation;
- workload queues, reservations, preemption, or multi-user GPU leasing;
- canary host upgrades or fleet rollback;
- centralized metrics, alert routing, log retention, or incident management;
- replicated storage, backup policy, or disaster recovery;
- secret distribution and credential rotation across a fleet;
- multi-node training placement and network-fabric topology scheduling.

Some of these almost certainly exist as manual procedures, private systems, or
features of external services. We should not attribute a tool or architecture
without evidence. The current exabox description also refers to future,
much-larger hardware; it is not proof of a publicly documented production
orchestrator.

## The workflows 67daisies should own

The unit of design should be a completed operator workflow, not a daemon or a
dashboard.

### Workflow A: receive and identify a machine

```text
scan BMC network
  -> establish machine identity
  -> collect DMI, PCIe, disks, NICs, firmware and sensors
  -> match or create a hardware class
  -> write operator-owned inventory
```

Required evidence: immutable machine identity, BMC identity, board serial,
device topology, firmware versions, management reachability, and raw facts.

### Workflow B: qualify power, cooling, and topology

```text
declare role and acceptance profile
  -> check power domain and limits
  -> validate PCIe links, NUMA, memory and storage
  -> run representative burn-in
  -> retain temperatures, errors and performance baselines
  -> accept or quarantine
```

Required evidence: threshold results, sensor series, PCIe/AER errors,
throughput, test artifact hashes, and a reason for quarantine.

### Workflow C: image or re-image safely

```text
build signed image
  -> produce destructive plan
  -> select console / virtual media / PXE / rescue adapter
  -> install and reboot through BMC
  -> wait for SSH
  -> rerun qualification
  -> record installed generation
```

Required evidence: input lock files, image digest, reviewed disk plan, console
log, boot result, installed generation, and verification report.

### Workflow D: make a machine a CI worker

```text
select accepted host by capability
  -> acquire time-bounded lease
  -> prepare isolation and caches
  -> run job
  -> capture logs, metrics and artifacts
  -> clean GPU processes and workspace
  -> verify baseline health
  -> release or quarantine host
```

Required evidence: lease holder, source revision, environment, resource
selection, cache inputs, result, cleanup, and post-job health.

### Workflow E: deploy and update a fleet

```text
resolve immutable release
  -> calculate per-host plan
  -> update one canary per hardware/failure class
  -> verify workload and machine health
  -> continue with bounded concurrency
  -> stop or roll back on threshold breach
```

Required evidence: artifact digest, host plans, batch boundaries, health gates,
rollback generation, and final fleet convergence.

### Workflow F: repair or replace a failed node

```text
alert or operator report
  -> gather in-band and BMC evidence
  -> compare with healthy peer and baseline
  -> attempt bounded recovery
  -> quarantine if confidence is low
  -> move workload or CI lease
  -> re-image or replace hardware
```

Required evidence: symptoms, attempted operations, hardware errors, affected
work, recovery decision, and replacement identity.

## Contract changes implied by the research

The existing toolchain proposal is directionally correct, but four contracts
become first-class:

1. **Hardware class:** expected topology, firmware ranges, performance floors,
   thermal limits, compatible image, and qualification suite.
2. **Qualification report:** observed facts plus pass, degrade, or quarantine
   decisions and retained evidence.
3. **Lease:** exclusive or shared resource claim with deadline, cleanup policy,
   owner, and recovery behavior.
4. **Failure domain:** rack, power circuit, switch, BMC network, storage domain,
   and other correlated risks used by rollout and placement.

These should remain plain versioned documents. A later daemon may coordinate
concurrent leases, but it must implement the contract rather than own the only
copy of state.

## What to copy and what to improve

Copy from tiny corp:

- hardware classes that encode actual expected topology;
- BMC and firmware management as ordinary lifecycle operations;
- real workload burn-in with thermal and performance thresholds;
- reference hardware used continuously in CI;
- visible shell-level escape hatches;
- local datasets and weights as schedulable cache locality;
- small ordered migrations instead of a sprawling management service.

Improve for a general toolchain:

- produce a plan before disk, RAID, firmware, or power mutations;
- pin update inputs to immutable revisions and verify signatures;
- record structured evidence instead of relying on display text and log files;
- make stages explicitly idempotent, resumable, and rollback-aware;
- separate durable data from disposable RAID/cache by contract;
- rotate BMC and host credentials through operator-selected secret adapters;
- model rack, power, network, and topology failure domains;
- quarantine unhealthy workers automatically after a job;
- support bounded fleet rollout without building Kubernetes.

## Recommended first demonstrator

The first credible demo should recreate and improve the tinybox bring-up path
on one machine or realistic VM fixture:

```text
facts inspect
metal plan
metal apply
metal qualify
run attach github-actions
run smoke-test
doctor host
```

The qualification profile should check CPU, RAM, disks, PCIe topology, network,
temperatures, and a role-specific benchmark. Every command emits structured
evidence, and the destructive operation consumes the exact plan previously
reviewed.

Once that works on one accelerator box, run the same contracts across ten
machines with leases, a canary rollout, and one deliberately degraded PCIe
link. That would demonstrate a product tiny corp could plausibly use rather
than another attractive server dashboard.
