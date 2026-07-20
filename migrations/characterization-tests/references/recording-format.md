# Recording Format

Surface-neutral structure — one markdown file per interaction under `migration/characterization/recordings/`, named `<surface>_<stimulus-slug>.md`. Surface references (e.g. [http.md](http.md)) define how stimulus and observation map to their protocol.

```markdown
# <stimulus, in surface notation>

surface: <http | jms | batch | cli | db>
recorded: <YYYY-MM-DD>, target: <coordinates>
seed: <data assumptions, or "none">
accepted: <name>, <YYYY-MM-DD> — <reason>        <!-- only after a reconcile decision -->

## Expected

<surface-specific observation metadata>

## Observation (normalized)

<normalized observation>
```

## Normalization Principle

Apply identically at record and replay time — a rule applied on one side only produces false diffs. Mask volatile values with placeholder tokens:

| Volatile content | Replacement |
|---|---|
| Timestamps, dates, times | `«TS»` |
| Session/correlation identifiers | `«SESSION»` |
| Security tokens | `«TOKEN»` |
| Generated ids (sequences, UUIDs) | `«ID»` |

- Masking is per-pattern, not per-recording: define system-specific patterns (id formats, date formats) once in `migration/characterization/normalization.md` and apply everywhere.
- Surface references add protocol-specific rules (headers to drop, ordering guarantees) on top of these.

## Rules

- A recording with an unmaskable volatile value (e.g. a computed total that legitimately varies) narrows its assertion: record the stable part, note the exclusion under `seed:`.
- Never hand-edit a normalized observation except through a reconcile decision with an `accepted:` line.
