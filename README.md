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

The action sets up node, installs npm packages and runs a command supplied in inputs, see [action.yml](npm-run/action.yml). In order to use the action in your workflow copy the [action.yml](npm-run/action.yml) file into your project and reference it in the workflow file. See [test.yml](.github/workflows/test.yml) for an example of the action usage.

**Basic:**
```yaml
steps:
- uses: @ottofeller/github-actions/npm-run@main
  with:
    command: npm run test:all -- --verbose
```

**Advanced:**
```yaml
steps:
- uses: @ottofeller/github-actions/npm-run@main
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

# License
The scripts and documentation are not licensed. However the use is restricted to Ottofeller projects.
