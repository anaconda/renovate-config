# Dockerfile

To upgrade dependencies in a `Dockerfile` that are not automatically recognized, there is a configuration using regex matching.

You need to define your dependency version as an `ENV` var and then use it later on. The env variable name _must_ end in `_VERSION`.

An example:

```Dockerfile
# renovate: datasource=github-tags depName=conda/conda
ENV MINICONDA_VERSION="24.1.2"

RUN curl -L -O https://repo.continuum.io/miniconda/Miniconda3-py38_${MINICONDA_VERSION}-Linux-x86_64.sh \
    && bash Miniconda3-${MINICONDA_PYTHON_VERSION}_${MINICONDA_VERSION}-Linux-x86_64.sh -b -p /opt/conda \
    && rm -rf Miniconda3-${MINICONDA_PYTHON_VERSION}_${MINICONDA_VERSION}-Linux-x86_64.sh
```

The `# renovate: datasource=github-tags depName=conda/conda` tells renovate which

- [`datasource`](https://docs.renovatebot.com/modules/datasource/) to use, check the linked docs for all datasource you can use.
- `depName` to use. This is the name of the dependency and specific for the datasource. For e.g. GitHub, it is `{organization}/{repository}`.
