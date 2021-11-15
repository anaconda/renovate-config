# renovate-config

These are Anaconda's global default configurations for renovate.

If you work for Anaconda and have any questions, please ask the infrastructure team. In all other cases, feel free to open an issue.

## Usage

Check [docs.renovatebot.com/config-presets](https://docs.renovatebot.com/config-presets/) for details on config presets.

Renovate will open a „Dependency Dashboard“ issue in each repository, tracking outstanding or rejected updates. This issue will have three sections:

* **Awaiting Schedule**: renovate has detected an update, but has not yet run during the scheduled time to open Pull Requests (see the `schedule` configuration)
* **Open**: There is an open Pull Request for those updates
* **Ignored or Blocked**: Those updates are blocked because the existing Pull Request was closed.

### Simple

To use the `default.json` configuration, put the following content in your local repository's `renovate.json` file:

```json
{
  "extends": ["github>anaconda/renovate-config"]
}
```

[This default configuration](default.json) does the following:

* Uses the recommended base config by renovate (`extends[config:base]`)
* Disables rate limiting: number of Open PRs is unlimited, number of PRs opened per hour is unlimited: (`extends[:disableRateLimiting]`)
* Pins GitHub actions by digest, but add the tag it’s pinned to as comment (`extends[helpers:pinGitHubActionDigests]`)
* Enables the [pre-commit manager](https://docs.renovatebot.com/modules/manager/pre-commit/) that is in beta (`pre-commit` section)
* Adds `renovate` as label to all Pull Requests (`labels` section)
* Automatically sets Pull Request reviewers from the CODEOWNERS file if it exists (`reviewersFromCodeOwners`)
* Rebases PRs that are behind the target branch to ensure tests run against up-to-date config. (`extends[:rebaseStalePrs]`)
* Sets the timezone to UTC (`timezone`)
* Runs before 6am UTC each Monday (`schedule`). This timeframe makes renvoate run outside of business hours for both our US and European teams.

### Anaconda extensions

For specific usage that is not default in renovate, check the [`docs`](docs) folder for the type of file or dependency you want to update.
There’s for example some configuration that allows you to update arbitrary dependencies in Dockerfiles and there might be more to come.

### Team/project presets

You can add presets to this repository. Please name them accordingly, e.g. `infrastructure.json` for an infrastructure team specific config. To ensure global defaults are included, please extend the `default.json` in that preset. If you want to add another label, your config could look like this:

```json
{
  "extends": ["github>anaconda/renovate-config"],
  "labels": [
    "additional-label"
  ]
}
```

This config will then get merged with the config in `default.json. Settings you define in the team specific config will override the default config.

To use this preset, you will then set the following in `renovate.json` in your repositories

```json
{
  "extends": ["github>anaconda/renovate-config:infrastructure"]
}
```

## Customizing local repo config

You can override [any repository setting](https://docs.renovatebot.com/configuration-options/) by specifying it in your `renovate.json`.

There are also [presets](https://docs.renovatebot.com/presets-default/) you can use. Those combine multiple settings which are often used togehter.

Common settings you might want to change are:

* [`schedule`](https://docs.renovatebot.com/configuration-options/#schedule): Specifies when renovate runs
* [`labels`](https://docs.renovatebot.com/configuration-options/#labels): Add labels to the PR.
