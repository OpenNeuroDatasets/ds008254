# BIDS Neuroimaging Dataset

## Overview

This directory is the root of a BIDS-formatted neuroimaging dataset comprising **43 participants** (`sub-110` through `sub-152`). The dataset includes structural (T1w) and functional (BOLD) MRI, organized per subject following the [Brain Imaging Data Structure (BIDS)](https://bids.neuroimaging.io/) specification.

Raw DICOM data were converted to BIDS using **BIDScoin 4.6.1**. T1-weighted anatomical images have additionally been defaced and de-identified for sharing (see below).

Root-level files:

| File | Description |
|------|-------------|
| `dataset_description.json` | Dataset-level metadata (conversion, license, authors, funding, etc.) |
| `participants.tsv` / `participants.json` | Subject-level phenotypic information |
| `README` | This document |

## Directory Structure

Each subject follows standard BIDS layout. Example for `sub-152`:

```
sub-152/
  sub-152_scans.tsv
  anat/
    sub-152_acq-iso1mm_T1w.nii.gz
    sub-152_acq-iso1mm_T1w.json
    sub-152_acq-iso1mm_desc-defacemask_T1w.nii.gz    (optional)
  func/
    sub-152_task-flexibilitystability_run-1_bold.json
    sub-152_task-PA_dir-PA_bold.json
    ...
```

Functional BOLD NIfTI files (`.nii.gz`) for all subjects follow the same BIDS naming convention and may be merged into each subject's `func/` folder during dataset curation.

## Modalities

### Anatomical (`anat/`)

| Suffix | Description |
|--------|-------------|
| `T1w` | 3D T1-weighted MPRAGE, 1 mm isotropic (`t1_mprage_sag_iso_TI1000`), Siemens Prisma 3T |
| `desc-defacemask` | Binary mask documenting voxels removed during defacing (optional) |

### Functional (`func/`)

| Task / scan | Description |
|-------------|-------------|
| `task-flexibilitystability` | Task-based BOLD (e.g. `run-1`) |
| `task-PA` | Phase-encoding reference / fieldmap-related EPI (PA direction) |

Typical BOLD acquisition parameters (from sidecar JSON): TR = 1 s, TE = 30 ms, multiband factor = 5, 2 mm slice thickness, phase-encoding direction `j-` (AP).

Associated `_events.tsv` and `_events.json` files may be present for task runs where applicable.

## Defacing (Anatomical T1w Only)

Defacing applies **only** to T1w images. BOLD data are not defaced.

- **Method**: pydeface standard template and facemask, with affine registration via SimpleITK.
- **Effect**: Facial voxels are set to zero in `*_T1w.nii.gz`.
- **Mask file**: `*_desc-defacemask_T1w.nii.gz` records where defacing was applied (1 = retained, 0 = removed). It is **not** the original non-defaced image.

## JSON Sidecar De-identification

The following fields were removed from T1w and BOLD JSON sidecars:

- `InstitutionName`, `InstitutionalDepartmentName`, `InstitutionAddress`
- `DeviceSerialNumber`, `StationName`
- `AcquisitionTime`, `ProcedureStepDescription`, `StudyDescription`
- `ShimSetting`, `ConsistencyInfo`, `TxRefAmp`

Each de-identified sidecar includes `"Deidentified": true`. Defaced T1w sidecars additionally include `"Defaced": true`.



## Dataset Description Files

- **`dataset_description.json`**: Primary dataset metadata for the full BIDS collection (anatomical, functional, and other modalities). `GeneratedBy` lists both BIDScoin conversion and the deface pipeline.

## Processing Provenance

| Step | Tool | Location |
|------|------|----------|
| DICOM → BIDS | BIDScoin 4.6.1 | `./code/bidscoin` (if present) |
| T1w deface + de-identification | pydeface template + SimpleITK | `../task-deface-pipeline/scripts/` |
| BOLD JSON de-identification | custom sidecar scrubber | `../task-deface-pipeline/scripts/` |
| Processing log | — | `../task-deface-pipeline/logs/processing_report.csv` |
