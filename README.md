# AsciiDoc

Multi-platform Docker container with utilities to process AsciiDoc files (`asciidoc`, `a2x`, `ascidoctor`...).

[![Dockerfile](https://img.shields.io/badge/GitHub-Dockerfile-blue)](adoc/Dockerfile)
[![Docker Build](https://github.com/leplusorg/docker-adoc/workflows/Docker/badge.svg)](https://github.com/leplusorg/docker-adoc/actions?query=workflow:"Docker")
[![Docker Stars](https://img.shields.io/docker/stars/leplusorg/adoc)](https://hub.docker.com/r/leplusorg/adoc)
[![Docker Pulls](https://img.shields.io/docker/pulls/leplusorg/adoc)](https://hub.docker.com/r/leplusorg/adoc)
[![Docker Version](https://img.shields.io/docker/v/leplusorg/adoc?sort=semver)](https://hub.docker.com/r/leplusorg/adoc)
[![OpenSSF Best Practices](https://bestpractices.coreinfrastructure.org/projects/10081/badge)](https://bestpractices.coreinfrastructure.org/projects/11218)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/leplusorg/docker-adoc/badge)](https://securityscorecards.dev/viewer/?uri=github.com/leplusorg/docker-adoc)

## Example without using the filesystem

Let's say that you want to convert an AsciiDoc file input.adoc in your current working directory to HTML:

**Mac/Linux**

```bash
cat input.adoc | docker run --rm -i --net=none leplusorg/adoc asciidoctor -o - - > output.html
```

**Windows**

```batch
type input.adoc | docker run --rm -i --net=none leplusorg/adoc asciidoctor -o - - > output.html
```

## Example using the filesystem

Same thing, assuming that you want to convert an AsciiDoc file input.adoc in your current working directory to HTML:

**Mac/Linux**

```bash
docker run --rm -t --user="$(id -u):$(id -g)" --net=none -v "$(pwd):/tmp" leplusorg/adoc asciidoctor -o output.html input.adoc
```

**Windows**

In `cmd`:

```batch
docker run --rm -t --net=none -v "%cd%:/tmp" leplusorg/adoc asciidoctor -o output.html input.adoc
```

In PowerShell:

```pwsh
docker run --rm -t --net=none -v "${PWD}:/tmp" leplusorg/adoc asciidoctor -o output.html input.adoc
```

## Use as a GitHub Action

This repository is also a GitHub Action, so you can run any of the tools
bundled in the image (`asciidoctor`, `asciidoc`, `a2x`, `git`, `curl`...)
directly in a workflow, without writing your own `docker run` command.

```yaml
jobs:
  build-docs:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v5
      - name: Convert AsciiDoc to HTML
        uses: leplusorg/docker-adoc@v3.24.1
        with:
          args: asciidoctor -o output.html input.adoc
```

The command runs against your checked-out repository (`$GITHUB_WORKSPACE`),
as the workflow's own user, so any files it produces (such as `output.html`
above) are written back next to your sources and owned by the runner user.

### Choosing the version

The action runs the `leplusorg/adoc` image whose tag matches the ref you pin
the action to: `leplusorg/docker-adoc@v3.24.1` runs image `leplusorg/adoc:3.24.1`.
Because the image version, the GitHub release and the Git tag are always the
same value, pinning the action to a release gives you a reproducible tool set
without any extra configuration. Any other ref (a branch, a commit SHA, or a
local `./` checkout) falls back to the `latest` released image, and you can
always select a specific tag explicitly with the `version` input.

### Inputs

| Input     | Required | Default               | Description                                                                                                    |
| --------- | -------- | --------------------- | -------------------------------------------------------------------------------------------------------------- |
| `args`    | yes      |                       | Command to run inside the container. Interpreted by `/bin/sh`, so pipes, redirects and multiple commands work. |
| `version` | no       | pinned ref / `latest` | Tag of the `leplusorg/adoc` image to run, e.g. `3.24.1`.                                                       |
| `workdir` | no       | `.`                   | Directory, relative to the workspace, in which to run the command.                                             |
| `network` | no       | `default`             | Docker network mode. Set to `none` for a hermetic run with no network access.                                  |

### Security

This action is designed to minimise supply-chain risk for its users:

- It runs a **released, version-tagged** image of `leplusorg/adoc`. A full
  version tag such as `3.24.1` only ever points at the single build that was
  published for that release, so pinning the action to `@v3.24.1` gives you a
  stable, reproducible image.
- The `args` input is passed to the container through an environment variable
  and is **never interpolated into the runner's shell**, so it cannot inject
  commands into the runner. Your command is only ever evaluated inside the
  sandboxed container.
- The container runs unprivileged (`--security-opt no-new-privileges`, mapped
  to the workflow user), and its network can be disabled with `network: none`.

For the strongest guarantees, pin the action to a specific release tag (or its
commit SHA) rather than to `@main`:

```yaml
- uses: leplusorg/docker-adoc@v3.24.1
```

Each `leplusorg/adoc` image is published and signed with [Sigstore](#sigstore)
by this repository's own workflow, and the released image is exactly the build
that was tested (it is not rebuilt on release).

### Requirements

The action uses Docker and therefore requires a Linux runner (for example
`ubuntu-latest`) with Docker available, which is the case for GitHub-hosted
Linux runners.

## Software Bill of Materials (SBOM)

To get the SBOM for the latest image (in SPDX JSON format), use the
following command:

```bash
docker buildx imagetools inspect leplusorg/adoc --format '{{ json (index .SBOM "linux/amd64").SPDX }}'
```

Replace `linux/amd64` by the desired platform (`linux/amd64`, `linux/arm64` etc.).

## Provenance

To get the provenance for the latest image (in JSON format), use the
following command:

```bash
docker buildx imagetools inspect leplusorg/adoc --format '{{ json .Provenance }}'
```

## Sigstore

[Sigstore](https://docs.sigstore.dev) is trying to improve supply
chain security by allowing you to verify the origin of an
artifact. You can verify that the image that you use was actually
produced by this repository. This means that if you verify the
signature of the Docker image, you can trust the integrity of the
whole supply chain from code source, to CI/CD build, to distribution
on Maven Central or wherever you got the image from.

You can use the following command to verify the latest image using its
sigstore signature attestation:

```bash
cosign verify leplusorg/adoc --certificate-identity-regexp 'https://github\.com/leplusorg/docker-adoc/\.github/workflows/.+' --certificate-oidc-issuer 'https://token.actions.githubusercontent.com'
```

The output should look something like this:

```text
Verification for index.docker.io/leplusorg/xml:main --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The code-signing certificate was verified using trusted certificate authority certificates

[{"critical":...
```

For instructions on how to install `cosign`, please read this [documentation](https://docs.sigstore.dev/cosign/system_config/installation/).

## Request new tool

Please use [this link](https://github.com/leplusorg/docker-adoc/issues/new?assignees=thomasleplus&labels=enhancement&template=feature_request.md&title=%5BFEAT%5D) (GitHub account required) to request that a new tool be added to the image. I am always interested in adding new capabilities to these images.

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## Security

Please read [SECURITY.md](SECURITY.md) for details on our security policy and how to report security vulnerabilities.

## Code of Conduct

Please read [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for details on our code of conduct.

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file.
