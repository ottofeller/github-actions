# GitHub actions
A collection of GitHub actions for use within `ottofeller` projects.

# Contents
The following actions are included:

- npm-run
- create-release-gh-action
- publish-npm-gh-action
# `npm-run`

Provides the following functionality:

- Check out a repo
- Setting up a distribution of the requested Node.js version
- Install an app
- Run an npm script
- The dir to the app may be specified

## Usage

The action sets up node, installs npm packages and runs a command supplied in inputs, see [action.yml](npm-run/action.yml). In order to use the action in your workflow copy the [action.yml](npm-run/action.yml) file into your project and reference it in the workflow file. See [test.yml](.github/workflows/test-npm-run.yml) for an example of the action usage.

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

The action searches your repo for the most recent tag and creates a new one with patch update, then a draft release is created for the tag, see [action.yml](create-release/action.yml). See [test.yml](.github/workflows/test-create-release.yml) for an example of the action usage.

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

# `publish-npm`

Provides the following functionality:

- Check out a repo
- Setting up a distribution of the requested Node.js version
- Install dependencies
- Publishes the package to a specified registry

# Usage

The action sets up node, installs dependencies and runs an `npm publish` command, see [action.yml](publish-npm/action.yml). See [test.yml](.github/workflows/test-publish-npm.yml) for an example of the action usage.

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
```

## Supported syntax

* The `node-version`, `registry-url`, `scope` inputs are optional and is similar to `setup-node`. The latter is used under the hood and the inputs are fed into setup action. See [setup-node](https://github.com/actions/setup-node#readme) docs for details. By default `node-version` is set to *16*.
* `ref` is optional and represents the git reference used for publishing. Used in checkout action and has the same behavior: when checking out the repository that triggered a workflow, this defaults to the reference or SHA for that event; otherwise, uses the default branch.
* `npm-token` is required and represents your NPM token which is used for access to the registry.
* `access` is optional. It is used in `npm publish` command and tells the registry whether this package should be published as *public* or *restricted*. Defaults to *public*.
* `include-build-step` is an optional flag for running build script (`npm run build` command) before publishing. Defaults to `false`.
* `dry-run` is an optional flag for running the `npm publish` command with `--dry-run` option. Defaults to `false`.

# License
The scripts and documentation are not licensed. However the use is restricted to Ottofeller projects.
