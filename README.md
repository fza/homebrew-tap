# fza/homebrew-tap

Homebrew tap for my tools and forks

## Install

```sh
brew install fza/tap/<name>
```

Homebrew resolves `fza/tap` to this repository and taps it on first use. The tap is public, so
installing needs no token.

Upgrades: `brew update && brew upgrade`.

## What is here

| name | what it is | upstream |
|---|---|---|
| [`sdd`](Casks/sdd.rb) | Signal-Dialogue-Decision graph CLI, built from the fork [fza/sdd](https://github.com/fza/sdd) | [networkteam/sdd](https://github.com/networkteam/sdd) |

A fork installs under the same command name as its upstream. Installing both puts two binaries of
the same name on `PATH`, and whichever directory comes first wins, so keep one of them.

Fork versions read `<upstream version>.fza.<n>`, e.g. `0.18.0-alpha.2.fza.22`: the upstream
release the fork builds on, then the fork's own build number.

## Layout

| path | holds |
|---|---|
| `Casks/` | prebuilt binaries, written by each project's release job |
| `Formula/` | anything built from source on the installing machine |
| `CLAUDE.md` | rules for working in this tap, and what a project adds to publish here |

**A binary belongs in `Casks/`.** Homebrew distributes prebuilt artifacts as casks, and GoReleaser
generates them through `homebrew_casks:`; its `brews:` formula output is deprecated. A formula
stays the right shape only for software that has to be compiled on the installing machine.

**A cask is not hand-edited.** The owning project's release job regenerates and commits it, so a
change made here is overwritten by the next release. Fix the project's release configuration
instead.

## Ruby style

`brew style` runs Homebrew's own RuboCop configuration, which requires double-quoted strings. That
governs every `.rb` file here.
