# Working in this repo

## What this is

One Home Assistant **blueprint** — `aquarium_control.yaml` — plus its docs.
There is no application here, no build, no test suite, no CI.

A blueprint is a parameterised automation template. Users import it, then create
automations *from* it, filling in `input:` fields through the Home Assistant UI.
The file declares `blueprint.input` (what users pick), `variables` (computed
values), `triggers`, `conditions` and `actions`.

Consequences that are easy to miss:

- **The file in this repo is not what runs.** Importing copies it to
  `config/blueprints/automation/<user>/aquarium_control.yaml` on the user's
  Home Assistant. Editing here changes nothing until they re-import. When
  debugging, confirm which version their install actually has.
- **A user's automation stores only the input values**, referencing the
  blueprint by path. So input keys are a public API: renaming one, or changing
  its selector type, breaks existing automations.
- **The import badge in `README.md` points at `main`**, not at a tag. Whatever
  is on `main` is what new users get, released or not.
- **It cannot be tested here.** Templates render only inside Home Assistant.
  Local checks catch YAML and Jinja mistakes; everything else is verified by the
  maintainer running it and exporting a trace.

## What it controls, and why that matters

A planted aquarium: a light on a daily sunrise/plateau/sunset curve, and a CO2
injection valve synchronised to it. This is live livestock, not a dashboard.

- **CO2 is the dangerous output.** Too much, or left on with the lights off,
  drops pH and can kill fish. It is deliberately forced off in maintenance mode,
  and it must stop before the lights do — plants stop consuming CO2 when the
  light goes. The offset input is negative: CO2 leads the lights at both ends
  (on early so it has dissolved before photosynthesis starts, off early for the
  same reason).
- **Light drives algae.** Too long or too bright grows algae rather than plants,
  which is why duration and max brightness are dashboard-adjustable helpers
  rather than fixed blueprint inputs — they get tuned over weeks.
- **Maintenance mode** is a temporary override for feeding or cleaning: full
  brightness, CO2 off.

A wrong schedule is not a cosmetic bug. Prefer aborting a run over acting on a
guessed value.

## How the automation actually works

Stateless and idempotent by design. It stores nothing between runs: every run
recomputes elapsed time from wall clock and the helper entities, derives the
brightness the lights *should* have right now, and acts on that. A missed run
is harmless because the next one recomputes from scratch — nothing accumulates
or drifts.

- A `time_pattern` trigger fires every minute, plus triggers for restart,
  maintenance toggle, and each helper entity.
- `mode: single` with `max_exceeded: silent` — an overlapping run is dropped
  without a log line. Don't rely on a trigger's run always happening.
- Lights are commanded only when the computed brightness differs from the
  previous minute, or when something other than the minute tick ran the
  automation. So "the schedule changed" is the signal, not "the light is wrong".
  Anything that makes reality diverge without the schedule moving — a manual
  change, a light returning from unavailable — is corrected at the next
  brightness change, not immediately. This is a deliberate tradeoff; see
  `CHANGELOG.md` for 2.0.0.

The three helper entities (start time, duration, max brightness) live in the
user's Home Assistant, not here. They are `input_datetime` / `input_number`
helpers the user creates by hand, per the README, and the blueprint reads them
at runtime so they can be changed from a dashboard without editing automations.

## Syntax: track current Home Assistant

Use the current forms, not the legacy ones that still happen to work.

- `triggers:` / `conditions:` / `actions:`, not the singular keys
- `action:` for service calls, not `service:`
- `trigger: <platform>` inside a trigger, not `platform:`
- Purpose-built template functions over hand-rolled equivalents:
  `has_value(entity)` rather than comparing `states(entity)` against a list of
  `unavailable` / `unknown`

When a current form raises the minimum version, declare it rather than leaving
it implied:

```yaml
blueprint:
  homeassistant:
    min_version: 2024.10.0
```

The plural keys need 2024.10. If you use a selector option or template function
added later, raise `min_version` to match — an undeclared requirement fails at
import with a confusing error instead of a clear one.

## Selectors and inputs

Pass a `target:` selector straight to the action. Do not expand it into entity
ids unless something genuinely needs per-entity state, and if you do, say why in
a comment — the obvious simplification is to delete it, and the next reader will
try.

