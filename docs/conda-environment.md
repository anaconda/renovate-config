# Conda environment.yml

Renovate has a conda datasource already, so we can manually tag dependencies to use it.

To do so, you need to add the line `# renovate datasource=conda depName=${channel}/${dependency}` *directly above* the line with your dependency.

Example:

```yml
name: your-project
channels:
  - defaults
dependencies:
  # renovate datasource=conda depName=main/pytest
  - pytest
  # renovate datasource=conda depName=main/pytest-cov
  - pytest-cov
  # renovate datasource=conda depName=main/coverage
  - coverage
  # renovate datasource=conda depName=main/yapf
  - yapf==0.31.0
```
