# labelle-registry

Source of truth for Labelle provider namespace ownership and versioned package-index metadata.

## Status

`providers.json` is live and read by labelle-cli (`labelle providers resolve`) from
`https://raw.githubusercontent.com/labelle-toolkit/labelle-registry/main/providers.json`.
It uses registry schema 2 (provider contract §4, labelle-cli `docs/provider-contract-v1.md`):

- each record pins one release: `package`, `repo`, `version`, the full `commit`, and the
  `sha256` of `https://codeload.github.com/<repo>/tar.gz/<commit>`;
- `namespace` and `targets` repeat what that release's `plugin.labelle` declares, so the CLI
  can name the owner of a missing target or namespace without downloading archives
  (`--accept` checks them against the verified manifest);
- `defaults` lists the exact releases `labelle init` proposes. It is empty for now.

Adding a release means appending a record in a reviewed PR. Records are never edited or
removed once a project may have pinned them.

## Planned responsibilities

- Reviewed namespace and target ownership records.
- Release metadata containing immutable source identity, archive hash, command-contract requirements, supported host metadata, exact bootstrap Zig version, and projectless command declarations.
- Versioned index snapshots published to the existing Labelle release infrastructure.
- Default package selection as data, consistent with the agreed consent/authentication policy.

Provider workflows submit releases only for their assigned namespace. One registry publisher validates and publishes snapshots. Namespace ownership changes require review. Publication credentials belong in CI secret storage, never in this repository.

## Resolution policy

Inside projects, the CLI uses explicit project declarations and locks. Outside projects, initial resolution selects a compatible stable release and records an approved immutable global pin. Subsequent execution uses that pin; an explicitly scoped provider-update operation moves it only after successful verification/preparation. Updating the CLI binary must not silently update providers.

The registry supplies data. Resolution, verification, host-tool compilation, and execution belong to the CLI. It does not host game runtime implementations.

## Implementation references

- [Provider architecture: CLI #406](https://github.com/labelle-toolkit/labelle-cli/issues/406)
- [Agreed ownership and resolution decisions](https://github.com/labelle-toolkit/labelle-cli/issues/406#issuecomment-5843024463)
- [Contract gaps, including target lookup and default-package consent: CLI #411](https://github.com/labelle-toolkit/labelle-cli/issues/411)

Finalize the schema and publishing contract before adding live entries. Creating this repository does not authorize any provider release or namespace automatically.
