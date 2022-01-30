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

# Default Preset
Because JSON doesn't support comments, this readme walks through all the options.

## Extends

Config Block:
```json
{
  "extends": [
    "config:base",
    "group:recommended"
  ]
}
```

[Ref](https://docs.renovatebot.com/configuration-options/#extends)

`"config:base"` - [Use a set of defaults defined by Renovate](https://docs.renovatebot.com/presets-config/#configbase)

`"group:recommended"` - [Use a default group that will group monorepos and known packages that must be upgrade together, like apollo server together](https://docs.renovatebot.com/presets-config/#configbase)


## Labels

`"labels": ["dependencies"],` - [Apply the label `dependencies` to all prs by renovate](https://docs.renovatebot.com/configuration-options/#labels)

## Pin Digests

`"pinDigests": true,` - [Pins docker images to their digest version so that if an upstream image is updated we get the pinned one and renovate will update the pinned version](https://docs.renovatebot.com/configuration-options/#labels)

## Platform Automerge

`"platformAutomerge": true,` - [Use github's automerge functionality if it is enabled, instead of Renovates auto-merging](https://docs.renovatebot.com/configuration-options/#platformautomerge)

## Supress Notifications

`"suppressNotifications": ["prIgnoreNotification"],` - [Use this field to suppress various types of warnings and other notifications from Renovate](https://docs.renovatebot.com/configuration-options/#suppressnotifications)


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
  "automerge": true
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