---
name: autoplay
description: Play this Unity game automatically with bots (host + simulated seats or real network clients, the game's real code paths) to check a precise goal, with screenshots, recordings and exported state at the right moments, then answer with evidence; and extend the autoplay framework (new lever, event, check, scenario) when a goal is not covered yet. Use when the user asks to test / check / verify / playtest / reproduce something in-game automatically ("check that every power…", "lance une partie auto", "fais jouer des bots", "autoplay", "teste en jeu", "teste en réseau", "simulate a game"), or to add / improve an autoplay feature ("ajoute à l'autoplay", "autoplay doesn't cover…", "new scenario", "extend autoplay"). PREFER this over manual play mode or ad-hoc scripts for any in-game behaviour or visual check.
---

<!-- Template shipped with the com.unpseudo.autoplay package (Skill~/autoplay/SKILL.md). Copy it into the project's
     .claude/skills/autoplay/, fill in "This project", and keep game-specific knowledge there, not in the package. -->

# Autoplay — goal → bots play → evidence → answer

## This project

Fill in once, when the game is plugged in:

- Game adapter (`IAutoplayGame` + bot driver): `<path>`.
- Project wrapper around the package CLI: `<path to unityctl wrapper>` (sets `AUTOPLAY_PROJECT`, guarded files, game
  args, and resolves the package's `Tools~` folder: `Packages/com.unpseudo.autoplay/Tools~` if a local clone overrides
  the package, else the newest `Library/PackageCache/com.unpseudo.autoplay@*/Tools~`).
- Reference of every lever, event, capture, check and scenario: `<path to REFERENCE.md>`. Look there first.
- Scenarios folder: `<path>`. Table of the scenarios and their last result: `<path or below>`.
- Game-specific rules (client vs server state, forced compositions, time levers…): `<path>`.

Package docs (in the package folder, see "Where the package lives"): `README.md` (layout, outputs, hard-won lessons,
read once), `EXTENDING.md` (adding to the framework: where code goes, rules, definition of done), `CHANGELOG.md`.

## Where the package lives

The package is its own git repository (`UnPseudoTotalementNormal/com.unpseudo.autoplay`), pinned by tag in
`Packages/manifest.json`. `Library/PackageCache/` is read-only: **never edit it** (lost on the next resolve). A change
that works for any game goes into the package repository (clone it into the project's `Packages/com.unpseudo.autoplay/`
to iterate, release a tag, bump the manifest, delete the clone). Anything that knows the game goes into the adapter.

## 1. Turn the request into a goal you can check

Write down, before running anything: **what must be observed, when, on whose screen, and what counts as pass/fail**.
Map it to what the run produces:
- game logic / flow / errors → `report.json` (outcome, phase trace, counters, errors) + `events.ndjson`
- exact values at a moment → the capture's state file (`NNN-label.json`, game state under `"game"`, probes)
- visuals → the capture images at that moment (bursts give several timings of one moment)

Need another moment or value? Add a `capture.Request(...)` / `RequestBurst(...)` or an `AddProbe(...)` in the driver.

**Force the situation from the start.** A random game does not guarantee the case under test: before launching, ask
"what must happen for this run to prove something, and what guarantees it?" If nothing does, add the lever first, then
run. Afterwards check in the journal that the situation really happened — otherwise report "not covered", never "OK".

## 2. Environment

- Prefer your own headless editor (`Unity.exe -batchmode -automated -projectPath … -logFile …`): modal dialogs are
  auto-cancelled and nothing shows on the user's screen. A batchmode editor never reaches the end of frame: complete
  games and screenshots run in a **windowed development build**.
- Everything long runs **in the background**, never in a blocking foreground loop.
- **Watch the watchdog**: follow `AutoplayRuns/alerts.log` for `WATCHDOG|RUN.FAIL|DIED|ABORT|TIMEOUT`; on an alert read
  the folder's `watchdog.json`, then write `extend <s>` or `abort <reason>` into `<folder>/watchdog-control.txt`.
  Never wait out a global timeout on a stuck run.
- Players launch **without focus**, muted, never cursor-locked: the user keeps working. Never minimize one (black
  captures). Never drive the user's real mouse or keyboard; real input uses virtual devices only.
- On Windows, stopping a task can leave child processes (bash, python, powershell, the game) alive: kill them too.

## 3. Pick the run mode

| Goal | Mode |
|---|---|
| Whole game / flow / errors / visuals | wrapper `build` (only if code changed) then `play-build <seed> <port>` |
| Replication / desync between host and clients | wrapper `play-net <clients> <seed>` (real client processes) → `compare_runs.py` verdict |
| Quick logic check that never needs end of frame | a PlayMode test calling `AutoplayRunner.Run` |
| Can a player actually do it through the UI? | `-autoplay-real-input` (virtual mouse / keyboard through the real EventSystem) |

Anything touching a client path is validated with `play-net`: host + simulated bots runs everything on the server and
hides client bugs. Coverage = several seeds; say which roles / features a run actually covered.

**Declarative scenarios** — prefer them for anything worth re-running: a JSON file (goal, mode, levers, expectations;
format in the docstring of `Tools~/run_scenario.py`), run with `python -X utf8 <tools>/run_scenario.py <scenario>`.
A passing scenario stays as a regression test; `Tools~/campaign.py` replays a folder + random games. In an analyzer
command, `{run}` is the run folder and `{tools}` the package's `Tools~`.

**Animations**: `-autoplay-record "kindRegex:seconds"` (trigger on an event BEFORE the animation) → frames +
`tracks.csv`; look at ONE contact sheet (`Tools~/contact_sheet.py`), assert with an `animation` check whose thresholds
come from a measured reference run, never invented.

## 4. Not covered? Extend, do not hand it back

Add the lever / event / check, then prove the goal with a scenario, following `EXTENDING.md` (definition of done:
compile, scenario PASS — on `play-net` if a client is involved — shown able to fail, reference updated). Hand a point
to a human playtest only when the extension is out of scope, and then say which extension would cover it.

## 5. Read the evidence — cheaply

1. Wrapper `last-run` (outcome, trace, counters, deduplicated errors).
2. A small goal-specific analyzer over the state files rather than reading hundreds of files.
3. A few targeted images only (the ones an analysis flags, plus one passing example).

## 6. Answer

- Verdict per goal (pass / fail / not covered) with the run folder, key capture paths and numbers.
- Separate **game bugs** (root cause only if proven: instrument with a tagged log before theorising), **tool problems**
  (fix them) and **design observations** (report only).
- Never commit `AutoplayRuns/`; stage explicit files only.
