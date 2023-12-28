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

:warning: This configuration will change with the best practices Anaconda uses. If you want to review changes before applying them, [pin your extension](https://docs.renovatebot.com/config-presets/#github).

[This default configuration](default.json) has a `description` list that will give you an overview over the settings we use by default.

### Anaconda extensions

For specific usage that is not default in renovate, check the [`docs`](docs) folder for the type of file or dependency you want to update.
There’s for example some configuration that allows you to update arbitrary dependencies in Dockerfiles and there might be more to come.

When adding an extension, always use the `description` field to give a short summary of what the configuration does.

### Team/project presets

You can add presets to this repository. Please name them accordingly, e.g. `infrastructure.json` for an infrastructure team specific config. To ensure global defaults are included, please extend the `default.json` in that preset. If you want to add another label, your config could look like this:

```json
{
  "description": "Custom configuration for infrastructure team that will add other labels then the default",
  "labels": [
    "different-label"
  ]
}
```

Please set the `description` field for every configuration block to help others understand what the configuration block does.

To use this preset, you will then set the following in `renovate.json` in your repositories

```json
{
  "extends": ["github>anaconda/renovate-config", "github>anaconda/renovate-config:infrastructure"]
}
```

This config will then get merged with the config in `default.json`. Settings you define in the team specific config will be merged with the default config. In case of conflicting configuration, the config from the last `extends` entry is used.

## Customizing local repo config

You can override [any repository setting](https://docs.renovatebot.com/configuration-options/) by specifying it in your `renovate.json`.

There are also [presets](https://docs.renovatebot.com/presets-default/) you can use. Those combine multiple settings which are often used togehter.

Common settings you might want to change are:

* [`schedule`](https://docs.renovatebot.com/configuration-options/#schedule): Specifies when renovate runs
* [`labels`](https://docs.renovatebot.com/configuration-options/#labels): Add labels to the PR.
