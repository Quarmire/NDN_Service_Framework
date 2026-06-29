# Feature 020: DI Auto Evidence Documentation Sync

Status: Superseded

## Goal

Move the accepted auto-assignment campaign evidence from Feature 019 into the
durable project documentation and proposal defense slides.

Superseded: the proposal-defense slide was removed because this NDNSF-DI auto
campaign is too detailed for the proposal deck. The experiment documentation
may still keep the evidence, but the PPT/PDF deck should not include the extra
auto-selection slide.

## Scope

- Update NDNSF-DI experiment documentation with the 5-run auto campaign.
- Update the native DI roadmap to replace the older concurrency-boundary note
  with the accepted auto-selection evidence.
- Add one concise proposal-defense slide for auto layout selection.
- Update speaker notes for the new slide.

## Non-Goals

- New experiments.
- Changing runtime behavior.
- Regenerating PPTX.
- Editing unrelated proposal chapters.

## Acceptance

- [x] Documentation names the result directory and command.
- [x] Documentation records c1/c2/c4 selected candidates and latency evidence.
- [x] Slides compile after removing the auto layout selection page.
- [x] Speaker notes compile after removing the matching note.

## Accepted Evidence

Updated documentation:

- `docs/experiments.md`
- `docs/native-di-roadmap.md`

The proposal-defense slide addition was later reverted. Keep this feature as
historical context for why the experiment was documented, but do not treat it
as current slide-deck content.
