---
name: Run a PlantPredict energy prediction
description: >-
  Create a project, configure a prediction, attach a power plant design, queue the simulation and read the results
  from the PlantPredict Performance API.
api: openapi/terabase-energy-plantpredict-openapi-original.yaml
operations:
  - createProject
  - createPrediction
  - createPowerPlant
  - runPrediction
  - getPredictionOverview
  - getResultSummary
  - getResultDetails
  - cancelPrediction
generated: '2026-08-05'
method: generated
---

# Run a PlantPredict energy prediction

Base URL `https://api.plantpredict.terabase.energy`. Every request needs
`Authorization: Bearer <token>` — get the token from the Cognito client-credentials
endpoint (`authentication/terabase-energy-authentication.yml`), requesting both
`transactions/get` and `transactions/post`.

## Before you start

Confirm the library objects the prediction will reference already exist, because a
prediction cannot run without them:

- a **Weather** file — `searchWeather`, or import one (see the weather-import skill)
- a **Module** — `listModules`
- an **Inverter** — `listInverters`

Enum values (model types, statuses) are not frozen into the spec. Call `getDefinitions`
once and resolve integers against that catalog rather than hard-coding them.

## Steps

1. **Create the site.** `createProject` (`POST /Project`) with the site name and
   lat/lon. The response is `{"id": <integer>}`. Watch for an `X-Message` response
   header — a duplicate project name is warned about, not rejected, and the call is
   **not** idempotent, so a retry creates a second project. There is no idempotency
   key; on a timeout, call `searchProjects` or `getMyProjects` to check before retrying.
2. **Create the prediction.** `createPrediction`
   (`POST /Project/{projectId}/Prediction`) with the simulation period, model
   selections and the `weatherId`. Returns `{"id": <integer>}`. This operation
   declares `429` — back off exponentially from 30s if you hit it; no `Retry-After`
   header is sent.
3. **Attach the plant design.** `createPowerPlant`
   (`POST /Project/{projectId}/Prediction/{predictionId}/PowerPlant`) with the
   topology: `blocks[] -> arrays[] -> inverters[] -> dcFields[]`, where each inverter
   carries an `inverterId` and each DC field a `moduleId` from the library. Also
   declares `429`.
4. **Queue the run.** `runPrediction`
   (`POST /Project/{projectId}/Prediction/{predictionId}/Run`). This is asynchronous;
   it returns immediately.
5. **Poll.** `getPredictionOverview`
   (`GET /Project/{projectId}/Prediction/{predictionId}/Overview`) until `status`
   reaches `2` (Issued/complete). Poll on a fixed interval; do not busy-loop.
   `cancelPrediction` stops a run that is still in flight — a completed prediction
   cannot be cancelled and returns `400 text/plain`.
6. **Read the results.** `getResultSummary` for headline yield, `getResultDetails`
   for the loss tree, `getAverageEnergy` for the long-run average, `getNodalJson` for
   nodal output.

## Error handling

Branch on `Content-Type`, not on the status code alone:

- `400 application/json` -> `{message, modelState}`; `modelState` maps field names to
  lists of messages. Fix the named fields.
- `400 text/plain` -> a runtime/database message, e.g. "A successfully completed
  prediction cannot be cancelled."
- `401` -> empty body, no `Content-Type`. The token expired; fetch a new one.
- `403` -> empty body. The token lacks `transactions/post` or the account lacks the
  elevated role that status-change operations require.
- `404 text/plain` -> the resource does not exist or is not visible to this company.
- `429` -> empty body. Exponential backoff from 30s.
- `500 text/plain` -> opaque. Safe to retry a GET; for a POST verify state first.

Full catalogue: `errors/terabase-energy-problem-types.yml`.
