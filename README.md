# LAMBDA: Lakehouse-enabled AI-ready Multi-modal Bioimaging Data Architecture

**Version 0.1.0 - Pilot Phase**

LAMBDA is a federated data architecture for discovering and accessing bioimaging and scattering experiments across DOE user facilities. It enables cross-facility data integration without requiring facilities to change their internal systems or commit to specific technology stacks.

## Quick Links

- 📄 [**Data Organization Contract v0.1.0**](contract_v0.1.0.json) - Structure and metadata requirements
- 🔍 [**Facility Search API v0.1.0**](facility_search_api_v0.1.0.md) - Discovery interface specification
- 📖 [**Strategic Vision**](#strategic-vision) - Why LAMBDA and what it achieves

## What is LAMBDA?

LAMBDA addresses a fundamental challenge: valuable scientific data from bioimaging and scattering experiments remains siloed within individual DOE facilities. Researchers can't easily discover experiments performed elsewhere, compare datasets across instruments, or build comprehensive training sets for machine learning.

Rather than mandating infrastructure changes, LAMBDA defines:

1. **A common data organization contract** - How experimental data should be structured when delivered to consumers
2. **A federated search API** - How facilities expose experiment metadata for discovery
3. **Clear separation of concerns** - Search, organization, and transfer are independent layers

## Core Principles

### Minimal & Practical
- Lightweight specifications that can be implemented quickly
- Focus on essentials: discovery and data organization
- Defer complex problems (authentication, transfer protocols, workflow validation)

### Federated & Autonomous
- No central data repository
- Facilities maintain complete control over internal systems
- Thin compatibility layer over existing infrastructure

### Technology Agnostic
- Works with Globus, Tiled, datafed, oncat, HTTP, rsync
- Adapts to emerging technologies (BER Lakehouse, American Science Cloud)
- Interfaces, not implementations

## The Three Specifications

### 1. Data Organization Contract ([contract_v0.1.0.json](contract_v0.1.0.json))

Defines the structure of a LAMBDA-compliant experiment package:

```
<experiment_id>/
  experiment_info.json          # Experiment metadata
  raw_data/
    raw_data_info.json          # Data unit manifest
    unit_1/                     # Raw experimental data
    unit_2/
  products/
    product_info.json           # Derived data manifest
    product_1/
      workflow.json             # Provenance metadata
      <output files>
  raw_metadata/
    raw_metadata_info.json      # Auxiliary metadata manifest
    <calibrations, logs, etc>
```

**Key features:**
- UUID-based experiment identification
- Complete provenance tracking (who, what, when, where, how)
- SHA256 checksums for all files
- Extensible via `_extensions_` wrapper
- JSON Schema validation throughout

### 2. Facility Search API ([facility_search_api_v0.1.0.md](facility_search_api_v0.1.0.md))

Defines endpoints that every facility must expose:

**`GET /search`** - Query experiments by:
- SEGUID (sequence identifier)
- Protein/sample name
- Technique (cryo-ET, SAXS, XRD, etc.)
- Facility, instrument, date range

**`GET /health`** - API status and capability advertisement

**Returns:** Metadata for discovery (NOT data transfer)

### 3. Data Transfer API ([data_transfer_api_v0.1.0.md](data_transfer_api_v0.1.0.md))

TBD

## Who Should Use LAMBDA?

### DOE User Facilities
Implement the Search API and produce contract-compliant data packages. Gain:
- Increased visibility and impact of facility data
- Cross-facility collaboration opportunities
- Minimal implementation burden
- No forced infrastructure changes

### Researchers & Data Consumers
Search across facilities, access consistently-organized data. Gain:
- Discover experiments across DOE complex
- Build multi-facility datasets
- Use same tools regardless of data origin
- Accelerate AI/ML model development

### Tool Developers
Build once, work everywhere. Gain:
- Consistent input data structure
- Predictable metadata schemas
- Clear provenance for reproducibility
- Growing user base across facilities

## Strategic Vision

LAMBDA v0.1.x is explicitly a **pilot phase**, not a final architecture. The goal is to:

1. **Validate assumptions** through real implementations
2. **Discover pain points** before they become embedded
3. **Build evidence** for v1.0 decisions based on practice
4. **Enable early science** while infrastructure matures

### Balancing Act

LAMBDA navigates between:
- **Current systems** (datafed, oncat, Globus, facility-specific) ← works with these today
- **Emerging standards** (BER Lakehouse, American Science Cloud) ← adapts as these clarify
- **Future technologies** (not yet known) ← stays flexible

We don't predict which technologies will dominate. We provide interfaces that can wrap whatever emerges.

## Implementation Guide

### For Facilities

TBD

### For Consumers

TDB

## Repository Structure

```
lambda-specs/
├── README.md                           # This file
├── contract_v0.1.0.json               # Data organization contract
├── facility_search_api_v0.1.0.md      # Search API specification
├── examples/                           # Example conformant datasets
│   ├── cryo-et-example/
│   ├── saxs-example/
│   └── xrd-example/
├── validators/                         # Contract validation tools
│   └── (to be developed)
└── docs/
    ├── strategic-vision.md            # Full strategic context
    ├── pilot-plan.md                  # Pilot phase roadmap
    └── faq.md                         # Frequently asked questions
```

## Current Status

**Contract:** v0.1.0 - Ready for comments and feedback  
**Search API:** v0.1.0 - Ready for comments and feedback  
**Pilot Facilities:** Recruiting   
**Reference Implementation:** Will commence after contract and API are frozen.  

