---
layout: default
title: HMIS CSV Fixer for VA FY2026
---

# HMIS CSV Fixer for VA FY2026

Python tool that fixes HMIS CSV exports to match VA FY2026 format requirements for upload to Volunteers of America North Louisiana (26-AR-470) VA HMIS system.

## Purpose

Transforms standard HMIS FY2026 CSV exports to match the specific format required by the VA's data collection system.

## What This Does

### Client.csv Transformations (46 → 38 columns)

- **Sex Field**: Consolidates Woman/Man/NonBinary/etc. into single Sex field (0=Woman, 1=Man, 99=Not collected)
- **Race Fields**: Adds HispanicLatinao and MidEastNAfrican (both set to 0)
- **RaceNone Logic**: Sets to 99 when any race selected (VA doesn't accept 0)
- **White Field**: Cannot be blank, defaults to 0
- **Veteran Years**: Validates 1920-2021 range only
- **Removes**: Woman, Man, NonBinary, CulturallySpecific, Transgender, Questioning, DifferentIdentity, GenderNone, DifferentIdentityText

### Enrollment.csv Transformations (70 → 64 columns)

- **Removes**: SexualOrientationOther, AlcoholDrugUseDisorderFam, PreferredLanguageDifferent
- These are HMIS FY2026 fields that VA's spec doesn't include

### Other Files

- All other CSVs copied unchanged
- ExportID automatically read from Export.csv and applied to Client.csv and Enrollment.csv

## Usage

### Fix New Export

```powershell
# Extract new zip to test##_extracted folder
Expand-Archive -Path new_export.zip -DestinationPath test##_extracted

# Run fix script
python _backup/fix_hmis_csv.py test##_extracted test##_fixed

# Create upload zip
Compress-Archive -Path test##_fixed\* -DestinationPath test##.zip
```

### Key Files

- `_backup/fix_hmis_csv.py` - Main transformation script
- `_backup/validate_hmis.py` - Pre-upload validator
- `_backup/TRANSFORMATION_GUIDE.md` - Detailed field mappings for Power Query

## Success Metrics

- **test37.zip**: Uploaded successfully with 0 errors
- **Data Quality Score**: 94.4% overall, 100% on critical fields
- **VA Response**: "No issues found. The upload was successful and the data has been accepted."

## Power Query M Code

All M queries have been updated to:

1. Promote headers BEFORE setting column types
2. Apply VA FY2026 transformations (Client.csv and Enrollment.csv only)
3. Match exact column names from CSV files

Files reviewed:

- ✅ Client.csv - VA FY2026 transformations applied
- ✅ Enrollment.csv - Removed 3 FY2026 fields not in VA spec
- ✅ Exit.csv - Fixed header promotion
- ✅ Export.csv - No changes needed
- ✅ Funder.csv - Fixed header promotion
- ✅ HealthAndDV.csv - Fixed header promotion
- ✅ IncomeBenefits.csv - Fixed header promotion
- ✅ Project.csv - Fixed header promotion
- ✅ ProjectCoC.csv - Fixed header promotion
- ✅ Services.csv - Fixed header promotion
- ✅ EmploymentEducation.csv - No changes needed
- ✅ Disabilities.csv - No changes needed

## Important Notes

- ExportID must match Export.csv (script handles this automatically)
- CSVVersion in Export.csv should be "2024 v1.10" or similar
- Client.csv MUST have exactly 38 columns for VA acceptance
- Enrollment.csv MUST have exactly 64 columns for VA acceptance
- RaceNone value of "0" will fail VA validation

## Test History

- test29-test36: Various validation errors fixed
- test37: ✅ SUCCESS - 0 errors
- test38: ✅ Fixed with correct ExportID from Export.csv
