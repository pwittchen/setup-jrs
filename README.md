# setup-jrs

Set up [jrs](https://getjrs.dev), a fast JVM build tool written in Rust, in your
GitHub Actions workflows.

The action downloads a prebuilt jrs from the
[jrs releases](https://github.com/pwittchen/jrs/releases), checks it against the
release's `SHA256SUMS`, and puts it on `PATH`. It runs on Linux, macOS and
Windows runners, x86_64 and arm64. It can also cache jrs's dependency and build
cache between runs.

## Usage

jrs needs a JDK 17 or newer, so set one up first:

```yaml
steps:
  - uses: actions/checkout@v5
  - uses: actions/setup-java@v6
    with:
      distribution: temurin
      java-version: '21'
  - uses: pwittchen/setup-jrs@v1
  - run: jrs test
```

### A specific version

```yaml
- uses: pwittchen/setup-jrs@v1
  with:
    version: '0.11.1'
```

### The version the project asks for

`version: manifest` installs the newest release that `project.jrs-version` in
`jrs.toml` allows: `"0.10"` is the newest 0.10.x, `"0.10.1"` that release
exactly, since a 0.x minor may break; from 1.0, the newest release of the same
major version at or above it.

```yaml
- uses: pwittchen/setup-jrs@v1
  with:
    version: manifest
    manifest: services/api/jrs.toml   # default: jrs.toml
```

### Caching dependencies

`cache: true` points `JRS_CACHE_DIR` at a fixed directory and saves it with
[`actions/cache`](https://github.com/actions/cache), keyed on the runner's OS
and architecture and the hash of every `jrs.lock` in the repository. A changed
lockfile restores the closest earlier cache, so only the new jars are
downloaded. The build cache lives in the same directory, so unchanged modules
are restored instead of compiled.

```yaml
- uses: pwittchen/setup-jrs@v1
  with:
    cache: true
```

A `JRS_CACHE_DIR` the workflow sets itself is kept and cached instead.

### A build matrix

```yaml
jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        java: ['17', '21']
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-java@v6
        with:
          distribution: temurin
          java-version: ${{ matrix.java }}
      - uses: pwittchen/setup-jrs@v1
        with:
          cache: true
      - run: jrs build
      - run: jrs test
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `version` | `latest` | The release to install, such as `0.11.1`, `latest`, or `manifest`. |
| `manifest` | `jrs.toml` | The manifest `version: manifest` reads, relative to the working directory. |
| `cache` | `false` | Whether to cache the jrs dependency and build cache between runs. |
| `cache-dependency-path` | `**/jrs.lock` | The files whose hash keys the cache, as a glob relative to the workspace. |

## Outputs

| Output | Description |
| --- | --- |
| `version` | The version of jrs installed, such as `0.11.1`. |
| `path` | The directory jrs was installed into. |
| `cache-hit` | Whether the cache was restored for the exact key. |

## How it works

`latest` is resolved to a concrete release first, and each release is
installed into the runner's tool cache under its version, so a self-hosted
runner that already has it does not download it again. The archive is always
checked against the release's `SHA256SUMS` before it is unpacked.

## License

[Apache 2.0](LICENSE)
