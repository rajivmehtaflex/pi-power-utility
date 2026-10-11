# Pi Power Utility

A monorepo for reusable [Pi](https://github.com/earendil-works/pi) coding-agent packages: extensions, skills, prompt templates, themes, and related utilities.

Each independently installable package lives under `packages/<package-name>/`. The repository root is also a private Pi-package aggregator: its `package.json` manifest discovers resources in those packages, so a single Git install can enable the monorepo's declared resources.

## Packages

| Package | Description | Docs |
|---|---|---|
| [`rename-proc`](./packages/rename-proc/) | Automatically names the active session from its conversation | [README](./packages/rename-proc/README.md) |

## Install

Install all resources declared by the monorepo root:

```sh
pi install git:github.com/rajivmehtaflex/pi-power-utility@main
```

To install an individual package from a local checkout:

```sh
pi install ./packages/rename-proc
```

Package installation loads executable extension code. Review the source and trust the package before installing it. Project-scoped package declarations require project trust; see the [Pi package gallery](https://pi.dev/packages) and Pi's package documentation.

## Develop

Clone the repository and try an extension directly without adding it to settings:

```sh
gh repo clone rajivmehtaflex/pi-power-utility
cd pi-power-utility
pi --extension ./packages/rename-proc/extensions/rename-proc.ts
```

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the package layout, monorepo conventions, and validation steps.

## Repository layout

```text
packages/                 Independently installable Pi packages
  rename-proc/             Session-naming extension package
    extensions/            Pi extension entry points
    package.json           Package metadata and Pi resource manifest
    README.md              Package-specific usage and behavior
package.json               Private workspace + root Pi-package manifest
CONTRIBUTING.md            Guidance for adding and changing packages
LICENSE                    Apache-2.0
```

## License

Apache-2.0. See [LICENSE](./LICENSE).
