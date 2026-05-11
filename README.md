# Pharmacy Patient Profile — HAPI FHIR Demo

A single-page web app that pulls a unified patient profile from a [HAPI FHIR](https://hapifhir.io/) server — modeled around what a **pharmacist** needs to see before dispensing a medication: identity, allergies, active prescriptions, and conditions.

Built as a hands-on demo of the FHIR R4 resource model and REST API. No framework, no build step — one HTML file, vanilla JavaScript, ~450 lines.

> **Live demo:** _coming soon (GitHub Pages)_
> **Backend:** Public HAPI sandbox at [`hapi.fhir.org/baseR4`](https://hapi.fhir.org/baseR4)

---

## Why this project

A pharmacist's first 30 seconds with a patient are a safety check: *Who is this person? What are they allergic to? What are they already on? What conditions might contraindicate the new prescription?*

In a paper-and-phone world, the pharmacist asks the patient and trusts memory. In a FHIR-enabled world, those answers come from four standard resources that any FHIR-compliant system can serve — regardless of which EHR generated them. This project shows that pattern end-to-end against a real FHIR server.

The bigger point: FHIR makes patient context *portable*. Same code that reads from this public sandbox would work against an Epic, Cerner, or HAPI-based deployment in production with no architectural changes.

---

## What it does

| Feature | FHIR mechanic |
|---|---|
| Search patient by name | `GET /Patient?name={q}` returns a `Bundle` of matching `Patient` resources |
| View patient demographics | `GET /Patient/{id}` — name, gender, birthDate, telecom |
| List allergies with severity badges | `GET /AllergyIntolerance?patient=Patient/{id}` |
| List active medications with dosage instructions | `GET /MedicationRequest?subject=Patient/{id}&status=active` |
| List clinical conditions | `GET /Condition?subject=Patient/{id}` |
| Seed a fully-loaded demo patient (one click) | 7 sequential `POST`s — Patient, 2× AllergyIntolerance, 2× MedicationRequest, 2× Condition |

All four detail queries fire **in parallel** from the patient detail view — a small but realistic optimization for FHIR clients (`Promise.all`).

---

## FHIR resources & references

The data model mirrors how real EHRs structure clinical context. References are stored as IDs (not embedded data), so the same Patient can be the `subject` of unbounded Observations, Conditions, MedicationRequests, etc., without duplication.

```
                ┌──────────────┐
                │   Patient    │
                │  id: 132...  │
                └──────┬───────┘
       ┌───────────────┼───────────────┬────────────────┐
       │               │               │                │
       ▼               ▼               ▼                ▼
┌─────────────┐ ┌──────────────┐ ┌──────────────┐ ┌─────────────┐
│ Allergy     │ │ Allergy      │ │ Medication   │ │ Condition   │
│ Intolerance │ │ Intolerance  │ │ Request      │ │             │
│ Peanuts     │ │ Penicillin   │ │ Lisinopril   │ │ Hypertension│
│ severe      │ │ moderate     │ │ active       │ │ active      │
└─────────────┘ └──────────────┘ └──────────────┘ └─────────────┘
   patient →       patient →      subject →         subject →
   Patient/132     Patient/132    Patient/132       Patient/132
```

Every dependent resource references the Patient by URI (`Patient/{id}`), not by embedded payload — the FHIR normalization principle.

---

## Design decisions (and the tradeoffs)

### Vanilla JS over React/Vue
A framework would obscure the focus of the demo — which is the FHIR API, not modern frontend tooling. One HTML file means anyone reading the code sees every fetch, every render, every state transition in 450 lines. Easier to evaluate as a portfolio piece.

### Public HAPI sandbox over self-hosted server
Self-hosting HAPI is straightforward but irrelevant to the demo. The point is to show the client-side FHIR pattern; the backend is a swappable detail. The same code points at a private HAPI deployment with a one-line URL change.

### Sequential POSTs during seeding
First version fired all 7 seed POSTs in parallel via `Promise.all`. The public sandbox rate-limits CORS preflights and returned `429 Too Many Requests` on the 7th request, so 6 of 7 resources landed. Switched to a `for...of` loop — slightly slower (~6s vs ~3s) but reliable. A self-hosted server wouldn't have this constraint.

### Parallel GETs during profile load
The four read queries (Patient, AllergyIntolerance, MedicationRequest, Condition) are independent and fire concurrently. Patient profile loads in one round-trip latency, not four. Same data, ~4× faster.

### Coded terminologies in seed data
Allergies use SNOMED CT (`227493005` for Peanuts), medications use RxNorm (`314076` for Lisinopril 10mg), and conditions use SNOMED CT (`38341003` for Hypertension). Real FHIR data lives or dies by terminology bindings — the demo seeds realistic codes rather than free-text strings so the resources validate against US Core profiles.

### Duplicate-detection workaround
The HAPI public sandbox refuses to create resources that match existing patient identity (HTTP 412). The seed function appends a 6-digit timestamp suffix to the patient's family name so repeat clicks always succeed.

---

## How to run locally

```bash
git clone https://github.com/jadayelracha-code/pharmacy-fhir-demo.git
cd pharmacy-fhir-demo
python3 -m http.server 8765
# open http://localhost:8765
```

No `npm install`, no build step, no environment variables. Open the file directly in a browser if you prefer — `file://` works for GETs, but the sandbox blocks POSTs without an `http://` origin, so the seed button needs the local server.

---

## What I'd build next

Roadmap if this were going beyond a portfolio piece:

- **Drug interaction warnings** — cross-reference active medications via the OpenFDA / RxNav API; flag pairs flagged in `drug_interactions` tables
- **Patient timeline view** — `GET /Encounter?patient=...` to show clinical events chronologically; `_revinclude` to pull related Observations
- **SMART on FHIR authorization** — replace the open sandbox with an OAuth flow against a SMART-enabled sandbox (e.g. SMART Health IT, Logica)
- **Write workflows** — "mark medication as dispensed" → `POST /MedicationDispense` linked to the original `MedicationRequest`
- **Resource validation feedback** — surface `OperationOutcome` details from the server when a POST fails, instead of generic error messages
- **Offline-first** — IndexedDB cache + background sync, for low-connectivity pharmacy environments

---

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | Vanilla JS + HTML + CSS, no framework |
| HTTP | Native `fetch` API, `Accept: application/fhir+json` |
| Backend | Public HAPI FHIR R4 server (replaceable) |
| Auth | None (sandbox is open) — SMART on FHIR is the production path |
| Hosting | Static file, no server required |

---

## About this repo

Built by [@jadayelracha-code](https://github.com/jadayelracha-code) as a hands-on FHIR demo. Not affiliated with HAPI FHIR or HL7. The HAPI sandbox is a shared community resource — please don't seed thousands of test patients.
