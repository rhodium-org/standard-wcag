# standard-wcag

The **Web Content Accessibility Guidelines (WCAG) 2.2** expressed as a
[throughline](https://pypi.org/project/throughline/) **source** — a standalone, grounded
requirements graph that a consuming project composes with
[throughline-compose](https://github.com/rhodium-org/throughline-compose).

This repository holds no application code. It is a directory of small YAML items with
permanent UIDs, validated by `tl check`. Consumers import it under a namespace and
reference its criteria as `wcag:SR-0019`.

## Status

A grounded graph of
<!-- tl:count type == 'intent' -->
4
<!-- tl:end --> principle intents,
<!-- tl:count type == 'user_requirement' -->
13
<!-- tl:end --> guideline requirements and
<!-- tl:count type == 'system_requirement' -->
86
<!-- tl:end --> success criteria, published to [`docs/spec.md`](docs/spec.md). The counts
are rendered from the live graph by `tl:count`, so they cannot drift.

## Why this source is multi-root

A throughline source gets **as many root intents as the standard has genuine "why"s** — a
single umbrella root would throw away the reason each criterion exists, which is the whole
point of IDD. WCAG's structure gives four distinct reasons, so this graph has **four
co-equal root intents** (all `normative: false`):

| Root | Principle |
|---|---|
| `INT-0001` | **Perceivable** — content is available to a sense the user has |
| `INT-0002` | **Operable** — the interface can be operated by any input method |
| `INT-0003` | **Understandable** — information and operation make sense |
| `INT-0004` | **Robust** — content survives across user agents and assistive tech |

- Each of WCAG's **13 guidelines** is a `user_requirement` that `derives_from` **its own
  principle** (not a catch-all), and carries a `rationale` — who it serves and why.
- Each **success criterion** is a `system_requirement` that `implements` its guideline.
  Its `rationale` is the W3C **"Intent of this Success Criterion"** — the leaf-level *why*.
  WCAG's hierarchy is strict (one principle per guideline), so a criterion grounds up to a
  principle through `implements` → `derives_from` with no extra edge.

## Versions and conformance levels — attributes, not forks

WCAG 2.1 is a superset of 2.0 and 2.2 a near-superset of 2.1, so **one graph carries all
three versions at once** — you don't fork the repo or duplicate criteria per version:

- `attrs.wcag_version` — the version that **introduced** the criterion (`2.0` / `2.1` /
  `2.2`). Filter `wcag_version in {2.0, 2.1}` to get "the WCAG 2.1 set".
- `attrs.level` — the conformance grade (`A` / `AA` / `AAA`).
- `attrs.source_ref` — the published WCAG number (`1.4.3`), never the UID.
- `attrs.wcag_removed` — a **declared, validated** attribute (optional, values `{2.2}`)
  naming the version that **withdrew** the criterion. A criterion is present in version `V`
  exactly when `wcag_version ≤ V < wcag_removed`.
- The one criterion removed in 2.2 — **4.1.1 Parsing** — is kept as a throughline
  **tombstone** (`status: deleted`, `attrs.wcag_removed: "2.2"`), never dropped, so UID
  history stays intact.

**Removed is not the same as superseded.** 4.1.1 was *withdrawn without a named successor*,
so it is a pure tombstone. When a future edition **replaces** a criterion with a named one,
the withdrawn criterion also carries a directional `superseded_by` link (`system_requirement`
→ `system_requirement`) pointing at its replacement — the *why* survives the swap. That link
type and its endpoint rule are declared in `throughline.toml`; WCAG 2.2 has no such case, so
no `superseded_by` edge exists yet (the generator's `SUPERSEDED_BY` map is empty). The
schema is ready, so a real supersede is a data change, not a schema change.

**Editions are git tags of this one repo.** `v2.2.0` tags the WCAG 2.2 edition; a future
WCAG 3.0 (a different model) would be `v3.0.0` on this same repo. A consumer pins
`wcag@v2.2.0`. This is the same editions-as-tags model as `standard-asvs`, chosen over
per-version folders because the versions are additive and share one immutable UID space.

## Modelling conventions

- **throughline UIDs are this source's own** (`SR-0019`…), immutable and never the WCAG
  number. The WCAG number lives in `attrs.source_ref`.
- **Success-criterion `text`** is the criterion's normative statement; **`rationale`** is
  its published Intent. Both come from the authoritative W3C sources (see NOTICE).
- The graph is generated from the vendored canonical JSON by `tools/generate.py`
  (permanent-UID, additive); re-run it after `tools/fetch_intents.py` refreshes the Intent
  paragraphs. Editing the spec means editing the data + generator, not the YAML by hand.

## Composing it

In a consuming project's `throughline.toml`:

```toml
[sources.wcag]
url = "https://github.com/rhodium-org/standard-wcag"
ref = "v2.2.0"
```

Then reference a criterion as `wcag:SR-0019` (Contrast Minimum) from your own items.

## Licence

Repository structure, tooling and configuration: Apache-2.0 (see `LICENSE`). The WCAG
criterion text and Intent paragraphs are © W3C, reused under the W3C Document License with
attribution — see `NOTICE`. Authoritative source: https://www.w3.org/TR/WCAG22/.
