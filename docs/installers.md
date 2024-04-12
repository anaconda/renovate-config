# Miniconda and Anaconda Distribution Installers

## Updating dependencies

Projects that depend on the Miniconda or Anaconda Distribution installers can be updated using `renovate`.

To use the data sources for the installers

```json
{
  "extends": ["github>anaconda/renovate-config:installers"]
}
```

A regex manager should be used to configure the update (see the [Examples section](#examples) for more).
The following additional capture groups are needed to find the latest installer version:

- `datasource`: must be `custom.miniconda_installer` or `custom.anaconda_installer`
- `depName`: the platform + extension, e.g., `Linux-x86_64.sh`, `Windows-x86_64.exe`, or `MacOSX-arm64.pkg`
- `currentDigest` (optional): the sha256 checksum of the installer

A single versioning template can be used for both installers:

```json
{
  "customManagers": [
    {
      "customType": "regex",
      "matchString": ["see Examples section for example match strings"],
      "versioningTemplate": "regex:^(py\\d+_)?(?<major>\\d+)\\.(?<minor>\\d+)(\\.(?<patch>\\d+))?(\\-(?<build>\\d+))?$"
    }
  ]
}
```

If multiple Miniconda installers are in the repository, updates will all be bundled into one pull request,
even if they are different architectures and file extensions.
Anaconda Distribution and Miniconda, however, will appear in seperate PRs.

> [!NOTE]
> Miniconda installers are published for multiple Python versions.
> This `renovate` configuration will always choose the "latest" installer,
> i.e., the installers found in the [Miniconda documentation](https://docs.anaconda.com/free/miniconda/).

## Examples

### Simple updates

The example below shows a snippet of an example `Dockerfile`.
It only sets the Miniconda version, which must include the Python version, and the installer suffix is given in the comment line.

```Dockerfile
# renovate: datasource=custom.miniconda_installer depName=Linux-x86_64.sh
ARG INSTALLER_VERSION=py312_24.1.0-0
```

The corresponding regex manager looks like this:

```json
{
  "customManagers": [
    {
      "customType": "regex",
      "matchString": [
        "# renovate:\\s*datasource=(?<datasource>[\\w\\.]+?) depName=(?<depName>[\\w\\-\\.]+?)\\sARG INSTALLER_VERSION=(?<currentValue>(py\\d+_)?[\\d\\.]+\\-\\d+?)\\s"
      ],
      "versioningTemplate": "regex:^(py\\d+_)?(?<major>\\d+)\\.(?<minor>\\d+)(\\.(?<patch>\\d+))?(\\-(?<build>\\d+))?$"
    }
  ]
}
```

The `matchString` and `versioningTemplate` are designed to capture both Miniconda and Anaconda Distribution installers.
The `matchString` is pretty-printed below.

```
# renovate:\s*datasource=(?<datasource>[\w\.]+?) depName=(?<depName>[\w\-\.]+?)\s
ARG INSTALLER_VERSION=(?<currentValue>(py\d+_)?[\d\.]+\-\d+?)\s
```

### Update with checksums

The example below shows a snippet of a `bash` script.
It contains the full URL for the Anaconda Distribution installer and its checksum.
`depName` is not in the comment line and will be extracted from the `INSTALLER_URL` directly.

```bash
# renovate datasource=custom.anaconda_installer
INSTALLER_URL="https://repo.anaconda.com/archive/Anaconda3-2023.09-0-Linux-aarch64.sh"
SHA256SUM="69ee26361c1ec974199bce5c0369e3e9a71541de7979d2b9cfa4af556d1ae0ea"
```

The corresponding regex manager looks like this:

```json
{
  "customManagers": [
    {
      "customType": "regex",
      "matchString": [
        "# renovate:\\s*datasource=(?<datasource>[\\w\\.]+?)\\sINSTALLER_URL=[\"']https:\\/\\/repo\\.anaconda\\.com\\/\\w+\\/\\w+-(?<currentValue>(py\\d+_)?[\\d\\.]+\\-\\d+?)-(?<depName>[\\w\\-\\.]+?)[\"']\\sSHA256SUM\\w*=[\"'](?<currentDigest>[a-f\\d]+?)[\"']\\s"
      ],
      "versioningTemplate": "regex:^(py\\d+_)?(?<major>\\d+)\\.(?<minor>\\d+)(\\.(?<patch>\\d+))?(\\-(?<build>\\d+))?$"
    }
  ]
}
```

The `matchString` and `versioningTemplate` are designed to capture both Miniconda and Anaconda Distribution installers.
The `matchString` is pretty-printed below.

```
# renovate:\s*datasource=(?<datasource>[\w\.]+?)\s
INSTALLER_URL=["']https:\/\/repo\.anaconda\.com\/\w+\/\w+-(?<currentValue>(py\d+_)?[\d\.]+\-\d+?)-(?<depName>[\w\-\.]+?)["']\s
SHA256SUM=["'](?<currentDigest>[a-f\d]+?)["']\s
```

## Developers: understanding the transform templates

Since the installers are updated in a format that is not covered using pre-existing `renovate` managers,
a [custom datasource](https://docs.renovatebot.com/modules/datasource/custom/) had to be created.
It can be found in the [installer.json](../installers.json) file in this repository.
It uses a transform template to convert the `.files.json` file into a format `renovate` can process.
Unfortunately, `renovate` expects a an array of objects whereas the `.files.json` file is a nested object.
The file needs to be processed using [JSONata](https://jsonata.org/) and stored as `transformTemplates`.

Pretty-printed, the template is the following for Miniconda:

```
{
  "releases": [
    $map($sift($, function($v, $k) {
      $not($k.$contains("latest"))
      and $v.sha256 = $lookup($, "Miniconda3-latest-{{ packageName }}").sha256
    }).$keys(), function($v) {
      {
        "digest": $lookup($, $v).sha256, "version": $contains($v.$split("-")[2], /^[0-9]+$/) ? $v.$split("-")[1] & "-" & $v.$split("-")[2] : $v.$split("-")[1],
        "changeLogUrl": "https://docs.anaconda.com/free/miniconda/miniconda-release-notes/#miniconda-" & ($contains( $v.$split("-")[2], /^[0-9]+$/) ? $v.$split("-")[1].$split("_")[1] & "-" & $v.$split("-")[2] : $v.$split("-")[1].$split("_")[1]).$replace(".", "-"),

        "sourceUrl": "https://repo.anaconda.com/miniconda/" & $v, releaseTimestamp": $fromMillis($lookup($, $v).mtime * 1000, "[Y0001]-[M01]-[D01]T[H01]:[m01]", "-6")
      }
    }
  )],
  "sourceUrl": "https://repo.anaconda.com/miniconda/",
  "homepage": "https://anaconda.com"
}
```

and the following for Anaconda Distribution:

```
{
  "releases": $map(
    $sift($, function($v, $k){
      $k.$contains("{{ packageName }}")
    }).$keys(), function($v) {
      {
        "digest": $lookup($, $v).sha256,
        "version": $contains($v.$split("-")[2], /^[0-9]+$/) ? $v.$split("-")[1] & "-" & $v.$split("-")[2] : $v.$split("-")[1],
        "changeLogUrl": "https://docs.anaconda.com/free/anaconda/release-notes/#anaconda-" & ($contains( $v.$split("-")[2], /^[0-9]+$/) ? $v.$split("-")[1] & "-" & $v.$split("-")[2] : $v.$split("-")[1]).$replace(".", "-"),
        "sourceUrl": "https://repo.anaconda.com/archive/" & $v,
        "releaseTimestamp": $fromMillis($lookup($, $v).mtime * 1000, "[Y0001]-[M01]-[D01]T[H01]:[m01]", "-6")
      }
    }
  ),
  "sourceUrl": "https://repo.anaconda.com/archive/",
  "homepage": "https://anaconda.com"
}
```

The `$map` function creates an array of releases containing data such as the version, the digest (the sha256 sum), changelog URL, etc.
The `$sift` function is used to filter the appropriate installer file names that are passed into `$map`.

All `$sift` functions make use of the `{{ packageName }}` template value.
`packageName` corresponds to `depName` in the regular expressions of the `matchStrings` (`depName` cannot be used in the JSONata because renovate only replaces [a few variables](https://docs.renovatebot.com/modules/datasource/custom/#usage)).
The value of `depName` and `packageName` is the installler suffix (e.g., `Linux-x86_64.sh`)
to capture and group installer types.
Overloading `depName` and `packageName` this way has the drawback that by default `renovate` would create a pull request for each architecture.
To override this behavior, the `packageRules` bundle all Miniconda/Anaconda Distribution updates into a single PR each.

Since Miniconda has multiple installer variants and also a set of "latest" installers, the logic is more complex.
The `$sift` condition only grabs the installer that is identical to the "latest" installer, i.e., `releases` will only contain the latest installer.
JSONata flattens arrays with only one element, which is why `[]` is wrapped around the `$map` function for Miniconda, but not for Anaconda Distribution.

The version string uses a ternary operator to account for the build number, which not all installers have.
The same pattern is used in `changeLogUrl`, but `.` needs to be replaced with `-` to get the appropriate anchor. For Miniconda, the extra `$split` filters out the `py*_` prefix in the version.

The timestamp is given in Central Standard Time, which is used for release dates in documentation.
The format specifier follows the [XPATH F&O 3.1](https://www.w3.org/TR/xpath-functions-31/#date-picture-string) specification, as required by JSONata.

The transform template currently produces more data than needed because it is expected to become global once it is fully tested.
