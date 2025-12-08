# LAMBDA Facility Search API Specification v0.1.0

## Overview
Each participating facility exposes a standard REST API for experiment discovery via metadata search. This specification covers ONLY the search/discovery layer. Data transfer, mapping, and access control are separate concerns addressed in other specifications.

## Base URL
Each facility chooses their base URL. Examples:
- `https://als-lambda.lbl.gov/api/v1`
- `https://slac-lambda.stanford.edu/api/v1`

## Content Type
All endpoints MUST accept and return `application/json` for both success and error responses.

## Forward Compatibility
Clients MUST ignore unknown top-level fields and unknown object fields in all responses. This allows the API to evolve without breaking existing clients.

## Error Format
Error responses SHOULD follow a common structure:
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Invalid technique 'cryo-maybe'"
}
```

---

## Endpoints

### 1. Search Experiments
**`GET /search`**

Query experiments across metadata fields.

**Query Parameters** (all optional, combined with AND logic):
- `seguid` - string, comma-separated for complexes (e.g., `abc123` or `abc123,def456`). Whitespace around comma-separated values MUST be ignored.
- `protein_name` - string, case-insensitive substring match
- `technique` - string, must match controlled_vocabularies.techniques from the LAMBDA contract
- `facility` - string, must match controlled_vocabularies.facilities from the LAMBDA contract
- `instrument` - string, case-insensitive substring match (free text)
- `date_start` - ISO 8601 datetime (inclusive). Must use YYYY-MM-DDTHH:MM:SSZ format (UTC, trailing Z, no timezone offsets) matching LAMBDA contract
- `date_end` - ISO 8601 datetime (inclusive). Must use YYYY-MM-DDTHH:MM:SSZ format (UTC, trailing Z, no timezone offsets) matching LAMBDA contract

**Matching Rules:**
All provided filters are combined with logical AND.
- `protein_name`: case-insensitive substring match
- `facility`, `technique`: exact match against controlled vocabulary
- `instrument`: case-insensitive substring match (free text)
- `seguid`: Request `seguid=seg1,seg2` means "experiment has all of these SEGUIDs"
- `date_start`, `date_end`: 
  - If only `date_start` is provided: return experiments with `experiment_info.date >= date_start`
  - If only `date_end` is provided: return experiments with `experiment_info.date <= date_end`
  - If both are provided: return experiments with `date_start <= experiment_info.date <= date_end`

**Response: 200 OK**
```json
{
  "results": [
    {
      "experiment_id": "550e8400-e29b-41d4-a716-446655440000",
      "contract_version": "0.1.0",
      "facility": "ALS",
      "facility_endpoint": "https://als-lambda.lbl.gov/api/v1",
      "seguid": ["abc123def456", "789ghi012jkl"],
      "protein_name": "lysozyme-nanobody complex",
      "technique": "cryo-ET",
      "instrument": "Titan Krios",
      "date": "2025-03-15T14:30:00Z",
      "PI": {
        "name": "Jane Smith",
        "institute": "UC Berkeley",
        "orcid": "0000-0002-1825-0097"
      }
    }
  ],
  "count": 1
}
```

**Response Field Definitions:**
- `results`: Array of matching experiments. All objects in the array MUST have the same structure (homogeneous). Ordering is UNSPECIFIED in version 0.1.x. Clients MUST NOT assume sorted results. Future versions MAY introduce explicit sort options.
- `count`: Integer. Number of experiments returned in the current `results` array. In version 0.1.x this is also the total number of matching experiments, because pagination is not implemented. MUST be a JSON integer, not a string.

**Empty Result Sets:**
When no experiments match, the API MUST return `results: []` and `count: 0`.

**Field Notes:**
- `experiment_id`: UUID from experiment_info.experiment_id in the LAMBDA Data Organization Contract
- `contract_version`: Version string from experiment_info.contract_version (allows client to verify compatibility)
- `facility`: Must match controlled_vocabularies.facilities from the LAMBDA contract
- `technique`: Must match controlled_vocabularies.techniques from the LAMBDA contract
- `seguid`: Array of SEGUID strings. Omit field entirely if no SEGUID exists for this experiment. Never emit `[null]`.
- `date`: Timestamp in YYYY-MM-DDTHH:MM:SSZ format (UTC, trailing Z, no timezone offsets) matching LAMBDA contract
- `facility_endpoint`: Full base URL of this facility's API (enables routing for multi-facility queries)
- Other metadata fields should match `experiment_info.json` from LAMBDA contract where applicable

**Error Responses:**
- `400 Bad Request` - Invalid query parameters (e.g., malformed date, unknown technique). Facilities MUST reject malformed parameters with `400 Bad Request` rather than silently coercing or ignoring them.
- `500 Internal Server Error` - Facility-side failure

Error responses MUST be returned with HTTP status code and JSON body using the standard error format.

**Pagination:**
Version 0.1.x response does not include pagination fields. Future versions MAY add `next_token` and/or `total_count`.

---

### 2. Health Check
**`GET /health`**

Check facility API status. In version 0.1.x, `/health` MUST be publicly accessible without authentication.

**Response: 200 OK**
```json
{
  "status": "healthy",
  "facility": "ALS",
  "api_version": "0.1.0",
  "contract_version": "0.1.0",
  "seguid_algorithm": "IUPAC_SEGUID_v1"
}
```

**Field Notes:**
- `api_version`: Semantic version of this API. The path prefix /api/v1 denotes the major version. Minor/patch bumps within v1 SHOULD remain backwards compatible.
- `seguid_algorithm`: (Optional) SEGUID algorithm and version used by this facility. Facilities SHOULD advertise this for reproducibility.

**Error Responses:**
- `503 Service Unavailable` - Facility experiencing issues

Error responses MUST be returned with HTTP status code and JSON body using the standard error format.

---

## Implementation Notes

**Version 0.1.x Scope:**
- Search/discovery functionality only
- Metadata-based queries across federated facilities
- No authentication/authorization specified (facilities may implement independently)
- No pagination (add in future versions if needed)
- No create/update/delete operations (experiments registered out-of-band)

**Out of Scope (separate specifications):**
- Data transfer initiation and execution
- Transfer status monitoring
- Facility-to-LAMBDA contract mapping/translation
- Access control and authentication for data transfer

**Future Evolution (search API only):**
- Authentication schemes for search API
- Pagination for large result sets
- Advanced query operators (ranges, wildcards, OR logic)
- **Instrument whitelisting**: In version 0.1.x, `instrument` is free text with substring matching. Future versions should introduce a controlled vocabulary of registered equipment/instrument names (similar to `facility` and `technique` in the LAMBDA contract) to enable precise federation searches across facilities.

**SEGUID Calculation:**
- SEGUID values MUST be computed with a facility-independent SEGUID algorithm
- The specific algorithm and version SHOULD be advertised via `/health` endpoint (e.g., `seguid_algorithm: "IUPAC_SEGUID_v1"`)
- For protein-DNA complexes: array contains SEGUID for each component
- Order should be documented (e.g., alphabetical, or chain order)
- If no SEGUID exists for an experiment, omit the `seguid` field entirely from responses (never emit `[null]`)

---

**Status:** Draft v0.1.0 (Search/Discovery API only)  
**Last Updated:** 2025-12-05  
**Note:** Data transfer, mapping, and access specifications are separate documents (to be developed)
