# UI Configuration Validator

**→ [uladzislau-ustsinovich.github.io/JSON-Validator](https://uladzislau-ustsinovich.github.io/JSON-Validator/)**

A single-page tool for checking a firm's UI configuration JSON (`FirmConfiguration.uiConfig`,
typed as `IUiConfiguration`) before it is saved.

Everything runs in the browser. Nothing that is pasted into the page leaves it — there is no
network call, no analytics and no build step.

## What it does

- **Validates** the JSON: syntax errors with line and column, duplicate keys, unknown fields,
  wrong types, and values outside the supported enums.
- **Checks placement.** A real field at the wrong path is silently ignored by the app — no error,
  no effect. The validator reports the path the app actually reads and can move unambiguous cases
  for you with *Fix placement*.
- **Validates against a chosen release.** The version picker switches between 6.0, 6.1 and
  6.2 (develop), and the choice is remembered. A value that is legal on a newer release is reported
  as a version mismatch rather than a typo.
- **Formats and minifies** the JSON, with a copy button for both the input and the canonically
  ordered output.
- **Documents every field**: path, type, allowed values, default, and the behaviour behind it,
  generated from the same schema the validator uses, so the two cannot drift apart.

It also flags the two behaviours that cause most support tickets:

- arrays are deep-merged **by index**, so a short `navigationWidget.cards` array leaves the
  default cards in the tail positions;
- a quoted `"false"` is truthy, so it switches a feature **on**.

Keys the team adds on purpose — `Note`, `Note2`, `showApiTagVersion-WARNING`, anything ending in
`-WARNING` / `-NOTE` / `-TODO` or starting with `_` — are reported as notes, not errors. JSON has
no comment syntax, and those keys are how a note gets left in the file.

## Version differences

6.0 and 6.1 carry an identical configuration schema. They differ from develop (6.2) in two ways
that change what a firm should send:

| What differs                | 6.0 and 6.1                              | 6.2 (develop)       |
| --------------------------- | ---------------------------------------- | ------------------- |
| `defaultPageLevelDateRange` | no `10_years`, `15_years`, `20_years`    | all three available |
| `emailAssistant.actions`    | 5 built-in actions                       | `[]`                |
| `emailAssistant.tones`      | 4 built-in tones                         | `[]`                |

Everything else is the same — same fields, same nesting, same navigation-widget rules including
the six-card cap.

The email assistant is the trap when moving between releases. On 6.0 and 6.1, sending two tones
does not give a firm two tones: positions 1–2 are your entries merged key by key over the first
two built-ins, and positions 3–4 are the remaining built-ins, untouched. The same configuration
on 6.2 yields exactly the two tones you sent.

The built-in action and tone entries are firm-internal (prompt templates and agent ids), so this
public page models them by shape and count rather than reproducing them. That is all the validator
needs in order to warn correctly.

## Hosting

The page is served from GitHub Pages at
[uladzislau-ustsinovich.github.io/JSON-Validator](https://uladzislau-ustsinovich.github.io/JSON-Validator/),
built from `index.html` on `main`. Pushing to `main` redeploys it within a minute or two.

`index.html` has no dependencies and makes no network calls, so it works just as well opened
straight from disk or served from any other static host.

## Keeping it in sync with the code

The schema, the defaults and the enum values are mirrored from the front-end repository:

| Page constant         | Source                                                                                             |
| --------------------- | -------------------------------------------------------------------------------------------------- |
| `SCHEMA`              | `IUiConfiguration` in `src/shared/wrappers/firm-configuration/constants.tsx`                        |
| `DEFAULTS_62`         | `DEFAULT_UI_CONFIGURATION` in the same file, on `develop`                                          |
| `VERSIONS`            | per-release deltas: `DATE_RANGES` and the `emailAssistant` defaults                                |
| `DATE_RANGES_62`      | `DATE_RANGES` in `src/api/models/auto-generated-metrics.tsx`                                       |
| `INVESTOR_CALC_CODES` | `TREND_ANALYSIS_CALCULATION_CODE_OPTIONS` in `src/investor/containers/trend-analysis/constants.tsx` |
| `NAV_ICONS`           | `NAVIGATE_WIDGET_ICONS` in the firm-configuration constants                                        |

When a field is added to `IUiConfiguration`, add it to `SCHEMA` and `DEFAULTS_62` in `index.html`.
The documentation tab and the validator both read from those constants, so no other edit is needed.

When a release diverges, add the delta to `VERSIONS`:

- a differing enum goes in `enums` and is referenced from the node with `valuesKey`;
- a differing array default goes in `builtInArrays` as a count and a summary;
- a field that only exists in some releases is the one case that still needs a small code change.
