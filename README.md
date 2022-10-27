# GitHub actions
A collection of GitHub actions for use within `ottofeller` projects.

# `npm-run`
Provides the following functionality:
- Check out a repo
- Setting up a distribution of the requested Node.js version
- Install an app
- Run an npm script
- The dir to the app may be specified

## Usage
The action sets up node, installs npm packages and runs a command supplied in inputs, see [action.yml](npm-run/action.yml). See [test-npm-run.yml](.github/workflows/test-npm-run.yml) for an example of the action usage.

NOTE: You do not need to check out the repo before running the action. Moreover if you do check it out, the result will be thrown away as the action checks out from a provided ref (with default branch as a fallback).

**Basic:**
```yaml
steps:
- uses: ottofeller/github-actions/npm-run@main
  with:
    command: npm run test:all -- --verbose
```

**Advanced:**
```yaml
steps:
- uses: ottofeller/github-actions/npm-run@main
  with:
    node-version: 16
    registry-url: https://npm.pkg.github.com/
    scope: @npmscope
    directory: app
    command: npm run test:all -- --verbose
```

## Supported syntax
The `node-version`, `registry-url`, `scope` inputs are optional and are similar to those in the `setup-node` action. The latter is used under the hood and the inputs are fed into setup action. See [setup-node](https://github.com/actions/setup-node#readme) docs for details.\
The `directory` is optional and defaults to `./`.\
The only required input is `command` which is the command that will be run in `bash` after installation of node and dependencies.

# `create-release`
Provides the following functionality:
- Check out a repo
- Find the latest tag and bumps it with a new patch version
- Creates a draft release with the new tag
- Outputs ID of the created release

## Usage
The action searches your repo for the most recent tag and creates a new one with patch update, then a draft release is created for the tag, see [action.yml](create-release/action.yml). See [test-create-release.yml](.github/workflows/test-create-release.yml) for an example of the action usage.

**Basic:**
```yaml
steps:
  - uses: ottofeller/github-actions/create-release@main
  with:
    initial-version: 0.0.1
    release-branches: main,master,dev
    bump-level: minor
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

**Advanced:**
```yaml
steps:
  - uses: ottofeller/github-actions/create-release@main
  with:
    initial-version: 0.0.1
    release-branches: main,master,dev
    bump-level: minor
    github-token: ${{ secrets.GITHUB_TOKEN }}
    update-main-package_json: true
    update-children-path: |
      packages/first
      packages/second
      packages/third
    update-children-bump-level: |
      minor
      none
      patch
```

## Supported syntax
`initial-version` and `release-branches` inputs are used in `anothrNick/github-tag-action@1.36.0` action and have similar semantics. See [github-tag-action](https://github.com/anothrNick/github-tag-action#readme) docs for details.

* `initial-version` is optional and defaults to `0.0.1`
* `release-branches` is a comma separated list of branches to create a release from, optional and defaults to `main`
* `bump-level` is a level of version to bump which might be one of `major`, `minor`, `patch` and defaults to `patch`
* `github-token` is required and represents your GitHub token which is used for accessing the repo.
* `update-root-package_json` is an optional flag that triggers `package.json` version update in the root folder. Defaults to `false`.
* `update-children-path` is optional. Uses `npm version` command to update version in `package.json` files within specified folders. Provide a space separated list of folder paths for each package.json file to be updated. Applies changes only if `update-children-bump-level` input is provided. Defaults to empty string which effectively passes the step.
* `update-children-bump-level` is optional. A space separated list of version update attribute of `npm-version` command to be used with `update-children-path` input. Make sure the two lists match in length (for paths with missing values of bump-level no update will be performed). To skip updates for a package in the middle of the list pass `none`. Note that a specific version may be provided instead of bump level. Defaults to empty string.

Note that the last three inputs create a new commit. `update-children-path` and `update-children-bump-level` work together but independent of `update-root-package_json`.
- provide only `update-root-package_json: true` to bump the root package version and leave children as is;
- provide `update-children-path` and `update-children-bump-level` without `update-root-package_json` to bump version of only the children packages and leave the root one as is;
- provide the three inputs to update both root and children packages.

# `latest-release-commit-hash`
Obtains the commit hash of the latest release. It does:
- Search through git history and find the latest release (excluding *prerelease* and *draft* types).
- Exit with code 1 if no release was found.
- Output commit hash of the latest release if found.

## Usage
The action searches your repo for the most recent release tag and outputs its commit hash, see [action.yml](latest-release-commit-hash/action.yml). See [test-latest-release-commit-hash.yml](.github/workflows/test-latest-release-commit-hash.yml) for an example of the action usage.

### Use the commit hash within a job:
```yaml
steps:
  - id: commit-hash
    uses: ottofeller/github-actions/latest-release-commit-hash@main
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  - uses: actions/checkout@v3
    with:
      fetch-depth: 0
      ref: ${{ steps.commit-hash.outputs.commit_hash }}
```

### Pass the commit hash to other jobs:
```yaml
jobs:
  set-commit-hash:
    runs-on: ubuntu-latest
    outputs:
      commit_hash: ${{ steps.commit-hash.outputs.commit_hash }}
    steps:
    - id: commit-hash
      uses: ottofeller/github-actions/latest-release-commit-hash@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}

  lint:
    needs: set-commit-hash
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
        ref: ${{ needs.set-commit-hash.outputs.commit_hash }}
    - uses: ottofeller/github-actions/npm-run@main
      with:
        command: npm run lint
