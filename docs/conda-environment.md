# Conda environment.yml

Renovate has a conda datasource already, so we can manually tag dependencies to use it.

To do so, you need to add the line `# renovate: datasource=conda depName=${channel}/${dependency}` *directly above* the line with your dependency.

You can also capture pypi dependencies by placing the line `# renovate: datasource=pypi` *directly above* the line with your dependency

Note: You must specify the exact current version of your conda dependency before Renovate will recognize available upgrades

:information_source: This manager also supports the **deprecated notations** `# renovate datasource=conda` and `# renovate datasource=pypi` **without a colon**. Please add a colon wherever you find these to get be compatible with renovate upstream.

## Example:

```yml
name: your-project
channels:
  - defaults
dependencies:
  # renovate: datasource=conda depName=main/pytest
  - pytest
  # renovate: datasource=conda depName=main/pytest-cov
  - pytest-cov
  # renovate: datasource=conda depName=main/coverage
  - coverage
  # renovate: datasource=conda depName=main/yapf
  - yapf=0.31.0
  - pip
  - pip:
    # renovate: datasource=pypi
    - uvicorn==0.17.6
```
