---
name: Terabaseenergy
description: Use when building utility-scale solar energy predictions, configuring power plant designs, managing weather and equipment libraries, running simulations, analyzing results, or automating workflows via REST API or Python SDK. Reach for this skill when working with projects, predictions, modules, inverters, weather data, and energy storage systems.
metadata:
    mintlify-proj: terabaseenergy
    version: "1.0"
---

# PlantPredict Skill Reference

## Product summary

PlantPredict is a cloud-based performance modeling platform for utility-scale solar PV systems. It combines sophisticated photovoltaic algorithms with an intuitive web interface to deliver bankable energy yield predictions throughout the project lifecycle—from pre-development through operational monitoring. The platform supports complex system designs (multiple blocks, arrays, inverters), 3D shading analysis, energy storage integration, and uncertainty analysis (P50, P90, P99).

**Key entry points:**
- Web UI: https://plantpredict.terabase.energy
- API base URL: https://api.plantpredict.terabase.energy
- Python SDK: `pip install plantpredict`
- OpenAPI spec: `/api-docs/api-reference/plantpredict-api.yaml`
- Primary docs: https://docs.plantpredict.com

**Core workflow:** Create Project → Add Weather/Modules/Inverters → Create Prediction → Configure PowerPlant → Set Simulation Settings → Run Prediction → Analyze Results

## When to use

Reach for PlantPredict when:
- **Building energy predictions** for solar projects at any stage (prospecting, engineering, construction validation, operations)
- **Designing power plants** with multiple blocks, arrays, inverters, and DC fields
- **Analyzing shading impacts** using 3D terrain-aware modeling
- **Modeling hybrid PV+battery systems** with custom dispatch strategies
- **Running uncertainty analysis** to generate P50, P90, P99 exceedance probabilities for project finance
- **Automating workflows** via REST API or Python SDK (batch predictions, parameter sweeps, integrations)
- **Comparing design scenarios** across multiple predictions in a single project
- **Exporting detailed results** for financial analysis, nodal data, or external tools

## Quick reference

### Core entities and hierarchy

| Entity | Purpose | Parent | Children |
|--------|---------|--------|----------|
| **Project** | Named site (lat/lon) | — | Predictions |
| **Prediction** | Simulation configuration | Project | PowerPlant, Weather, TimeSeries |
| **PowerPlant** | Physical plant design | Prediction | Blocks → Arrays → Inverters → DC Fields |
| **Weather** | Hourly meteorological data | Company library | Referenced by Prediction |
| **Module** | PV panel specification | Company library | Referenced by DC Field |
| **Inverter** | Inverter specification | Company library | Referenced by Array |
| **Shade Scene** | 3D terrain/objects | Prediction | DC Field shading calculations |

### API authentication (OAuth 2.0 Client Credentials)

```
Token URL: https://terabase-prd.auth.us-west-2.amazoncognito.com/oauth2/token
Scopes: transactions/post transactions/get
Credential type: HTTP Basic auth (Client ID:Secret)
API base: https://api.plantpredict.terabase.energy
```

Generate credentials via UI: Gear icon → User profile → **Generate API Credentials** (admin only; non-admins request from company admin).

### Essential API endpoints

| Task | Endpoint | Method |
|------|----------|--------|
| Create project | `POST /Projects` | POST |
| List projects | `GET /Projects` | GET |
| Create prediction | `POST /Projects/{projectId}/Predictions` | POST |
| Create PowerPlant | `POST /Projects/{projectId}/Predictions/{predictionId}/PowerPlant` | POST |
| Run prediction | `POST /Projects/{projectId}/Predictions/{predictionId}/Run` | POST |
| Get results summary | `GET /Projects/{projectId}/Predictions/{predictionId}/Results/Summary` | GET |
| Get detailed results | `GET /Projects/{projectId}/Predictions/{predictionId}/Results/Detailed` | GET |
| Upload weather | `POST /Weather/Parse` → `POST /Weather/Finalize` | POST |
| Create module | `POST /Modules` | POST |
| Create inverter | `POST /Inverters` | POST |

