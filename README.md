# KaiProva Animal Data Specification

> Standard data formats for animal registration, weigh records, and movement tracking across the KaiProva verified low-carbon beef platform.

## Overview

KaiProva requires per-animal data from birth through processing to generate verified carbon footprints. This specification defines the CSV upload formats for farmers, rearers, and finishers enrolling animals onto the platform.

The specification supports both **New Zealand (NAIT)** and **Australian (NLIS)** traceability systems.

## Quick Start

1. Download the appropriate CSV template for your country
2. Fill in your animal data (one row per animal)
3. Upload via the KaiProva farmer portal at `app.kaiprova.com`
4. Animals are automatically grouped into mobs based on birth date, breed, and sex

## CSV Templates

| Template | Country | Description |
|----------|---------|-------------|
| [`animal_registration_nz.csv`](templates/animal_registration_nz.csv) | New Zealand | NAIT-tagged animals |
| [`animal_registration_au.csv`](templates/animal_registration_au.csv) | Australia | NLIS-tagged animals |
| [`weigh_record.csv`](templates/weigh_record.csv) | NZ / AU | Periodic weight records |
| [`movement_event.csv`](templates/movement_event.csv) | NZ / AU | NAIT/NLIS movement events |

## Animal Registration Format

### Required Fields

| Field | Type | Description | Example (NZ) | Example (AU) |
|-------|------|-------------|--------------|--------------|
| `eid` | string | Electronic ID (RFID tag) | `982 123807241049` | `982 012345678901` |
| `vid` | string | Visual ID (ear tag number) | `23523` | `AU4412-0023` |
| `sex` | enum | Animal sex | `bull` | `heifer` |
| `breed` | string | Breed or cross description | `Friesian x Hereford` | `Jersey x Angus` |
| `birth_date` | ISO 8601 | Date of birth | `2025-06-15` | `2025-07-10` |
| `birth_weight_kg` | decimal | Live weight at birth (kg) | `39.5` | `38.0` |
| `birth_farm_id` | string | NAIT number or NLIS PIC of birth property | `47-0088` | `3ABCD1234` |

### Optional Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `dam_eid` | string | Dam's electronic ID (if known) | `982 123800001234` |
| `sire_breed` | string | Sire breed (if known) | `Hereford` |
| `birth_weight_method` | enum | How birth weight was obtained | `scale` / `estimate` |
| `collection_date` | ISO 8601 | Date collected from birth farm | `2025-07-03` |
| `collection_weight_kg` | decimal | Weight at collection (kg) | `42.0` |
| `notes` | string | Free text notes | `Twin calf, smaller of pair` |

### EID Format

| Country | Format | Example | Notes |
|---------|--------|---------|-------|
| New Zealand | `982 XXXXXXXXXXXX` | `982 123807241049` | 15-digit RFID. Space after country code `982` |
| Australia | `982 XXXXXXXXXXXX` | `982 012345678901` | Same ISO 11784/11785 standard |

### Sex Values

| Value | Description |
|-------|-------------|
| `bull` | Intact male |
| `steer` | Castrated male |
| `heifer` | Female (not calved) |

### Traceability ID Format

| Country | System | ID Format | Example | Description |
|---------|--------|-----------|---------|-------------|
| New Zealand | NAIT | `XX-XXXX` | `47-1234` | NAIT location number |
| Australia | NLIS | Alphanumeric PIC | `3ABCD1234` | Property Identification Code |

## Weigh Record Format

Periodic weight records are the foundation of KaiProva's carbon verification. The LCA engine uses weight trajectory data to calculate per-animal emissions with validated accuracy.

**Recommended frequency:** Monthly weighing produces the most accurate carbon footprints. Minimum two data points required (birth weight + one subsequent weigh).

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `eid` | string | Yes | Electronic ID matching registration | `982 123807241049` |
| `weigh_date` | ISO 8601 | Yes | Date of weighing | `2025-09-15` |
| `live_weight_kg` | decimal | Yes | Live weight in kilograms | `185.5` |
| `location_id` | string | Yes | NAIT number or NLIS PIC where weighed | `47-1234` |
| `method` | enum | No | Weighing method | `platform_scale` / `walk_over` / `estimate` |
| `condition_score` | decimal | No | Body condition score (1–10 scale) | `5.5` |
| `notes` | string | No | Free text | `Post-drench weigh` |

## Movement Event Format

Movement events track NAIT/NLIS transfers between properties. These are critical for chain-of-custody verification.

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `eid` | string | Yes | Electronic ID | `982 123807241049` |
| `event_date` | ISO 8601 | Yes | Date of movement | `2025-07-03` |
| `event_type` | enum | Yes | Type of movement | `transfer_in` / `transfer_out` |
| `from_location_id` | string | Yes | Sending property NAIT/NLIS ID | `47-0088` |
| `to_location_id` | string | Yes | Receiving property NAIT/NLIS ID | `47-1234` |
| `live_weight_kg` | decimal | No | Weight at movement (kg) | `42.0` |
| `notes` | string | No | Free text | `Weaned, collected by rearer` |

## Mob Grouping

KaiProva automatically groups individual animals into **mobs** based on:

- Birth date (within a 30-day window)
- Sex (bulls, steers, heifers grouped separately)
- Current location (same property)
- Breed composition (similar crosses grouped together)

Farmers can also manually assign mob names during upload. Mobs are the primary unit for contract matching, supply forecasting, and carbon certification.

## Validation Rules

The platform validates all uploads against these rules:

| Rule | Description |
|------|-------------|
| **Unique EID** | Each electronic ID must be unique across the platform |
| **Valid date range** | Birth date must be within the last 24 months |
| **Weight bounds** | Birth weight: 25–55 kg. Subsequent weights must be ≥ birth weight |
| **Known location** | NAIT/NLIS location IDs must exist in the national register |
| **Required fields** | All required fields must be present and non-empty |
| **Country consistency** | All animals in a single upload must be from the same country |

Validation errors are returned per-row with specific field references, allowing partial uploads where valid rows are accepted and invalid rows are flagged for correction.

## Integration Notes

### For software developers

The KaiProva API accepts these same formats via JSON. CSV upload is the farmer-facing interface; API integration is available for farm management software, dairy company systems, and processor platforms.

```
POST /api/v1/animals/register
Content-Type: application/json

{
  "country": "NZ",
  "farm_id": "47-1234",
  "animals": [
    {
      "eid": "982 123807241049",
      "vid": "23523",
      "sex": "bull",
      "breed": "Friesian x Hereford",
      "birth_date": "2025-06-15",
      "birth_weight_kg": 39.5,
      "birth_farm_id": "47-0088"
    }
  ]
}
```

### NAIT/NLIS API Integration

KaiProva is pursuing OSPRI Information Provider accreditation (NZ) and NLIS database integration (AU) to enable automated movement verification. Until accreditation is complete, movement events are validated against farmer-uploaded data.

## Data Privacy

- All animal data is owned by the enrolling farmer
- Farm identity is never visible to processors until a supply match is accepted
- Processor capacity data is encrypted and invisible to KaiProva platform operators
- Data is stored in compliance with NZ Privacy Act 2020 and Australian Privacy Act 1988

## Licence

This specification is published under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You are free to use, adapt, and build upon this specification with attribution to KaiProva / Alps2Ocean Foods NZ Tapui Ltd.

---

**KaiProva** — The verified low-carbon beef platform. Every footprint followed.

© 2026 Alps2Ocean Foods NZ Tapui Ltd

