---
name: Import a weather file into the PlantPredict library
description: >-
  Find, download or upload an hourly weather file, quality-check it, and finalize it into the company weather library
  so predictions can reference it.
api: openapi/terabase-energy-plantpredict-openapi-original.yaml
operations:
  - searchWeather
  - validateWeatherDownload
  - downloadWeather
  - parseWeather
  - qualityCheckWeather
  - finalizeWeatherImport
  - finalizeGeneratedWeather
  - getWeather
  - getWeatherDetails
  - updateWeatherDetails
  - getHorizonScene
generated: '2026-08-05'
method: generated
---

# Import a weather file into the PlantPredict library

Weather files live in a **company-wide** library and are referenced by Predictions
through `weatherId`. Import once, reuse across projects.

## Path A — search the existing library first

`searchWeather` (`GET /Weather/Search`) before importing anything. A file for the
same site may already be in the company library, and duplicate weather files are the
most common avoidable cost in this API.

## Path B — download from a provider

1. `validateWeatherDownload` (`POST /Weather/Download/ValidateDownload`) to check the
   location and provider combination is available before spending a download.
2. `downloadWeather` (`POST /Weather/Download/{providerId}`) to pull the file.
3. `finalizeGeneratedWeather` (`POST /Weather/FinalizeGeneratedWeather`) to commit it
   to the library.

## Path C — upload your own file

1. `parseWeather` (`POST /Weather/Parse`) to read the uploaded file into PlantPredict's
   internal structure.
2. `qualityCheckWeather` (`POST /Weather/QualityCheck`) — do not skip this; it is the
   step that catches unit and mapping errors before they silently distort a prediction.
3. `finalizeWeatherImport` (`POST /Weather/FinalizeImport`) to commit.

## After import

- `getWeather` / `getWeatherDetails` to read the record back and confirm the fields
  landed as expected.
- `updateWeatherDetails` / `updateWeatherTitle` to correct metadata.
- `getHorizonScene` (`GET /Weather/GetHorizonScene`) for the horizon profile at the
  weather file's location.
- `exportWeather` to pull the file back out.

## Notes

- All of these operations are `transactions/post` scope except the GETs.
- None of them is idempotent and none accepts an idempotency key; a repeated finalize
  creates a second library record. Search before you retry.
- Conventions: `conventions/terabase-energy-conventions.yml`.
