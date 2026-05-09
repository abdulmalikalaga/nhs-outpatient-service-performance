# 📊 Power BI Project: NHS Outpatient Service Performance Dashboard

## 🧠 Project Overview

### Objective
To analyse outpatient performance across a large NHS outpatient service by identifying:

- Clinics with the longest wait times
- Clinics and clinicians with the highest cancellation and no-show rates
- Underlying operational bottlenecks
- Demographic patterns contributing to service pressure

The dashboard enables operational leads to monitor performance, address inefficiencies, and support data-driven service planning.

### Context
The organisation provided datasets containing patient appointments, clinic schedules, cancellations, wait times, clinician assignments, and demographic information.

By modelling and visualising these datasets in Power BI, the goal was to reveal performance variations and highlight areas requiring operational improvement.

---

# 🗂️ Data Overview

## Source
All datasets were synthetic CSV files created for this project, representing:

- Appointment activity
- Patient demographics
- Clinic information
- Clinician assignments
- Cancellation events
- Clinic session capacity

## Description of Tables

### 1. appointments (Fact)
Contains appointment-level details: ID, patient, clinic, clinician, scheduled date, appointment date, status, outcome, and calculated wait time.

### 2. patients (Dimension)
Demographics including age, gender, postcode, and region.

### 3. clinics (Dimension)
Clinic name, specialty, and location.

### 4. clinicians (Dimension)
Clinician name, role, specialty.

### 5. cancellations (Fact)
Specific cancellation events tied to `appointment_id` and reason.

### 6. sessions (Fact)
Clinic session capacity and utilisation: session date, capacity slots, filled slots.

## Data Preparation in Power Query

Key transformations included:

- Renamed columns for cleaner, consistent naming
- Removed unnecessary characters (e.g., dashes)
- Created `AgeBand` using a custom M column because Power Query removed the “+” in “75+” when using conditional columns:

```m
if [age] >= 75 then "75+"
else if [age] >= 60 then "60–74"
else if [age] >= 45 then "45–59"
else if [age] >= 30 then "30–44"
else "18–29"
```

- Created flags in appointments:
  - `attended_flag`
  - `cancelled_flag`
  - `no_show_flag`

- Created `utilisation_pct` in sessions and formatted as percentage
- Confirmed data types for all columns
- Removed or validated problematic rows where necessary

---

# 🧮 Data Modelling

## Model Structure

A clean star schema was implemented.

### Fact Tables
- appointments
- cancellations
- sessions

### Dimension Tables
- patients
- clinics
- clinicians
- Date (DAX-generated)

## Relationships

- `appointments[patient_id] → patients[patient_id]`
- `appointments[clinic_id] → clinics[clinic_id]`
- `appointments[clinician_id] → clinicians[clinician_id]`
- `cancellations[appointment_id] → appointments[appointment_id]`
- `sessions[clinic_id] → clinics[clinic_id]`
- `Date[Date] → appointments[appointment_date], sessions[session_date]`

## Date Table (DAX)

```DAX
Date =
ADDCOLUMNS (
    CALENDAR ( DATE(2023,1,1), DATE(2025,12,31) ),
    "Year", YEAR([Date]),
    "Month", MONTH([Date]),
    "MonthName", FORMAT([Date], "MMMM"),
    "MonthShort", FORMAT([Date], "MMM"),
    "YearMonth", FORMAT([Date], "YYYY-MM"),
    "WeekOfYear", WEEKNUM([Date], 2),
    "Quarter", QUARTER([Date])
)
```

## Measures

A dedicated measures table was created. Examples include:

- Total Appointments
- Attended Appointments
- Cancellation Rate
- No-Show Rate
- Average Wait Time
- Follow-up / Referred / Discharged counts
- Avg Utilisation

A special measure was created for Page 3 to allow correct filtering by clinic:

```DAX
Patients per Clinic =
CALCULATE(
    DISTINCTCOUNT(appointments[patient_id])
)
```

This ensured the age-band visual responded correctly to clinic selection.

---

# 📈 Dashboard Design

## Purpose
To monitor outpatient activity, performance variation across clinics, and identify inefficiencies affecting patient experience and service delivery.

