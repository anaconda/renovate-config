# RDS versions

To update arbitrary RDS engine versions, you can use the following configuration:

```
# renovate:rdsFilter=[{"Name":"engine","Values":["aurora-postgresql"]},{"Name":"engine-version","Values":["14"]}]
rds_engine_version = "14.20"
```

The custom manager for RDS versions detects all `# renovate:rdsFilter={filter}` configurations.

The next line must contain `_version: "{version}"`. The prefix in front of `_version` is ignored. The quotes around the version are mandatory.

For more details on the filters, check out the [renovate documentation](https://docs.renovatebot.com/modules/datasource/aws-rds/#usage).
