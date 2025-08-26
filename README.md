# Irish-Healthcare-Waiting-Times-Dashboard

## 📝 Project Overview

Ireland’s healthcare system has long faced challenges with hospital waiting lists, where patients often experience delays before receiving treatment. This project analyses official waiting list data and presents it in an interactive Power BI dashboard.
The goal is to transform raw data into clear, actionable insights for healthcare stakeholders and highlight patterns that may support better decision-making.


## 🎯 Problem Statement

Long waiting times can negatively impact patient outcomes.
National datasets are available, but difficult to interpret in raw tables.
There is a need for a visual, interactive tool that allows policymakers and hospital administrators to explore trends by hospital, region, and speciality.



## 📂 Dataset Description

Source: National Treatment Purchase Fund (NTPF) https://www.ntpf.ie/waiting-list-data/open-data/

Files Included in Repo:
Waiting List by Hospital.xlsx – Patient counts by hospital, inpatient/outpatient, age group, and waiting time bands.
Waiting List by Speciality.xlsx – Patient counts by speciality, inpatient/outpatient, age group, and waiting time bands.
NTPF_WaitTimes.pbix – Power BI dashboard file.

**Note: This report only contains data between January 2023 to July 2025**

## 👉 Example data fields:

Date (monthly)

Hospital / Speciality

Patient Type (Adult/Child, Inpatient/Outpatient)

Wait Band (0–6 months, 6–12 months, 12–18 months, 18+ months)

Patient Count



## 🛠️ Methodology

Data Import – Loaded Excel datasets into Power BI.

Data Modelling – Created a date dimension for time-series analysis and relationships across datasets.:
- Rename the Hospital query to Fact_WaitByHospital
- Rename the Speciality query to Fact_WaitBySpecialty
- Unpivot the wait bands in the Hospital and Speciality table
- Created a dimension table 'DimHospital' containing Hospital, Latitude, and Longitude columns.
- Created a dimension table 'DimSpecialty' containing the speciality column.
- Created a dimension table 'DimPatientType' containing the PatientType column. (both in Fact_WaitByHospital and Fact_WaitBySpecialty)
- Created a dimension table 'DimCaseType' containing the CaseType column.
- DimWaitBand 
- Created DimDate table
  ```

  DimDate =
  VAR MinDate =
    MINX(
        {
            ( MIN ( Fact_WaitByHospital[date] ) ),
            ( MIN ( Fact_WaitBySpecialty[date] ) )
        },
        [Value]
    )
  VAR MaxDate =
    MAXX(
        {
            ( MAX ( Fact_WaitByHospital[date] ) ),
            ( MAX ( Fact_WaitBySpecialty[date] ) )
        },
        [Value]
    )
  RETURN
  ADDCOLUMNS(
    CALENDAR ( MinDate, MaxDate ),
    "Year", YEAR ( [Date] ),
    "Month", FORMAT ( [Date], "MMMM" ),
    "MonthNo", MONTH ( [Date] ),
    "YearMonth", FORMAT ( [Date], "YYYY-MM" ),
    "Year Month", FORMAT ( [Date], "YYYY mmm" )
  ) 
Created Various DAX Measures:

```

Patients (Hosp) =
SUM ( Fact_WaitByHospital[Patients] )

Patients >12m (Hosp) =
CALCULATE (
    [Patients (Hosp)],
    DimWaitBand[SortOrder] >= 3        -- 12-18 and 18+
)

% >12m (Hosp) =
DIVIDE ( [Patients >12m (Hosp)], [Patients (Hosp)] )

Patients - Adults (Hosp) =
CALCULATE ( [Patients (Hosp)], DimPatientType[PatientType] = "Adult" )

Patients - Children (Hosp) =
CALCULATE ( [Patients (Hosp)], DimPatientType[PatientType] = "Child" )

% Adults (Hosp) =
DIVIDE ( [Patients - Adults (Hosp)], [Patients (Hosp)] )

Patients - Inpatient (Hosp) =
CALCULATE ( [Patients (Hosp)], DimCaseType[CaseType] = "Inpatient" )

Patients - Outpatient (Hosp) =
CALCULATE ( [Patients (Hosp)], DimCaseType[CaseType] = "Outpatient" )

