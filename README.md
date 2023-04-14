# Renovate Config

This repository holds a re-usable configuration for [Renovate](https://www.whitesourcesoftware.com/free-developer-tools/renovate/) across Pocket.

# Usage 

Set up a `renovate.json` in your repo's root directory with the following contents:

```json
{
  "extends": [
    "github>Pocket/renovate-config"
  ]
}
```

This will use the default.json that is defined in this repository. [In the future we may have other json files representing different presets.](https://docs.renovatebot.com/config-presets/#github)

You also need to ensure the following:
* [renovate](https://github.com/apps/renovate) is allowed on your repository
* [renovate-approve](https://github.com/apps/renovate-approve) is allowed on your repository
* All dependabot and auto-merge files are removed from your repository. While, not necessary, they will conflict with each other.
* Allow auto-merge is enabled in your GitHub repository settings. While not necessary this will allow Renovate to use Github's default auto-merging strategy.
* If you want support for Vulnerability Alerts, ensure that the `Dependency graph` and `Dependabot alerts` options are enabled in `Code security and analysis` for your repo. Checks for vulnerability alerts are based off dependabot alerts, which reference the [GitHub Advisory Database](https://github.com/advisories) for packages with reported vulnerabilities. Checks for dependencies with active advisories:
  * Are performed freely at any time.
  * Do not wait for release stability.
  * Open PRs immediately, ignoring limits to the number of active PRs.
  * Do not attempt to group dependencies.
  * Are marked up differently from other PRs in order to enable driving alerts.
    * Commit messages are appended with the string `[SECURITY]`.
    * Branch names end with `*-vulnerability`


# Testing

Run the following to validate your renovate json

`cp default.json5 renovate.json5 && npx --package renovate -c 'renovate-config-validator'`

Note: We need to cp default.json5 because the renovate-validator only looks for files named renovate.

# Default Preset
Because JSON doesn't support comments, this readme walks through all the options.

## Extends

Config Block:
```json
{
  "extends": [
    "config:base",
    "group:recommended",
    ":semanticCommits"
  ]
}
```

[Ref](https://docs.renovatebot.com/configuration-options/#extends)

`"config:base"` - [Use a set of defaults defined by Renovate](https://docs.renovatebot.com/presets-config/#configbase)

`"group:recommended"` - [Use a default group that will group monorepos and known packages that must be upgrade together, like apollo server together](https://docs.renovatebot.com/presets-config/#configbase)

`":semanticCommits"` - [Use a default group that will enable semantic commits for production dependencies](https://docs.renovatebot.com/semantic-commits/#manually-enabling-or-disabling-semantic-commits)

`"workarounds:typesNodeVersioning"` - [Use node versioning for @types/node](https://docs.renovatebot.com/presets-workarounds/#workaroundstypesnodeversioning)

`"preview:dockerCompose"` - [Use docker-compose preview updating](https://docs.renovatebot.com/presets-preview/#previewdockercompose)

## Labels

`"labels": ["dependencies"],` - [Apply the label `dependencies` to all prs by renovate](https://docs.renovatebot.com/configuration-options/#labels)

## Pin Digests

`"pinDigests": true,` - [Pins docker images to their digest version so that if an upstream image is updated we get the pinned one and renovate will update the pinned version](https://docs.renovatebot.com/configuration-options/#labels)

## Platform Automerge

`"platformAutomerge": true,` - [Use github's automerge functionality if it is enabled, instead of Renovates auto-merging](https://docs.renovatebot.com/configuration-options/#platformautomerge)

## Suppress Notifications

`"suppressNotifications": ["prIgnoreNotification","onboardingClose"],` - [Use this field to suppress various types of warnings and other notifications from Renovate](https://docs.renovatebot.com/configuration-options/#suppressnotifications)

## Support Policy 

`"supportPolicy": "lts",` - Tells Renovate to prefer LTS releases of packages

## Package Rules

https://docs.renovatebot.com/configuration-options/#packagerules

The following are a set of custom package rules that we have enabled.

### Docker

For each docker update, add the label `docker-update`

```json
{
  "matchDatasources": ["docker"],
  "labels": ["docker-update"]
}
```

### NPM Stability

For all npm packages, make sure they have been around for at least 3 days. NPM packages can be pulled for up to 3 days after publishing.

```json
    {
  "matchDatasources": ["npm"],
  "stabilityDays": 3
}
```


### Automerge Versions

Auto merge patch, pinned, and digest updates.

Patch are patch updates to all types of packages, docker images.
Pinned are updates to the repo to pin a package to it's currently installed version.
Digest updates are updates to docker upstream images.

```json    
{
  "matchUpdateTypes": ["patch", "pin", "digest"],
  "dependencyDashboardApproval": false,
  "automerge": true
}
```

### Dependency Dashboard Minor/Major

This makes it so that Minor & Major versions of any package require a developer to click a checkbox on the issues to create a PR. 

This helps reduce GitHub Notification noise for developers.

https://docs.renovatebot.com/configuration-options/#dependencydashboardapproval

```json    
{
  "matchUpdateTypes": ["minor", "major"],
  "dependencyDashboardApproval": true
}
```


### Automerge Dev

Auto merge any type of dev dependencies. These are almost always safe.
```json    
{
  "matchDepTypes": ["devDependencies"],
  "automerge": true
}
```

### Group CDKTF

Groups all the CDKTF package updates together. This is because when CDKTF provider packages are updated they are almost always updated together and require the latest version of the CDKTF package. This also marks the commits as a fix type which will trigger a release.

```json
{
  "matchUpdateTypes": ["minor", "major", "patch", "pin"],
  "separateMajorMinor": false,
  "semanticCommitType": "fix",
  "matchPackagePrefixes": [
    "cdktf",
    "cdktf-cli",
    "constructs",
    "@cdktf/"
  ],
  "groupName": "cdktf"
}
```

### Node Update

Match all the node packages in our repos at the same time and use the Node versioning scheme that is defined. The package names will match in any of the package managers (docker, npm, node).

```json
{
  "matchDatasources": ["docker","node","npm"],
  "semanticCommitType": "ci",
  "matchPackageNames": ["node", "circleci/node", "cimg/node"],
  "versioning": "node"
}
```

