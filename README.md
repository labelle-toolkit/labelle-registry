# labelle-registry

A GitHub repository containing one small provider manifest: [providers.json](providers.json).
The CLI reads this file directly from GitHub, or from a local checkout for offline use.
There is no registry service, R2 bucket, generated snapshot, or publishing workflow.

The manifest is intentionally empty until working provider releases are available.
Repository scaffolds are not usable releases and must not be listed.

## Adding a release

Open a normal pull request adding one record to `providers`:

```json
{
  "package": "labelle-example",
  "repo": "labelle-toolkit/labelle-example",
  "version": "1.0.0",
  "commit": "<40 lowercase hexadecimal characters>",
  "sha256": "<64 lowercase hexadecimal characters>"
}
```

Replace the placeholders with real values. `commit` is an exact Git commit,
not a tag or branch. Compute SHA-256 over the compressed bytes downloaded from
`https://codeload.github.com/<repo>/tar.gz/<commit>`. This is an archive hash,
not Zig's package-content hash. GitHub may change generated archive bytes;
a changed hash must fail verification, never be silently accepted.

Each package/version pair occurs once, and a package belongs to one repository.
Versions are exact stable semver without a `v` prefix, prerelease or build suffix.
Published release records must not be repointed; add a new version for new code.
Unknown fields and duplicate JSON keys are rejected by the CLI.

The verified archive's `plugin.labelle` supplies command-contract requirements,
commands, namespace and targets. These are not duplicated in this manifest.
The CLI validates declarations and ownership before writing project pins.
Archives must contain a single root directory with regular files/directories;
links and escaping, conflicting or non-portable paths are rejected.

## Project usage

With the provider declared at an exact version in `project.labelle`:

```text
labelle providers resolve
labelle providers resolve --accept
```

The first command previews the selected GitHub commits and hashes. The second
downloads and verifies the archives, validates their manifests, and writes
`labelle.providers.lock` atomically. It does not execute package build scripts.
Commit this companion lock alongside `labelle.lock`; the latter continues to
record the project's ordinary dependencies and is not rewritten by resolution.

Normal provider commands use only project pins and cached, hash-verified
archives. They never consult this repository for updates. A version change
requires an explicit project update and another resolve. CLI self-update does
not move provider pins. Failed verification leaves the previous lock intact.

Offline resolution uses a local copy of `providers.json` and already cached
archives:

```text
labelle providers resolve /path/to/providers.json --offline --accept
```

Projectless bootstrap and default-package selection are future CLI work; they
do not require a separate registry service. See
[CLI #406](https://github.com/labelle-toolkit/labelle-cli/issues/406) and
[the provider contract](https://github.com/labelle-toolkit/labelle-cli/blob/main/docs/provider-contract-v1.md).
