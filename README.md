# UI Configuration Validator

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
- **Formats and minifies** the JSON, with a copy button for both the input and the canonically
  ordered output.
- **Documents every field**: path, type, allowed values, default, and the behaviour behind it,
  generated from the same schema the validator uses, so the two cannot drift apart.

It also flags the two behaviours that cause most support tickets:

- arrays are deep-merged **by index**, so a short `navigationWidget.cards` array leaves the
  default cards in the tail positions;
- a quoted `"false"` is truthy, so it switches a feature **on**.

## Hosting on GitHub Pages

```bash
git init
git add index.html README.md
git commit -m "Add UI configuration validator"
git remote add origin git@github.com:<org>/<repo>.git
git push -u origin main
```

Then in the repository: **Settings → Pages → Source: Deploy from a branch**, branch `main`,
folder `/ (root)`. The page is served at `https://<org>.github.io/<repo>/`.

`index.html` has no dependencies, so it works just as well opened directly from disk or served
from any static host.

## Keeping it in sync with the code

The schema, the defaults and the enum values are mirrored from the front-end repository:

| Page constant          | Source                                                        |
| ---------------------- | ------------------------------------------------------------- |
| `SCHEMA`               | `IUiConfiguration` in `src/shared/wrappers/firm-configuration/constants.tsx` |
| `DEFAULTS`             | `DEFAULT_UI_CONFIGURATION` in the same file                    |
| `DATE_RANGE_VALUES`    | `DATE_RANGES` in `src/api/models/auto-generated-metrics.tsx`   |
| `INVESTOR_CALC_CODES`  | `TREND_ANALYSIS_CALCULATION_CODE_OPTIONS` in `src/investor/containers/trend-analysis/constants.tsx` |
| `NAV_ICONS`            | `NAVIGATE_WIDGET_ICONS` in the firm-configuration constants    |

When a field is added to `IUiConfiguration`, add it to `SCHEMA` and `DEFAULTS` in `index.html`.
The documentation tab and the validator both read from those two constants, so no other edit is
needed.
