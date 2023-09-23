# Action: Release artifacts

This action downloads artifacts, and creates a release out of them. It also uses the provided `RELEASE_BODY` as text for the release.


Example usage
```yaml
name: Release Golang executable on tag

on:
  push:
    tags:
    - "v[0-9]+.[0-9]+.[0-9]+"

jobs:
  release-prerequisites:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    outputs:
      RELEASE_BODY: ${{ steps.release-prerequisites.outputs.RELEASE_BODY }}
    steps:
      - id: release-prerequisites
        uses: thetillhoff/action-release-prerequisites@v0.1.0

  golang-build:
    needs: release-prerequisites
    runs-on: ubuntu-latest
    permissions:
      contents: write
    strategy:
      matrix:
        os:
          - linux
          - windows
          - darwin
        arch:
          - amd64
    steps:
      - uses: thetillhoff/action-golang-build@v0.1.0
        with:
          OS: "${{ matrix.os }}"
          ARCH: "${{ matrix.arch }}"
          BUILDARGS: -ldflags="-X 'github.com/thetillhoff/webscan/cmd.version=${{ github.ref_name }}'"

  release-artifacts:
    needs:
      - golang-build
      - release-prerequisites
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: thetillhoff/action-release-artifacts@v0.1.0
        with:
          RELEASE_BODY: "${{needs.release-prerequisites.outputs.RELEASE_BODY}}"

```