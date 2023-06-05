# tflint plugins

Annotate your `.tflint.hcl` file as follows:


```hcl
plugin "aws" {
    enabled = true
    # renovate datasource=github-releases depName=terraform-linters/tflint-ruleset-aws
    version = "0.20.0"
    source  = "github.com/terraform-linters/tflint-ruleset-aws"
}
```