```yaml
- action: light.turn_on
  target: !input light_entities     # not a resolved entity list
```

Two things that have bitten this repo:

- **A state trigger cannot take a target.** `entity_id:` needs entity ids, so an
  input using a target selector cannot also drive an availability trigger.
- **Changing an input's selector type is breaking.** Home Assistant *merges* the
  stored value into the new shape rather than replacing it, leaving inputs like
  `{'0': 'light.x', 'entity_id': 'light.x'}`. A target schema rejects the stray
  key and the automation refuses to load. Document it, bump the major version,
  and quote the exact error users will see.

## Template facts worth not rediscovering

- `variables:` render in order; each is available to the next, with
  `literal_eval` applied — a template rendering `[1, 2]` yields a real list, so
  indexing the result of a previous variable works.
- `trigger` is always defined. A manual "Run actions" supplies
  `{'platform': None}` — defined, but with no `id`. Guard with
  `trigger.id | default('', true)`, not `trigger is not defined`.
- Manual runs skip top-level `conditions:` entirely. A guard there protects
  scheduled runs only.
- Jinja supports `**` unpacking, so `timedelta(**duration_input)` is fine.
- The duration selector is positive-only unless you set `allow_negative: true`.

## Don't invent fallbacks

A default that lets the automation keep running with invented data is worse than
stopping. This repo shipped a start time that fell back to `08:00:00` when the
helper was unavailable; for a tank scheduled at 14:00 that silently ran the
lights six hours early. It was replaced with a variable plus a condition that
aborts the run.

If a template would raise while variables render, use a placeholder to survive
rendering *and* a condition that stops the run before the placeholder is used.

Likewise, don't add defensive handling for a failure mode you have only
imagined. Fix what is observed.

## Comments

Comments explain **why**. If a comment restates the line below it, delete it.

```yaml
# bad
# Convert CO2 offset to seconds
co2_offset_seconds: "{{ timedelta(**co2_offset).total_seconds() | int }}"

# good
# Forced off during maintenance, for safety while working in the tank
co2_should_be_on: >-
```

## Verify before committing

Templates are not type-checked and a broken blueprint fails at runtime, on a
tank, unattended. Check what can be checked:

```bash
# YAML parses, with the Home Assistant tags registered
python3 -c "
import yaml
class L(yaml.SafeLoader): pass
L.add_constructor('!input', lambda l,n: {'!input': l.construct_scalar(n)})
yaml.load(open('aquarium_control.yaml'), L); print('OK')"
```

For non-trivial Jinja, render it locally with `jinja2` and mocked HA filters
before committing, and compare against the logic being replaced across the
boundary cases — not just the middle of the range. Refactors of the brightness
curve have been checked this way at every phase boundary.

State clearly what was verified and what was not. `expand()`, `has_value()`,
`device_entities` and friends only exist inside Home Assistant; local tests
mock them, and that is not the same as the code working.

## Debugging: traces are the source of truth

Ask for a trace export (Settings → Automations → Traces → download) rather than
reasoning from the config. The trace carries **Changed Variables**, with every
rendered value and type, plus the result of every condition and which branch
ran. Several confident theories in this repo's history were killed by one trace.

Read the numbers before proposing a fix. One reported "bug" here turned out to
be correct behaviour observed four minutes before a cutoff.

## Docs must match the code

Three surfaces drift, in rising order of how often they are missed:

1. `README.md`
2. `CHANGELOG.md`
3. the blueprint's own `description:` — what users read in the import dialog,
   and the one that goes stale

When behaviour changes, update all three in the same commit, including the
tradeoffs. If a light that goes offline is no longer corrected within 60
seconds, the README says so.

## Versioning and releases

Semantic versioning. A change requiring users to touch their existing
automation is a major bump, whatever its size. Record breaking changes under a
`### Breaking` heading at the top of the release, with the remedy and the exact
error.

Version lives in `README.md` and `CHANGELOG.md` only — blueprints have no
version field. Tag annotated, as `vX.Y.Z`.

## Commits

Commit messages explain the reasoning, not the diff: what was wrong, why the
chosen fix, and what tradeoff it accepts. Commits are gpg-signed; if signing
times out, retry rather than disabling it.
