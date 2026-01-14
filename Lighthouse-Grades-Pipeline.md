# Lighthouse Grades Pipeline

Multi-stage dataflow pipeline for tracking and reporting Lighthouse program student academic performance across quarterly grading periods.

## Pipeline Flow Diagram

```
📁 OneLake Files: ServicePoint/Lighthouse Grades/*.csv
    ↓
📊 raw_lighthouse_grades
    ↓
🔄 lighthouse_grades_transform ← 📊 clean_lighthouse_entries
    ↑                            ← 📊 lighthouse_client_master
    ↓
    ├─→ 📋 1q_lh_grades
    └─→ 📋 2q_lh_grades
```

---

## Pipeline Architecture

### Stage 1: raw_lighthouse_grades
**Source**: OneLake Files → `ServicePoint/Lighthouse Grades/` folder  
**Output**: `raw_ligthouse_grades` (lakehouse table)  
**Update Frequency**: Manual CSV upload  
**Purpose**: Ingest quarterly GPA data for Lighthouse students

**Input File Format** (CSV):
- Last Name
- First Name
- Client ID
- Entry Date
- Provider
- Lighthouse Location Enrolling
- School Year
- 1st Quarter GPA
- 2nd Quarter GPA
- 3rd Quarter GPA

**Transformations**:
- Reads all CSV files from the folder
- Promotes first row to headers
- Type casting: Client ID (integer), Entry Date (date), GPAs (decimal)
- Column renaming to standardized snake_case format

**Output Columns**:
- `client_id` - ServicePoint client identifier
- `first_name`, `last_name` - Student name
- `entry_date` - Program enrollment date
- `program` - Provider/program name
- `lighthouse_location_enrolling` - School location
- `school_year` - Academic year (e.g., "2025-26")
- `1Q_grades`, `2Q_grades`, `3Q_grades` - Quarterly GPA values

**Filtering**:
- Filters to current school year (2025-26)

---

### Stage 2: lighthouse_grades_transform
**Inputs**: 
- `clean_lighthouse_entries` (from attendance pipeline)
- `lighthouse_client_master` (from attendance pipeline)
- `raw_ligthouse_grades` (from Stage 1)

**Output**: Multiple tables for reporting
- `1q_lh_grades` - 1st Quarter performance by location
- `2q_lh_grades` - 2nd Quarter performance by location

**Purpose**: Calculate grade performance metrics and generate location-level summaries

**Transformation Logic**:

1. **Grade Assessment**:
   - Joins `lighthouse_client_master` with `raw_ligthouse_grades` by `client_id`
   - Adds grade columns (1Q, 2Q, 3Q) to student master records
   
2. **Performance Categorization**:
   - Creates `1Q_grade_check` field:
     - "GOOD" if GPA >= 2.5
     - "BAD" if GPA < 2.5
     - "NOGRADE" if GPA is null
   
3. **Counter Creation**:
   - `1q_counter`: 1 if GOOD, 0 otherwise
   - `control`: Always 1 (for total count)
   
4. **Student Filtering**:
   - Removes students with null grades (no data available)
   - Filters to students still active after August 1, 2025

**Output Tables**:

#### 1q_lh_grades
**Columns**:
- `lighthouse_location_enrolling` - School location
- `meeting_target` - Count of students with GPA >= 2.5
- `total_counted` - Total students with grade data
- `Division` - Percentage meeting target (meeting_target / total_counted)

**Enrollment Date Filter**:
- Includes students enrolled between 8/1/2025 and 10/17/2025 (1st quarter period)
- Also includes students enrolled before 8/1/2025 (returning students)

**Aggregation**: Groups by location, sums counters, calculates percentage

---

#### 2q_lh_grades
**Columns**:
- `lighthouse_location_enrolling` - School location
- `meeting_target` - Count of students with GPA >= 2.5
- `total_counted` - Total students with grade data
- `Division` - Percentage meeting target

**Enrollment Date Filter**:
- Includes students enrolled between 10/17/2025 and 12/20/2025 (2nd quarter period)
- Also includes students enrolled before 8/1/2025 (returning students)

**Aggregation**: Groups by location, sums counters, calculates percentage

---

## Academic Performance Threshold

**Target GPA**: 2.5 (C+ average)
- Students with GPA >= 2.5 are considered "meeting target"
- Used as success metric for program effectiveness

---

## Quarter Periods

### 1st Quarter
**Dates**: August 13, 2025 - October 17, 2025  
**Eligibility**: Students enrolled on or before October 17, 2025

### 2nd Quarter
**Dates**: October 20, 2025 - December 17, 2025  
**Eligibility**: Students enrolled on or before December 20, 2025

### 3rd Quarter
**Dates**: January 6, 2026 - March 6, 2026  
**Eligibility**: (To be implemented)

---

## Pipeline Execution Order

