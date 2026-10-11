# Contributing

This repository is a monorepo for Pi coding-agent packages. Keep package boundaries clear so a package can be installed on its own or included through the root monorepo manifest.

## Package conventions

- Put each package in `packages/<kebab-case-name>/`.
- Give each package its own `package.json` and `README.md`.
- Use conventional Pi resource directories (`extensions/`, `skills/`, `prompts/`, and `themes/`) and declare nonstandard paths in that package's `pi` manifest.
- The root `package.json` is a private aggregator. Its `pi` manifest globs package resources, while npm workspaces provide a shared development layout. Do not put package-specific extension code at the repository root.
- Keep package versions in each package's `package.json`. The private root aggregator is not intended for npm publication.
- Declare Pi-provided runtime modules such as `@earendil-works/pi-coding-agent` in `peerDependencies`, not `dependencies`; do not vendor Pi's runtime.
- Add package-specific dependencies only when needed, and document their purpose.

## Adding a package

1. Create `packages/<name>/` with a manifest and README.
2. Add its resources under the conventional directories and ensure the root `pi` manifest includes any nonstandard resource paths.
3. Add the package to the table in the root README.
4. Document installation, supported Pi behavior, configuration/auth requirements, side effects, and any model or network usage.
5. Test the package directly and through the root package manifest before committing.

## Local validation

For a single extension, Pi can load its entry point directly:

```sh
pi --extension ./packages/<name>/extensions/<entry>.ts --list-models
```

From the repository root, package paths and JSON manifests can also be checked with:

```sh
node -e 'JSON.parse(require("node:fs").readFileSync("package.json", "utf8"))'
node -e 'for (const p of ["packages/rename-proc/package.json"]) JSON.parse(require("node:fs").readFileSync(p, "utf8"))'
```

For packages that call models, test both the normal model path and any fallback path. Never commit credentials, local settings, session transcripts, or generated dependencies.

## Releases and installs

- Install the whole monorepo from Git with `pi install git:github.com/rajivmehtaflex/pi-power-utility@<ref>`.
- Install one package from a checkout with `pi install ./packages/<name>`.
- Pin Git installs to a release tag or commit when reproducibility matters.
- Keep the root manifest in sync with the resources intended to ship from the monorepo install.
