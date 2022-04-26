# GitHub workflow go-version

Support to upgrade the Go version in a GitHub workflow (for e.g. the `actions/setup-go` action) is built-in. Renovate will automatically detect the `go-version` string.

:warning: Ensure that there are **no quotes** around the version number.

```yml
name: Example for Go version uprades

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  helm-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.head_ref }}

      - uses: actions/setup-go@v3
        with:
          go-version: 1.18.1
```