### Prediction statuses

| Status | Code | Meaning |
|--------|------|---------|
| Draft (Private) | 0 | Visible only to creator |
| Draft (Shared) | 1 | Visible to company |
| Active | 2 | Approved for use |
| Retired | 3 | Archived |

### Simulation model choices (key parameters)

- **Irradiance decomposition:** DISC, ERBS, Boland, Orgill-Hollands
- **Transposition model:** Hay-Davies, Perez, 3D transposition
- **Module temperature:** Heat Balance, Sandia, NOCT-SAM, measured surface
- **Shading:** Horizon, sky diffuse, ground-reflected, direct beam (geometric or 3D scene)
- **Degradation:** Linear, stepped, LeTID (light-induced degradation)
- **Tracking:** Fixed tilt, true tracking, backtracking, terrain-aware backtracking

## Decision guidance

### When to use UI vs API

| Scenario | Use UI | Use API |
|----------|--------|---------|
| One-off project design, interactive exploration | ✓ | — |
| Batch predictions, parameter sweeps, automation | — | ✓ |
| Comparing 2–3 design scenarios | ✓ | ✓ |
| Integrating with external tools (ERP, finance, GIS) | — | ✓ |
| Rapid prototyping with Python | — | ✓ |

### When to use Block Builder vs Map Builder

| Approach | Use when | Pros | Cons |
|----------|----------|------|------|
| **Block Builder** | Defining plant hierarchy manually, complex multi-block designs | Full control, precise specifications | More steps, manual entry |
| **Map Builder** | Designing on geographic layout, automated array placement | Visual, fast layout, terrain integration | Less granular control |

### When to use 2D vs 3D shading

| Model | Use when | Accuracy | Compute time |
|-------|----------|----------|--------------|
| **Horizon shading** | Simple terrain, distant obstructions | Low | Fast |
| **3D scene** | Complex terrain, row-to-row shading, detailed site analysis | High | Slower |
| **Geometric 3D** | Tracker systems with terrain awareness | Medium-high | Medium |

## Workflow

### Typical prediction workflow

1. **Create project**
   - Navigate to Projects → Create New Project
   - Enter name, latitude, longitude, timezone
   - Confirm location details

2. **Prepare libraries** (if not already done)
   - Import or create weather file (Weather Library)
   - Import or create module (Module Library)
   - Import or create inverter (Inverter Library)
   - Verify all assets are marked Active or Global

3. **Create prediction**
   - Open project → Create New Prediction
   - Select weather file, module, inverter defaults
   - Confirm prediction created in Draft status

4. **Configure environmental conditions**
   - Click Update (Weather Data)
   - Select weather file, set soiling/spectral/albedo if needed
   - Save and return

5. **Design power plant**
   - Click Update (PV Blocks & Arrays)
   - Choose Block Builder or Map Builder
   - Define blocks, arrays, inverters, DC fields
   - Set DC/AC ratios, GCR, mounting type
   - Save PowerPlant

6. **Configure simulation settings**
   - Click Update (Model Choices)
   - Select irradiance decomposition, transposition, temperature, shading, degradation models
   - Set prediction timeframe (start/end dates)
   - Configure uncertainty analysis (P50, P90, P99) if needed
   - Save

7. **Run prediction**
   - Click Run Prediction button
   - Monitor queue status
   - Wait for completion (minutes to hours depending on complexity)

8. **Review results**
   - Click View Energy Results
   - Review Summary Results (annual energy, losses, P-values)
   - Explore P50 Loss Tree (energy flow diagram)
   - Export nodal data if detailed component analysis needed

### API workflow (Python SDK example)