1. **Stage 1**: `raw_lighthouse_grades` - Ingest grade data from CSV files
2. **Stage 2**: `lighthouse_grades_transform` - Calculate performance metrics

**Dependencies**: This pipeline requires the Lighthouse Attendance pipeline to have run first to populate `clean_lighthouse_entries` and `lighthouse_client_master`.

---

## How the Numbers Are Calculated
*Plain-language explanation for non-technical readers*

### Stage 1: Import Grade Data
**Process**: Upload quarterly grade reports to OneLake
- **File Location**: ServicePoint folder → Lighthouse Grades subfolder
- **File Format**: Excel or CSV with student grades
- **System reads**: Client ID, student name, enrollment date, school location, and quarterly GPAs
- **Result**: All student grade records stored in the database

### Stage 2: Calculate Grade Performance

#### Step 1: Match Students to Grades
- **Takes**: Student master list from enrollment pipeline
- **Matches**: Each student to their grade records
- **Adds**: 1st, 2nd, and 3rd quarter GPA columns to student profiles

#### Step 2: Check Grade Performance
For each student with a 1st quarter grade:
- **If GPA >= 2.5**: Mark as "GOOD" (meeting target)
- **If GPA < 2.5**: Mark as "BAD" (not meeting target)
- **If no grade**: Mark as "NOGRADE" (excluded from calculations)

#### Step 3: Count Students by Location
For 1st quarter report:
- **Group students** by their school location
- **Count** how many students have grades
- **Count** how many students meet the 2.5 GPA target
- **Calculate percentage**: (Students meeting target) ÷ (Total students with grades)

**Example**: 
- Broadmoor STEM has 25 students with 1st quarter grades
- 18 of them have GPA >= 2.5
- Performance rate: 18 ÷ 25 = 72%

#### Step 4: Filter by Enrollment Timing
**1st Quarter Report** includes only students who:
- Enrolled between August 1 and October 17, 2025, OR
- Were already enrolled before August 1 (continuing students)

**2nd Quarter Report** includes only students who:
- Enrolled between October 17 and December 20, 2025, OR
- Were already enrolled before August 1 (continuing students)

This ensures we only count students who were present during the grading period.

---

## Example Walk-Through

**Scenario**: John Smith (Client ID: 98765) enrolled on August 15, 2025 at Bethune Elementary

1. **Stage 1 - Import**: 
   - Grade file uploaded with John's record: 1Q GPA = 3.2, 2Q GPA = 2.8
   - System imports: client_id=98765, 1Q_grades=3.2, 2Q_grades=2.8

2. **Stage 2 - Match**:
   - John's master profile is matched to his grade record
   - His profile now includes: lighthouse_location_enrolling="Bethune Elementary", 1Q_grades=3.2

3. **Stage 2 - Check Performance**:
   - 1Q GPA = 3.2, which is >= 2.5
   - 1Q_grade_check = "GOOD"
   - 1q_counter = 1 (counts toward "meeting target")

4. **Stage 2 - Location Summary (1st Quarter)**:
   - John enrolled 8/15/2025, which is during 1st quarter period ✓
   - He's included in Bethune Elementary's count
   - Bethune gets: +1 to total_counted, +1 to meeting_target

5. **Final Report**:
   - If Bethune has 20 students total with 16 meeting target
   - Performance: 16 ÷ 20 = 80% meeting 2.5 GPA threshold

---

## Data Quality Notes

### Known Issues
- Table name has typo: `raw_ligthouse_grades` (missing 'h') - maintained for consistency with existing queries

### Important Considerations
1. **Grade Availability**: Not all students may have grades for all quarters
   - Students without grades are excluded from percentage calculations
   - "NOGRADE" status helps track missing data

2. **Enrollment Timing**: Reports only include students enrolled during or before the quarter
   - Prevents counting students who weren't present for the grading period
   - Students enrolled before August 1 are included in all quarters

3. **GPA Threshold**: 2.5 GPA represents a C+ average
   - May need adjustment based on program goals
   - Same threshold applied across all locations

4. **Data Source**: Manual CSV upload required for each quarter
   - No automated integration with school district grade systems
   - Requires coordination with school staff for data collection

---

## Future Enhancements

### 3rd Quarter Implementation
- Add `3q_lh_grades` output table
- Filter logic: Students enrolled by March 6, 2026
- Same calculation methodology as 1Q and 2Q

### Additional Metrics
- Subject-specific performance (Math, ELA, Science, etc.)
- Grade trend analysis (improving vs. declining)
- Comparison to school/district averages

### Data Integration
- Direct integration with school district student information systems
- Automated grade imports (eliminate manual CSV process)
- Real-time grade updates

---

## Related Documentation
- [Lighthouse Attendance Pipeline](Lighthouse-Attendance-Pipeline.md)
- [ServicePoint Data Reference](ServicePoint-Data-Reference.md) (if exists)
