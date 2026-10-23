# CLAUDE.md

This repository is the public Homebrew tap `fza/tap`: casks and formulae for the owner's own
projects and forks. [README.md](README.md) is the user-facing description; this file holds the rules
for working in it.

## Rules

- **The repository is public.** Nothing from a private project, a private tap or a client lands
  here: no hosts, group or project names, registry paths, token names or internal specifics.
- **A cask is not hand-edited.** The owning project's release job regenerates and commits it, so a
  change made here is overwritten by the next release. Fix the project's release configuration.
- **A binary belongs in `Casks/`.** GoReleaser generates casks through `homebrew_casks:`; its
  `brews:` formula output is deprecated. `Formula/` is only for software compiled on the installing
  machine.
- **Ruby follows `brew style`**, Homebrew's RuboCop configuration, which requires double-quoted
  strings in every `.rb` file.
- **A new tool gets a row in the README's table** when its first cask lands: name, what it is, and
  its upstream for a fork.

## Publishing a project into this tap

### The credential

**One fine-grained personal access token per publishing project, scoped to this repository.**

1. <https://github.com/settings/personal-access-tokens/new>: repository access **Only select
   repositories** → `fza/homebrew-tap`; permission **Contents: Read and write**, nothing else. Set an
   expiry the owner will notice.
2. Publishing project: `gh secret set HOMEBREW_TAP_TOKEN -R fza/<project>`, value the token.

The workflow's own `GITHUB_TOKEN` cannot write here: it is scoped to the repository running the
workflow.

### The release configuration

`.goreleaser.yaml`, alongside the project's own `builds:` and `archives:`:

```yaml
homebrew_casks:
  - name: <project>
    repository:
      owner: fza
      name: homebrew-tap
      branch: main
      token: "{{ .Env.HOMEBREW_TAP_TOKEN }}"
    homepage: https://github.com/fza/<project>
    description: <one-line description>
    hooks:
      post:
        install: |
          if OS.mac?
            system_command "/usr/bin/xattr", args: ["-dr", "com.apple.quarantine", "#{staged_path}/<binary>"]
          end
```

**The quarantine hook is what makes an unsigned binary runnable.** Without it macOS refuses the
downloaded binary until the attribute is cleared by hand.

**A fork keeps its upstream's release configuration where it can.** Take the owner from the
environment GitHub Actions sets, `owner: "{{ .Env.GITHUB_REPOSITORY_OWNER }}"`, and the same file
releases into the upstream's tap there and into this one here, so pull requests back to the
upstream carry no fork-only edits.

**Prereleases need `skip_upload` set explicitly.** `skip_upload: auto` skips the cask for
prerelease tags, and a fork tag such as `v0.18.0-alpha.2.fza.22` is one. A fork whose releases are
all prereleases renders `skip_upload` from an Actions variable,
`"{{ with .Env.HOMEBREW_CASK_SKIP_UPLOAD }}{{ . }}{{ else }}auto{{ end }}"`, set to `false` on the
fork.

### The release job

```yaml
on:
  push:
    tags: ['v*']

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0
      - uses: actions/setup-go@v6
        with:
          go-version-file: go.mod
      - uses: goreleaser/goreleaser-action@v7
        with:
          version: "~> v2"
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          HOMEBREW_TAP_TOKEN: ${{ secrets.HOMEBREW_TAP_TOKEN }}
          HOMEBREW_CASK_SKIP_UPLOAD: ${{ vars.HOMEBREW_CASK_SKIP_UPLOAD }}
```

**A fork's tag history decides its changelog.** GoReleaser lists commits since the previous tag in
the fork. Push the upstream tag the fork builds on before the first fork release, or the changelog
reaches back to the first commit.
