---
name: Import a PVsyst project into PlantPredict
description: >-
  Send a PVsyst ZIP export to PlantPredict and poll until it becomes a project with one prediction per variant.
api: openapi/terabase-energy-plantpredict-openapi-original.yaml
operations:
  - importPVsystProject
  - getPVsystImportStatus
  - getProject
  - listPredictions
  - getPredictionOverview
generated: '2026-08-05'
method: generated
---

# Import a PVsyst project into PlantPredict

Migrates an existing PVsyst model into PlantPredict without touching the web UI.

## Steps

1. **Submit.** `importPVsystProject` (`POST /Project/Import/PVsyst`) with the ZIP
   exactly as PVsyst produced it. Returns an import job id.
2. **Poll.** `getPVsystImportStatus` (`GET /Project/Import/PVsyst/{jobId}`).
   Terabase's own guidance is to check **once every 10 seconds**. The job is visible
   only to the account that created it.
3. **Inspect the outcome.** The status result reports imported variants, skipped
   variants and failed variants separately (`PVsystImportedVariant`,
   `PVsystSkippedVariant`, `PVsystFailedVariant`, `PVsystBlockingReference` in the
   schema). Do not assume a completed job means every variant landed — read the
   per-variant lists.
4. **Confirm.** `getProject` and `listPredictions` on the new project id, then
   `getPredictionOverview` per prediction.

## What the API path does NOT do

The API import has **no review step** — everything that can be built is built. The
interactive review page (library matching, inline upload of a missing module/inverter/
weather file, project-wide resolution) exists only in the web application. If a variant
references a file missing from the archive, the API path resolves what it can and
reports the rest in the status result. Reconcile from the status payload, not from a UI.

## Known import behaviours worth checking after a run

Drawn from Terabase's published release notes (`changelog/terabase-energy-changelog.yml`):

- Timeframe start/end are stamped from the weather file — projects imported before
  release 12.30.0 have blank dates until re-imported.
- "Fixed tilted" plane variants import as Fixed Tilt; PVsyst states no row pitch for a
  fixed plane, so pitch is derived from the **company default GCR** and may need
  adjusting.
- Inverter set point maps from rated AC power (fixed in 12.29.0); check DC:AC ratios on
  anything imported earlier.
