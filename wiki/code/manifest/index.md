# manifest (version shell)

Code-free package whose only purpose is holding the canonical Manifest version number for Docker image releases.

## Contents

```
packages/manifest/
├── package.json    # { "name": "manifest", "version": "6.0.1", "private": true }
└── README.md
```

No `src/`, no tests, no dependencies.

## Why It Exists

The monorepo has three private packages (`manifest-backend`, `manifest-frontend`, `manifest-shared`) that are never published to npm. Docker is the only release artifact. To give changesets a single bumping target, a shell package `manifest` exists solely to receive the version bump.

`.changeset/config.json`:
```json
{
  "ignore": ["manifest-backend", "manifest-frontend", "manifest-shared"]
}
```

`npx changeset` only offers `manifest` as a target. Bumping it → Docker release.

## Current Version

`6.0.1`

## See Also

- [../../deployment.md](../../deployment.md) — release process
