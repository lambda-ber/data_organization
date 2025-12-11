# LAMBDA Experiment Directory Structure Example

# Sample LAMBDA Experiment Directory Structure

This is a complete example of a LAMBDA-compliant experiment directory for a cryo-ET study at ALS beamline 8.3.1.

## Directory Tree

```
als_bl8_3_1_20250315_446655440000_lysozyme/
│
├── experiment_info.json                           # Top-level experiment metadata
│
├── raw_data/                                       # Raw experimental data
│   ├── raw_data_info.json                         # Manifest of all data units
│   │
│   ├── unit_1/                                    # First data unit (grid position A1)
│   │   ├── tilt_series_###.mrc                    # Serial files: tilt_series_001.mrc through tilt_series_061.mrc
│   │   │                                          # (61 files, -60° to +60° in 2° steps)
│   │   ├── tilt_angles.txt                        # Tilt angle metadata
│   │   ├── acquisition_metadata.json              # Detailed acquisition parameters
│   │   └── checksums.sha256                       # SHA256 checksums for tilt series
│   │
│   ├── unit_2/                                    # Second data unit (grid position A2)
│   │   ├── tilt_series_###.mrc                    # tilt_series_001.mrc through tilt_series_041.mrc
│   │   │                                          # (41 files, -40° to +40° in 2° steps - shorter range)
│   │   ├── tilt_angles.txt
│   │   ├── acquisition_metadata.json
│   │   └── checksums.sha256
│   │
│   ├── unit_3/                                    # Third data unit (grid position B1)
│   │   ├── tilt_series_001.mrc
│   │   ├── tilt_series_002.mrc
│   │   ├── ... (files 003-058 omitted for brevity)
│   │   ├── tilt_series_059.mrc
│   │   ├── tilt_series_060.mrc
│   │   ├── tilt_series_061.mrc
│   │   ├── tilt_angles.txt
│   │   └── acquisition_metadata.json
│   │
│   └── unit_4/                                    # Fourth data unit (references external data)
│       └── external_reference.txt                 # Optional: human-readable note about reference
│                                                  # (Directory mostly empty - data is in another experiment)
│
├── products/                                       # Derived data products
│   ├── product_info.json                          # Manifest of all products
│   │
│   ├── product_1/                                 # IMOD preprocessing (unit_1)
│   │   ├── workflow.json                          # Provenance metadata for this workflow
│   │   ├── aligned_stack.mrc                      # Aligned tilt series
│   │   ├── fiducial_model.fid                     # Fiducial marker model
│   │   ├── alignment_log.txt                      # IMOD alignment log
│   │   └── transform.xf                           # Transformation parameters
│   │
│   ├── product_2/                                 # IMOD reconstruction (unit_1)
│   │   ├── workflow.json                          # Provenance (inputs from product_1)
│   │   ├── tomogram.mrc                           # Reconstructed tomogram
│   │   ├── reconstruction_log.txt                 # IMOD reconstruction log
│   │   └── sirt_iterations.txt                    # SIRT iteration parameters
│   │
│   ├── product_3/                                 # IMOD preprocessing (unit_2)
│   │   ├── workflow.json
│   │   ├── aligned_stack.mrc
│   │   ├── fiducial_model.fid
│   │   ├── alignment_log.txt
│   │   └── transform.xf
│   │
│   ├── product_4/                                 # IMOD reconstruction (unit_2)
│   │   ├── workflow.json
│   │   ├── tomogram.mrc
│   │   ├── reconstruction_log.txt
│   │   └── sirt_iterations.txt
│   │
│   ├── product_5/                                 # AreTomo preprocessing (unit_3)
│   │   ├── workflow.json
│   │   ├── aligned_stack.mrc
│   │   ├── aretomo_log.txt
│   │   └── alignment_params.json
│   │
│   └── product_6/                                 # AreTomo reconstruction (unit_3)
│       ├── workflow.json
│       ├── tomogram.mrc
│       ├── tomogram_even.mrc                      # Half-map for resolution estimation
│       ├── tomogram_odd.mrc                       # Half-map for resolution estimation
│       └── reconstruction_log.txt
│
└── raw_metadata/                                   # Auxiliary metadata
    ├── raw_metadata_info.json                     # Manifest of metadata files
    ├── microscope_calibration.json                # Pixel size, defocus, beam parameters
    ├── detector_dark_reference.mrc                # Dark reference image from detector
    ├── sample_preparation_protocol.pdf            # Sample prep documentation
    └── environmental_log.csv                      # Temperature, humidity during collection
```

## Key Features Demonstrated

### 1. Structured Root Directory Name
- **Format**: `facility_instrument_YYYYMMDD_uuidlast_samplename`
- **Example**: `als_bl8_3_1_20250315_446655440000_lysozyme`
  - Facility: `als` (Advanced Light Source)
  - Instrument: `bl8_3_1` (Beamline 8.3.1)
  - Date: `20250315` (March 15, 2025)
  - UUID fragment: `446655440000` (last 12 hex chars of experiment UUID)
  - Sample: `lysozyme`

### 2. Serial File Groups (unit_1, unit_2)
- Pattern-based file representation: `tilt_series_###.mrc`
- Range notation: `001-061` (61 files) or `001-041` (41 files)
- Checksum files for validation without bloating manifest
- Demonstrates 61x and 41x manifest size reduction

### 3. Individual File Listings (unit_3)
- Traditional approach: each file listed individually
- Used when serial pattern doesn't apply or file count is low
- Shows flexibility of contract

### 4. External Data Reference (unit_4)
- Minimal directory content (just a reference note)
- Actual data stored in another experiment
- Demonstrates re-analysis without data duplication
- Directory can be empty, contain symlink, or have reference file