Avg Wait (months) (Hosp) =
VAR WeightedSum =
    SUMX (
        Fact_WaitByHospital,
        Fact_WaitByHospital[Patients] * RELATED ( DimWaitBand[MidpointMonths] )
    )
RETURN
DIVIDE ( WeightedSum, [Patients (Hosp)] )

Median Wait Band (Hosp) =
VAR C0 = CALCULATE ( [Patients (Hosp)], DimWaitBand[SortOrder] = 1 )
VAR C1 = CALCULATE ( [Patients (Hosp)], DimWaitBand[SortOrder] = 2 )
VAR C2 = CALCULATE ( [Patients (Hosp)], DimWaitBand[SortOrder] = 3 )
VAR C3 = CALCULATE ( [Patients (Hosp)], DimWaitBand[SortOrder] = 4 )
VAR Tot = C0 + C1 + C2 + C3
VAR Half = DIVIDE ( Tot, 2 )
VAR Cum1 = C0
VAR Cum2 = C0 + C1
VAR Cum3 = C0 + C1 + C2
RETURN
SWITCH (
    TRUE(),
    Cum1 >= Half, "0-6 Months",
    Cum2 >= Half, "6-12 Months",
    Cum3 >= Half, "12-18 Months",
    "18 Months +"
)

Median Wait (months approx) (Hosp) =
VAR Band =
    [Median Wait Band (Hosp)]
RETURN
SWITCH (
    Band,
    "0-6 Months", 3,
    "6-12 Months", 9,
    "12-18 Months", 15,
    "18 Months +", 21,
    BLANK()
)

Patients (Hosp) - 12m+ only =
[Patients >12m (Hosp)]

Patients (Hosp) - All =
[Patients (Hosp)]

```

Dashboard Design – Built 3 interactive report pages:

Page 1: Overview

Page 2: Wait Time by Hospital

Page 3: Wait Time by Speciality

## 📊 Dashboard Features
🔹 Page 1 – Hospital Overview

- Map of Ireland → Average wait times by hospital.

- Stacked Bar Chart → Distribution of waiting times (0–6, 6–12, 12–18, 18+ months) and patient type.

- Trend Line → Changes in wait times over months.

- KPIs → Median wait time, % > 12 months.

- Donut Chart to show the split between Patient Type and Case Type

<img width="904" height="511" alt="Overview" src="https://github.com/user-attachments/assets/e93acfbe-0ae0-4577-8117-9935d45ef986" />


🔹 Page 2 – Wait Time by Hospitals


- Slicers - Date, Patient Type, Case Type

- Hospital ranking by Median Wait

- Wait band distribution by hospital (stacked bar)

- Matrix heatmap

- Trend for selected hospital (drilldown)

- Detail table with sparklines

<img width="899" height="511" alt="By Hospital" src="https://github.com/user-attachments/assets/c425d392-5a8f-472d-9457-513dffaaf050" />


🔹 Page 3 – Speciality Analysis

- KPIs - Total Patients, % Waiting > 12 Months

- Slicers - Patient Type, Speciality, Month

- Distribution of patients by wait band per speciality (Stacked Bar Chart (100%))

- Inpatient vs Outpatient (Clustered Column Chart)

- Trends over time per speciality (Line Chart) (Top 5)

<img width="902" height="511" alt="By Speciality" src="https://github.com/user-attachments/assets/23b292b8-d217-494c-bd96-985811bcbbda" />


## 📌 Key Insights

- Orthopaedics has the highest backlog, with the maximum patients waiting

- Galway University Hospital reports longer wait times than the national median.

- Child patients generally have shorter wait times compared to adults.

- Outpatient cases account for ~87% of total waiting lists.

## ⚙️ Tools & Technologies

- Power BI Desktop → Data modelling, DAX calculations, visualisations.

- Excel → Data source (https://www.ntpf.ie/waiting-list-data/open-data/).

## 🚀 How to Use the Dashboard

- Clone this repository.

- Download and install Power BI Desktop

- NTPF_WaitTimes.pbix file.

- Use filters to explore:
By hospital, 
By speciality, 
By inpatient/outpatient, 
By time period

## 🔮 Future Improvements

- Automate monthly refresh with Power BI Service.

- Add predictive forecasting for waiting times.

- Benchmark Irish wait times against other EU countries.

## 📖 References

NTPF Official Waiting List Data
https://www.ntpf.ie/waiting-list-data/open-data/