```

## Supported syntax
- Inputs:
  * `github-token` is required and represents your GitHub token which is used for accessing the repo.
- Outputs:
  * `commit_hash` is the commit hash of the latest release found in the repo.

# `publish-npm`
Provides the following functionality:
- Check out a repo
- Setting up a distribution of the requested Node.js version
- Install dependencies
- Publishes the package to a specified registry

## Usage
The action sets up node, installs dependencies and runs an `npm publish` command, see [action.yml](publish-npm/action.yml). See [test-publish-npm.yml](.github/workflows/test-publish-npm.yml) for an example of the action usage.

**Basic:**
```yaml
jobs:
  ...

  publish-npm:
    needs: [set-commit-hash, lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: ottofeller/github-actions/publish-npm@main
        with:
          npm-token: ${{ secrets.NPM_TOKEN }}
```

**Advanced:**
```yaml
jobs:
  ...

  publish-npm:
    needs: [set-commit-hash, lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: ottofeller/github-actions/publish-npm@main
        with:
          ref: ${{ needs.set-commit-hash.outputs.commit_hash }}
          node-version: 18
          registry-url: https://npm.pkg.github.com/
          scope: "@worldcoin"
          npm-token: ${{ secrets.NPM_TOKEN }}
          access: restricted
          include-build-step: true
          directory: packages/app
```

## Supported syntax
* The `node-version`, `registry-url`, `scope` inputs are optional and is similar to `setup-node`. The latter is used under the hood and the inputs are fed into setup action. See [setup-node](https://github.com/actions/setup-node#readme) docs for details. By default `node-version` is set to *16*.
* `ref` is optional and represents the git reference used for publishing. Used in checkout action and has the same behavior: when checking out the repository that triggered a workflow, this defaults to the reference or SHA for that event; otherwise, uses the default branch.
* `npm-token` is required and represents your NPM token which is used for access to the registry.
* `access` is optional. It is used in `npm publish` command and tells the registry whether this package should be published as *public* or *restricted*. Defaults to *public*.
* `include-build-step` is an optional flag for running build script (`npm run build` command) before publishing. Defaults to `false`.
* `dry-run` is an optional flag for running the `npm publish` command with `--dry-run` option. Defaults to `false`.
* `directory` is optional and defaults to `./`.

# `generate-db-diagram`
Provides the following functionality:
- Downloads mermerd tarball and extracts the executable
- Runs mermerd either with run config file or with `connectionString` CLI parameter
- Checks for changes and commits the created/updated file to the repo

## Usage
For detailed steps of the action see [action.yml](generate-db-diagram/action.yml). See below for examples of the action usage.

NOTE: You do not need to check out the repo before running the action.

**With [run config](https://github.com/KarnerTh/mermerd#use-a-predefined-run-configuration-eg-for-cicd):**
```yaml
steps:
- uses: ottofeller/github-actions/generate-db-diagram@main
  with:
    run-config: ./yourRunConfig.yaml
```

**With connection string:**
```yaml
steps:
- uses: ottofeller/github-actions/generate-db-diagram@main
  with:
    connection-string: postgresql://user:password@localhost:5432/yourDb
```

**Specify output filename:**
```yaml
steps:
- uses: ottofeller/github-actions/generate-db-diagram@main
  with:
    run-config: ./yourRunConfig.yaml
    output-filename: diagram.mmd
```

**Specify mermerd tarball URL:**
```yaml
steps:
- uses: ottofeller/github-actions/generate-db-diagram@main
  with:
    mermerd-tarball-url: https://github.com/KarnerTh/mermerd/releases/download/v0.3.0/mermerd_0.3.0_linux_amd64.tar.gz
    run-config: ./yourRunConfig.yaml
    output-filename: diagram.mmd
```

## Supported syntax
- `mermerd-tarball-url` is the URL to download mermerd executable from. Defaults to `v0.4.1` asset stored on github.
- `run-config` is the path to a run config file (YAML). If provided connection-string option is ignored.
- `connection-string` is the connection string for the DB.
- `output-filename` is the output file name. Defaults to `result.mmd`. When using run config file make sure the output matches the default one o remember to provide the custom one with this option.

# `generate-graphql-docs`
Runs [`spectaql`](https://github.com/anvilco/spectaql) to get Introspection Query and generates static documentation for a GraphQL schema.

## Usage
For detailed steps of the action see [action.yml](generate-graphql-docs/action.yml). See below for examples of the action usage.

NOTE: You do not need to check out the repo before running the action.

**Only with an introspection-url:**
```yaml
steps:
- uses: ottofeller/github-actions/generate-graphql-docs@main
  with:
    introspection-url: https://server.graphql/introspect
```

**With a custom [config](https://github.com/anvilco/spectaql#yaml-options):**
```yaml
steps:
- uses: ottofeller/github-actions/generate-graphql-docs@main
  with:
    config-path: ./yourConfig.yaml
    introspection-url: https://server.graphql/introspect
    headers: ${{ format('{"x-access-token":"{0}"}', github.ACCESS_TOKEN) }}
    target-dir: docs/graphql
```

## Supported syntax
- `config-path` is the path to a config file (YAML). Defaults to the config stored with the action. The latter sets only basic page info and no data for server querying. If you choose to use a custom config refer to the [spectaql readme](https://github.com/anvilco/spectaql#yaml-options) for a description of the file structure and examples.
- `introspection-url` is the URL for an Introspection Query. Defaults to `http://localhost:3000/`.
- `headers` is an arbitrary set of headers for the Introspection Query as a JSON string. Empty by default.
- `target-dir` is the output folder name. Defaults to `public`.

# License
The scripts and documentation are not licensed. However the use is restricted to Ottofeller projects.
