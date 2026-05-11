# Pharmacy Patient Profile — HAPI FHIR Demo

A single-file vanilla-JS web app that demonstrates a pharmacy-style patient profile view backed by the [HAPI FHIR](https://hapifhir.io/) REST API.

Built as a hands-on intro to FHIR concepts: resources, server-assigned IDs, references between resources, and search.

## What it does

- Search patients by name
- Show a profile with:
  - Demographics (name, gender, DOB, phone, email)
  - Allergies (with severity badges)
  - Active medications (with dosage instructions)
  - Conditions (with clinical status)
- Seed button that creates a fully-loaded demo patient (1 Patient + 2 AllergyIntolerance + 2 MedicationRequest + 2 Condition resources)

## How to run

Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8765
# then open http://localhost:8765
```

## Stack

- One HTML file, no build step, no dependencies
- Talks to the public HAPI sandbox at `https://hapi.fhir.org/baseR4`
- Vanilla JS `fetch()` for all FHIR REST calls

## FHIR resources used

| Resource | Endpoint |
|---|---|
| Patient | `GET /Patient?name=...` · `POST /Patient` · `GET /Patient/{id}` |
| AllergyIntolerance | `GET /AllergyIntolerance?patient=Patient/{id}` · `POST /AllergyIntolerance` |
| MedicationRequest | `GET /MedicationRequest?subject=Patient/{id}&status=active` · `POST /MedicationRequest` |
| Condition | `GET /Condition?subject=Patient/{id}` · `POST /Condition` |
