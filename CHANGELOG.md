# Changelog

Release tags are `vX.Y.Z` = the `version` in `package.json`. A project pins one in its manifest
(`…com.unpseudo.autoplay.git#vX.Y.Z`); updating the package there = bumping that tag.

## [0.2.0] - 2026-10-11

First release from its own repository (history kept from the game project it grew in).

### Added
- `{tools}` placeholder in a scenario's analyzer command: the package's `Tools~` folder, wherever the package lives
  (embedded in `Packages/` or read-only in `Library/PackageCache/`).
- `Skill~/autoplay/SKILL.md`: a game-agnostic Claude Code usage skill to copy into a project.
- MIT license, changelog, install / update / local-override instructions in the README.

### Changed
- The tools never write Python bytecode next to themselves (`Library/PackageCache` is read-only).
- Shell and Python tools are checked out with LF line endings on every platform.

## [0.1.0] - 2026-10-04

Embedded package of Corruption du Portail: runner, journal, captures + state export, animation recorder, watchdog,
command line, dev-build bootstrap, window guard, virtual mouse / keyboard, and the `Tools~` launchers, scenario runner,
campaign, desync detector, contact sheets and videos.
