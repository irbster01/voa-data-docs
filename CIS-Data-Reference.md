# CIS (Communities In Schools) Data Reference

Complete reference for CIS data tables, dataflows, and reporting structure in the VOA data warehouse.

---

## Table of Contents
- [Raw Data Tables](#raw-data-tables)
- [Data Sources & Collection](#data-sources--collection)
- [Common Use Cases](#common-use-cases)
- [Key Fields Reference](#key-fields-reference)

---

## Raw Data Tables

### raw_cis_caselist
**Source**: CIS Data Management (CISDM) - Caselist Details Export  
**Update Frequency**: Manual export from CISDM system  
**Primary Use**: Student enrollment and case management tracking

**Key Columns**:
- `client_id` (Student System ID) - Unique identifier linking to other systems
- `sasid` (Student ID) - State-assigned student ID used for matching to school records
- `sidno` - School district student ID number (alternate identifier)
- `client_full_name` (Student Name)
- `client_dob` (Birth Date)
- `school_name` (Home School)
- `entry_start_date` (Enrollment Begin Date)
- `first_cis_enrollment_date` (First CIS Enrollment Date)

**Other Fields**:
- Site Model Status
- School Year
- Sex/Gender, Race/Ethnicity, Ethnicity, Tribe, Languages
- Case Managed Status
- Case Management Intensity Level
- Case Manager
- Grade Level
- Enrollment Status, Enrollment Exit Date
- Referral information (date, reasons, source)
- Student Attributes
- Risk Factors (Individual Student & Family)
- Initiatives

**Notes**: 
- Deduplicates on `Student System ID` and `sasid` (state student ID)
- Primary matching to school district records uses `sasid`
- Critical for cross-system student matching

---

### raw_cisdm_progress_monitoring
**Source**: CIS Data Management (CISDM) - Progress Monitoring Export  
**Update Frequency**: Manual export from CISDM system  
**Primary Use**: Academic and behavioral goal tracking

**Key Columns**:
- `client_id` (Student System ID)
- `sasid` (Student ID) - State student identifier
- `school_name` (School)
- `case_manager` (Case Manager)
- `goal` (Goal description)
- `metric` (Specific metric being tracked)
- `baseline`, `target` - Starting point and goal target
- `baseline_date`, `target_set_date`
- `grading_period` (Grading Period)
- `metric_value` (Current metric value)
- `review_date` (Review Date)
- `entry_date`, `exit_date` (Enrollment dates)

**Calculated Fields**:
- `short_name_academics` - Categorizes metrics into subjects:
  - "ELA" (Language Arts, English, Reading)
  - "Math"
  - "Science"
  - "Social Studies"
  - "Credit Completion"
  - "NOT ACADEMIC"
- `attendance_type` - Categorizes attendance metrics:
  - "Tardies"
  - "Absences"
  - "Attendance Rate"
  - "NOT ATTENDANCE"

**Other Fields**:
- Grade Level, Gender, Race, Ethnicity
- Baseline Reporting Period
- Grading Period Progress
- Non-Numeric Value
- Days in Reporting Period, Excused Absences, Unexcused Absences
- Latest Progress All Grading Periods
- Progress Notes, Goal Narrative
- Enrollment Status, School Year

**Notes**:
- Tracks student progress toward individual goals across multiple grading periods
- Includes both academic and attendance/behavior metrics
- Critical for measuring student outcomes and case manager effectiveness

---

### raw_cisdm_accreditation (Student Drill)
**Source**: CIS Data Management (CISDM) - Accreditation Student Drill  
**Update Frequency**: Manual export from CISDM system  
**Primary Use**: Compliance and accreditation reporting

**Key Columns**:
- `Organization`
- `Home School`
- `Site ID`, `NCES ID`
- `# of Grading Periods`
- `Student ID`, `Student System ID`
- `Student Name`
- `Case Managed Status`
- `Case Management Intensity Level`
- `Grade Level`
- `Case Manager`
- `Current Enrollment Status`
- `Enrollment Begin Date`

**Compliance Tracking**:
- `Student Needs Assessment Entered?`
- `Student Support Plan Entered?`
- `# of Tier II/III Supports`
- `# of Check-Ins`
- `On Track for Monthly Check-Ins?`
- `# Goal Areas`
- `# ABCS Metrics Assigned`
- `# Non-ABCS Metrics Assigned`
- `# Progress Entries`
- `Expected Progress Entries for the Year`
- `# of Metrics with Goal Achievement entered`
- `Goal Achievement Entered for ALL assigned goals?`
- `EOY Status Entered`

**Demographics**:
- Race/Ethnicity
- Sex/Gender
- Ethnicity

**Notes**:
- Used to verify CIS program compliance with national accreditation standards
- Tracks documentation completeness and service delivery metrics
- Critical for annual accreditation reviews

---

### raw_cisdm_student_planned_supports
**Source**: CIS Data Management (CISDM) - Student Planned Supports Export  
**Update Frequency**: Manual export from CISDM system  
**Primary Use**: Tracking planned interventions and support services for students

**Notes**:
- Details what supports/interventions are planned for each student
- Links to student support plans

---

### raw_cisdm_student_support_detail
**Source**: CIS Data Management (CISDM) - Student Support Detail Export  
**Update Frequency**: Manual export from CISDM system  
**Primary Use**: Tracking delivered interventions and support services

**Notes**:
- Records actual delivery of support services
- Critical for tracking service hours and intervention implementation

---

### raw_cisdm_school_goals
**Source**: CIS Data Management (CISDM) - School Goals Export  
**Update Frequency**: Manual export from CISDM system  
**Primary Use**: School-level goal tracking and performance monitoring

**Notes**:
- Aggregates goals at school level
- Used for site-level reporting and analysis

---

## Data Sources & Collection

### Collection Process
1. **Manual Export**: Data is exported from CIS Data Management system
2. **File Storage**: Excel files are uploaded to OneLake Files folder: `2526 CIS/`
3. **Dataflow Ingestion**: Dataflows automatically pick up latest files from lakehouse
4. **Transformation**: Data is cleaned, standardized, and loaded to warehouse tables

### Source System
**CIS Data Management (CISDM)**  
National case management platform used by Communities In Schools affiliates

---

## Common Use Cases

### Student Matching Across Systems
**Tables Used**: `raw_cis_caselist`  
**Purpose**: Link CIS students to school district records (JCampus, Caddo Parish data)  
**Key Fields**: `client_id` (Student System ID), `sasid` (State-assigned Student ID)

### Progress Monitoring & Outcomes
**Tables Used**: `raw_cisdm_progress_monitoring`  
**Purpose**: Track student goal progress in academics, attendance, and behavior  
**Key Fields**: `goal`, `metric`, `baseline`, `target`, `metric_value`, `grading_period`

**Example Metrics**:
- Attendance Rate (%)
- Days Absent
- ELA/Math/Science grades
- Credit Completion
- Behavioral incidents

### Compliance & Accreditation Reporting
**Tables Used**: `raw_cisdm_accreditation`  
**Purpose**: Verify program meets CIS national standards  
**Key Checks**:
- Are needs assessments completed?
- Are support plans documented?
- Are check-ins happening monthly?
- Are progress entries recorded?
- Is goal achievement tracked?

### Service Delivery Tracking
**Tables Used**: `raw_cisdm_student_support_detail`, `raw_cisdm_student_planned_supports`  
**Purpose**: Track planned vs. delivered support services  
**Use**: Measure intervention fidelity and service hours

---

## Key Fields Reference

### Student Identifiers
| Field Name | Description | Example | Used For |
|------------|-------------|---------|----------|
| `client_id` / `Student System ID` | CIS internal ID | "12345678" | Primary CIS identifier |
| `sasid` / `Student ID` | State-assigned ID | "LA123456789" | **Primary key for matching to school records** |
| `sidno` | School district ID | "987654" | Alternate identifier (not used for matching) |

### Enrollment & Status
| Field Name | Description | Values |
|------------|-------------|--------|
| `entry_start_date` / `Enrollment Begin Date` | When student enrolled in CIS | Date |
| `exit_date` / `Enrollment Exit Date` | When student exited CIS | Date or NULL |
| `first_cis_enrollment_date` | First time ever enrolled in CIS | Date |
| `enrollment_status` | Current enrollment state | "Active", "Exited", etc. |
| `case_managed_status` | Whether receiving case management | "Case Managed", "Not Case Managed" |
| `case_management_intensity_level` | Level of support | "Tier 1", "Tier 2", "Tier 3" |

### Demographics
| Field Name | Description |
|------------|-------------|
| `school_name` / `Home School` | Student's school |
| `grade` / `Grade Level` | Current grade level |
| `gender` / `Sex/Gender` | Gender identity |
| `race` / `Race/Ethnicity` | Race/ethnicity |
| `client_dob` / `Birth Date` | Date of birth |

### Goals & Metrics
| Field Name | Description | Example |
|------------|-------------|---------|
| `goal` | Goal description | "Improve math grades" |
| `metric` | Specific measure | "Math Grade (%)" |
| `baseline` | Starting value | 65 |
| `target` | Goal target | 80 |
| `metric_value` | Current value | 72 |
| `grading_period` | Time period | "1st 9 Weeks", "2nd Semester" |
| `review_date` | When progress was reviewed | Date |

### Calculated Categories
| Field Name | Logic | Values |
|------------|-------|--------|
| `short_name_academics` | Categorizes academic metrics | "ELA", "Math", "Science", "Social Studies", "Credit Completion", "NOT ACADEMIC" |
| `attendance_type` | Categorizes attendance metrics | "Tardies", "Absences", "Attendance Rate", "NOT ATTENDANCE" |

---

## Related Documentation
- [Lighthouse Attendance Pipeline](../servicepoint/transformations/lighthouse_pipeline/README.md)
- School District Data (JCampus/Caddo Parish) - *documentation pending*
- ServicePoint Integration - *documentation pending*

---

## Data Quality Notes

### Known Issues
- Student IDs may have leading zeros that get dropped in Excel exports
- Some students may have multiple enrollment records (track by `first_cis_enrollment_date`)
- Manual exports mean data freshness varies by report type

### Data Validation Checks
- Verify `client_id` uniqueness in caselist
- Check for NULL values in key identifier fields
- Validate date ranges (entry_date should be before exit_date)
- Ensure case-managed students have assigned case managers

---

*Last Updated: January 13, 2026*