## Main Features

### Page 1 – Overview
- KPI cards:
  - Total appointments
  - Attended rate
  - Cancellation rate
  - No-show rate
  - Average wait days
- Total appointments by clinic (column chart)
- Wait time distribution
- Cancellation reasons (donut chart)
- Slicers for clinic, clinician, age band
- Edit interactions applied to prevent slicers from affecting visuals that must remain global

### Page 2 – Clinic & Clinician Performance
- Gauges for Attended Rate and Cancellation Rate with thresholds (red/amber/green)
- Slicers for clinic and clinician
- Cancellation rate by clinic visual
- Scatter chart comparing attendance rate vs average wait time
- Capacity vs utilisation visual using sessions data
- Conditional formatting thresholds aligned with operational expectations

### Page 3 – Patient Demographics
- Patient age-band distribution using the corrected “Patients per Clinic” measure
- Attempted map visual removed because synthetic postcodes were not geographically valid
- Additional visuals on outcomes and demographic patterns

## Interactivity
- Slicers for clinic, clinician, age band
- Corrected cross-filtering behaviour using Edit Interactions
- Gauges adjust based on slicer context when labelled generically (“Cancellation Rate”)

---

# 🔍 Insights

## 1. Clinic Performance
- Central Outpatient has the highest capacity and one of the longest average waits (~31 days)
- Eastside Cardio, Southside General, and Westside Ortho also show long waiting times

## 2. Cancellation & No-Show Behaviour
- Westside Ortho has the highest cancellation rate
- The most common cancellation reason in this clinic is Clinician Unavailable
- Across all clinics, Transport Issues is the most common cancellation reason
- Northside Neuro and Eastside Cardio contribute heavily to this
- Indicates potential accessibility issues

## 3. Clinician Performance
- Clinician 7 has the highest cancellation rate (30% across all clinics)
- In Northside Neuro, Clinician 7’s cancellation rate rises to 56%, primarily due to patient transport issues
- Westside Ortho has only one clinician, which may contribute to bottlenecks and increased cancellations

## 4. Demographic Patterns
- Patients aged 18–29 (male) booked the most appointments (36) and also had the highest cancellation rate (25%)
- In Westside Ortho, females aged 30–44 have the highest cancellation rate (50%)
- Patients aged 75+ generated the most appointments (26) across all clinics
- Most patients were either discharged or scheduled for follow-up; only 18% were referred onward

## 5. Capacity Utilisation
- Central and Eastside clinics had the highest average slot utilisation (~76%)
- Lower utilisation in other clinics suggests imbalances in scheduling and demand

---

# ⚙️ Challenges and Solutions

## Issue: Age Band “75+” losing the plus sign
Power Query stripped the “+” when using conditional columns.

### Solution
Used a Custom Column with M code to enforce text literals.

---

## Issue: Patient age-band visual not responding to clinic slicer
The patients table does not contain clinic fields.

### Solution
Created a measure using `DISTINCTCOUNT` filtered through appointments.

---

## Issue: Map visual unusable
Synthetic postcode data was not geographically valid.

### Solution
Removed the map visual to avoid misleading analysis.

---

## Issue: Bookmarks cannot be assigned to text boxes
Power BI does not support actions on text boxes.

### Solution
Used shapes/buttons instead.

---

# 🚀 Outcome / Value

The final dashboard enables the NHS outpatient service to:

- Identify clinics with excessive waits or poor slot utilisation
- Detect clinicians contributing to high cancellation rates
- Understand operational bottlenecks such as single-clinician clinics and accessibility issues
- Explore demographic influences on service demand and cancellations
- Support evidence-based workforce, scheduling, and accessibility decisions

This solution provides a clear, intuitive, and data-driven view of outpatient performance suitable for operational managers and clinical leads.

---

# 📁 PBIX File

[Download the Power BI Dashboard]

---

⭐ **If you found this project insightful, please star ⭐ this repository!**  
📬 *Let’s connect on [LinkedIn](https://www.linkedin.com/in/abdulmalikalaga/)
 — open to data-based analytics roles.