```python
from plantpredict.client import PlantPredictClient

# Authenticate
client = PlantPredictClient(
    client_id="YOUR_CLIENT_ID",
    client_secret="YOUR_CLIENT_SECRET"
)

# Create project
project = client.projects.create(
    name="My Solar Project",
    latitude=35.5,
    longitude=-106.5
)

# Create prediction
prediction = client.predictions.create(
    project_id=project.id,
    name="Base Case"
)

# Create power plant
power_plant = client.power_plants.create(
    project_id=project.id,
    prediction_id=prediction.id,
    blocks=[...]  # Define blocks, arrays, inverters
)

# Run prediction
client.predictions.run(
    project_id=project.id,
    prediction_id=prediction.id
)

# Get results
results = client.results.get_summary(
    project_id=project.id,
    prediction_id=prediction.id
)
print(f"P50 Energy: {results.p50_energy} MWh")
```

## Common gotchas

- **PowerPlant must be created before running.** A prediction starts in Draft status and requires a PowerPlant (via `POST .../PowerPlant`) before it can be queued. The API will reject run requests if PowerPlant is missing.

- **Weather file validation is strict.** Ensure GHI, DNI, DHI, temperature, and wind speed are within realistic ranges. Use the Quality Check endpoint before finalizing imports. Missing spectral data (humidity, dewpoint, precipitable water) silently disables spectral corrections.

- **Prediction status must be Draft or higher to run.** Retired predictions cannot be run. Change status before attempting to run.

- **API credentials are shown only once.** Store them securely (environment variables, secret manager). Regenerating credentials invalidates old ones immediately.

- **Batch predictions have limits.** UI allows up to 10 simultaneous runs from the Predictions page. API batch endpoints have different constraints; check endpoint documentation.

- **Cloned predictions get "– CLONE" suffix.** Rename immediately to avoid confusion in project libraries.

- **3D shade calculations are asynchronous.** Queue shade calculations separately; check processing status before running prediction.

- **Nodal data export is optional.** Configure which parameters to export via Nodal Data options before running; this affects file size and processing time.

- **Time series data must align with weather file.** Custom time series (tracker angles, LGIA limits, surface temperature) must have matching timestamps and frequency as the weather file.

- **Uncertainty analysis requires multiple runs.** P90, P99 calculations add processing time; P50 is always calculated.

## Verification checklist

Before submitting a prediction or exporting results:

- [ ] **Weather file is Active or Global** — Check Weather Library status
- [ ] **Module and inverter are Active or Global** — Check Module/Inverter Library status
- [ ] **PowerPlant is created and saved** — Verify via Prediction Overview
- [ ] **Simulation settings are configured** — Confirm timeframe, models, and degradation
- [ ] **Prediction has run successfully** — Check status; no errors in queue
- [ ] **Results are available** — View Energy Results button is active
- [ ] **P-values match requirements** — Verify P50, P90, P99 if needed for finance
- [ ] **Nodal data is configured** (if needed) — Check Nodal Data options
- [ ] **Project/prediction status is appropriate** — Draft for development, Active for approval
- [ ] **Export format is correct** — JSON for API, Excel/CSV for UI exports

## Resources

**Comprehensive documentation index:** https://docs.plantpredict.com/llms.txt

**Critical reference pages:**
1. [API Introduction & Domain Model](https://docs.plantpredict.com/api-docs/intro) — Start here for API concepts and authentication
2. [API Quick Start Guide](https://docs.plantpredict.com/api-docs/api_quick_start_guide) — Step-by-step OAuth setup and first request
3. [Prediction Overview](https://docs.plantpredict.com/user-guide/ui/prediction-overview) — Central hub for configuring predictions in the UI
4. [Models & Algorithms Introduction](https://docs.plantpredict.com/models/general/intro) — Technical foundation of the simulation engine
5. [Python SDK Repository](https://github.com/plantpredict/python-sdk) — Source code, examples, and community support

---

> For additional documentation and navigation, see: https://docs.plantpredict.com/llms.txt