### 5. Workflow Provenance Chain
- **Chain 1**: unit_1 → product_1 (preprocessing) → product_2 (reconstruction)
- **Chain 2**: unit_2 → product_3 (preprocessing) → product_4 (reconstruction)
- **Chain 3**: unit_3 → product_5 (preprocessing) → product_6 (reconstruction)
- Each product has `workflow.json` documenting inputs, software, parameters

### 6. Multiple Processing Approaches
- Units 1-2: IMOD pipeline
- Unit 3: AreTomo pipeline
- Demonstrates flexibility for different processing methods

### 7. Raw Metadata
- Calibration files (JSON format)
- Dark reference images (MRC format)
- Protocol documentation (PDF)
- Environmental logs (CSV)

## File Counts

- **Raw data files**: ~163 files (61 + 41 + 61 tilt series images + metadata)
- **Product files**: 21 output files across 6 products
- **Metadata files**: 4 auxiliary files
- **Manifest files**: 4 required JSON manifests
- **Total**: ~192 files

## Manifest Examples

### experiment_info.json snippet
```json
{
  "contract_version": "0.2.0",
  "experiment_id": "550e8400-e29b-41d4-a716-446655440000",
  "facility_experiment_id": "ALS-2025-001234",
  "experiment_name": "Lysozyme cryo-ET structure determination",
  "facility": {
    "name": "ALS",
    "instrument": "BL8.3.1"
  },
  "technique": "cryo-ET",
  "date": "2025-03-15T14:30:00Z",
  "sample_name": "lysozyme",
  "related_experiments": ["abc12345-6789-abcd-ef01-234567890def"]
}
```

### raw_data_info.json snippet (unit_1 with serial file group)
```json
{
  "units": [
    {
      "id": "unit_1",
      "path": "./unit_1/",
      "description": "Tilt series from grid position A1, -60° to +60° in 2° steps",
      "name": "grid_A1",
      "status": "active",
      "unit_uuid": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
      "files": [
        {
          "file_group": "serial",
          "pattern": "tilt_series_###.mrc",
          "range": "001-061",
          "description": "Tilt series images, 2° increments from -60° to +60°",
          "type": "mrc",
          "mime_type": "application/octet-stream",
          "typical_file_size": 33554432,
          "total_size": 2046820352,
          "checksum_file": "checksums.sha256"
        },
        {
          "filename": "tilt_angles.txt",
          "description": "Tilt angles for each frame",
          "sha256": "a1b2c3d4e5f67890123456789012345678901234567890123456789012345678",
          "file_size": 1024,
          "mime_type": "text/plain"
        },
        {
          "filename": "acquisition_metadata.json",
          "description": "Detailed acquisition parameters and timestamps",
          "sha256": "b2c3d4e5f6789012345678901234567890123456789012345678901234567890",
          "file_size": 8192,
          "mime_type": "application/json"
        }
      ]
    }
  ]
}
```

### raw_data_info.json snippet (unit_4 with external reference)
```json
{
  "units": [
    {
      "id": "unit_4",
      "path": "./unit_4/",
      "description": "High-quality tilt series from previous experiment (referenced)",
      "name": "reference_grid_C5",
      "status": "active",
      "external_data_reference": {
        "source_experiment_id": "abc12345-6789-abcd-ef01-234567890def",
        "source_unit_id": "unit_2",
        "source_facility_path": "/gpfs/als/archive/2025/01/lysozyme_original/raw_data/unit_2/",
        "note": "Using high-quality data from January 2025 collection for comparison"
      }
    }
  ]
}
```

### workflow.json snippet (product_2)
```json
{
  "workflow_run_id": "8d0f7780-8536-51ef-a55c-f18d2f01f9f8",
  "task_name": "IMOD tomographic reconstruction",
  "software": "IMOD",
  "version": "4.12.5",
  "timestamp": "2025-03-15T18:45:30Z",
  "data_input": [
    "products/product_1/aligned_stack.mrc",
    "products/product_1/transform.xf"
  ],
  "input_uuids": {
    "experiment_id": "550e8400-e29b-41d4-a716-446655440000",
    "unit_uuids": ["7c9e6679-7425-40de-944b-e07fc1f90ae7"]
  },
  "run_parameters": {
    "algorithm": "SIRT",
    "iterations": 10,
    "thickness": 1000,
    "binning": 2
  },
  "outputs": [
    {
      "filename": "tomogram.mrc",
      "description": "Reconstructed tomogram",
      "type": "tomogram",
      "schema_id": "cryoet/tomogram/v1",
      "sha256": "c3d4e5f6789012345678901234567890123456789012345678901234567890ab",
      "file_size": 536870912,
      "mime_type": "application/octet-stream"
    },
    {
      "filename": "reconstruction_log.txt",
      "description": "IMOD reconstruction log",
      "type": "log",
      "sha256": "d4e5f6789012345678901234567890123456789012345678901234567890abcd",
      "file_size": 4096,
      "mime_type": "text/plain"
    }
  ]
}
```

## Storage Footprint

- **Raw data**: ~2.5 GB (compressed tilt series)
- **Products**: ~1.5 GB (tomograms + intermediate files)
- **Metadata**: ~50 MB (calibrations, logs, protocols)
- **Total**: ~4 GB

## Workflow Execution Timeline

1. **2025-03-15 14:30:00Z** - Data collection (units 1-3)
2. **2025-03-15 16:00:00Z** - IMOD preprocessing (products 1, 3)
3. **2025-03-15 18:45:30Z** - IMOD reconstruction (products 2, 4)
4. **2025-03-15 17:30:00Z** - AreTomo preprocessing (product 5)
5. **2025-03-15 19:15:00Z** - AreTomo reconstruction (product 6)
```

This example demonstrates all major contract features in a realistic cryo-ET experiment scenario!