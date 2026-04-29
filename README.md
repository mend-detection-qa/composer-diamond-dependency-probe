# diamond-dependency

## Feature exercised

This probe exercises Composer's diamond-dependency resolution: two direct dependencies
(`monolog/monolog` and `symfony/cache`) both declare a constraint on `psr/log`, and
Composer resolves a single version satisfying both constraints.

## Diamond shape

```
             root
            /    \
  monolog/monolog  symfony/cache
      (^2|^3)         (^1|^2|^3)
            \    /
           psr/log
          (resolved: 3.0.2)
```

`psr/log` is the shared apex. Composer selects version 3.0.2 — the newest version
that satisfies both `^2.0 || ^3.0` (monolog) and `^1.1|^2|^3` (symfony/cache).

## Resolved dependency tree

| Package | Version | Scope | Direct |
|---------|---------|-------|--------|
| monolog/monolog | 3.10.0 | require | yes |
| symfony/cache | v6.4.36 | require | yes |
| psr/cache | 3.0.0 | require | no |
| psr/container | 2.0.2 | require | no |
| psr/log | 3.0.2 | require | no |
| symfony/cache-contracts | v3.6.0 | require | no |
| symfony/deprecation-contracts | v3.6.0 | require | no |
| symfony/service-contracts | v3.6.1 | require | no |
| symfony/var-exporter | v7.4.8 | require | no |

## Expected dependency tree

Key Mend detection expectations:

- `psr/log` must appear exactly **once** in Mend's library list (not once per path).
- Both `monolog/monolog` and `symfony/cache` must list `psr/log` as a child in the
  dependency tree — this encodes both parent relationships of the diamond apex.
- All 9 packages sourced from Packagist (registry), no VCS or path sources.
- No `packages-dev` entries; all packages are production scope.

## Failure modes targeted

- **Duplicate C entries**: Mend reports `psr/log` twice, once as child of monolog
  and once as child of symfony/cache, counted as two distinct libraries.
- **Wrong version for C**: Mend resolves a different `psr/log` version instead
  of the single Composer-resolved 3.0.2.
- **Missing parent link**: Mend links `psr/log` to only one of the two parents,
  losing the diamond structure.

## Probe metadata

- pattern: diamond-dependency
- pm: composer
- schema_version: "1.0"
- lockfile: composer.lock
- generated: 2026-04-29
- composer_version: ">=2.6"
- php_version: ">=8.1"