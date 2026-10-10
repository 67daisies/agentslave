# Engineering contract

Changes should make self-hosting more understandable, recoverable, or portable.

## Required properties

- State ownership is explicit.
- External services are behind narrow interfaces.
- Mutations have a dry-run or a clearly documented preview when practical.
- Destructive operations identify an exact target and record an audit event.
- New persistence includes backup and restore behavior.
- New background work includes timeouts, bounded retries, and cleanup.
- Operator errors state what failed, what remains safe, and how to recover.
- Configuration and public JSON contracts are versioned before third parties
  depend on them.
- Tests cover idempotency and interruption, not only the happy path.

## Repository rule

AgentSlave owns the report-to-pull-request workflow. General host management,
service packaging, secrets, networking, and fleet coordination belong in shared
libraries or separate products once their contracts are understood. Do not add
a feature here merely because this is the first repository.

## Change workflow

1. Write the operator outcome and failure behavior.
2. Identify state, credentials, and cleanup responsibilities.
3. Add the smallest contract and focused tests.
4. Exercise the failure path on a disposable environment.
5. Document installation, upgrade, rollback, backup, and restore impact.
6. Split unrelated product changes into separate commits.
