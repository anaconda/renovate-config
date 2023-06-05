# Go versions in Makefile

Annotate your `Makefile` as follows:

```Makefile
# Directly called by the CI
.PHONY: setup
setup:
# renovate: datasource=github-releases depName=golangci/golangci-lint
	go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.53.1
```

:warning: It is important to **not** indent the comment since Makefiles only support comments at the start of the line.
