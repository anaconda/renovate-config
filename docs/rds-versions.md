# RDS versions

## Custom Manager

The configuration in this repository enables renovation of RDS engine versions with a custom manager. This manager can be used in any `*.tf` and `*.tfvars` file and uses the following syntax:

```hcl
# renovate:rdsFilter=[{"Name":"engine","Values":["<engine>"]},{"Name":"engine-version","Values":["<engine major version>"]}]
```

The next line must contain `_version: "{version}"`. The prefix in front of `_version` is ignored. The quotes around the version are mandatory.

For more details on the filters, check out the [renovate documentation](https://docs.renovatebot.com/modules/datasource/aws-rds/#usage).

### Available engines

To find the available engines and their major versions, you can use the following query once authenticated to any AWS account (needs `jq` installed):

```shell
aws rds describe-db-engine-versions --output json \
  | jq 'reduce .DBEngineVersions[] as $v ({}; .[$v.Engine][$v.MajorEngineVersion] += [$v.EngineVersion])'
```

This will return a list of all available engines with their major and minor versions to use in the filter expression described above.

## Examples

### tfvars

In a `*.tfvars` file, you can use the following syntax:

```hcl
# renovate:rdsFilter=[{"Name":"engine","Values":["aurora-postgresql"]},{"Name":"engine-version","Values":["14"]}]
rds_engine_version = "14.20"
```

### tf

In a `*.tf` file, with e.g. the `rds-cluster` module, the syntax looks as follows:

```hcl
module "rds_cluster" {
  source  = "cloudposse/rds-cluster/aws"
  version = "2.6.0"

  namespace = "database-name"
  name      = "postgres"
  engine    = "aurora-postgresql"

  # renovate:rdsFilter=[{"Name":"engine","Values":["aurora-postgresql"]},{"Name":"engine-version","Values":["16"]}]
  engine_version             = "16.11"
  auto_minor_version_upgrade = false
}
```

In this example, some required keys have been left out. The important lines are `engine_version` and `auto_minor_version_upgrade`.
