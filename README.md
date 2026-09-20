# ============================================================
# HOSPITAL PATIENT CARE ANALYTICS PIPELINE
# Student: Chetan Kalagarla
# Student ID: 2300033450
# ============================================================

# ------------------------------------------------------------
# 1. IMPORT LIBRARIES
# ------------------------------------------------------------

import pandas as pd
import numpy as np
import sqlite3
import matplotlib.pyplot as plt

pd.set_option("display.max_columns", None)

np.random.seed(42)

print("Libraries imported successfully.")


# ------------------------------------------------------------
# 2. CREATE SYNTHETIC PATIENT REGISTRATION DATA
# ------------------------------------------------------------

patients = pd.DataFrame({
    "patient_id": range(1001, 1011),
    "age": [25, 67, 45, 72, 34, 58, 81, 29, 63, 50],
    "gender": ["F", "M", "F", "M", "F", "M", "F", "M", "F", "M"],
    "department": [
        "Cardiology",
        "General",
        "Neurology",
        "Cardiology",
        "General",
        "Orthopedics",
        "Cardiology",
        "General",
        "Neurology",
        "Orthopedics"
    ]
})

print("\nPATIENT REGISTRATION DATA")
print(patients)


# ------------------------------------------------------------
# 3. CREATE APPOINTMENT DATA
# ------------------------------------------------------------

appointments = pd.DataFrame({
    "patient_id": range(1001, 1011),
    "appointment_date": pd.date_range(
        "2026-09-01",
        periods=10,
        freq="D"
    ),
    "waiting_minutes": [
        18, 42, 27, 55, 15,
        35, 62, 20, 48, 30
    ],
    "appointment_status": [
        "Completed",
        "Completed",
        "Completed",
        "Completed",
        "Completed",
        "Completed",
        "Completed",
        "Completed",
        "Completed",
        "Completed"
    ]
})

print("\nAPPOINTMENT DATA")
print(appointments)


# ------------------------------------------------------------
# 4. CREATE LABORATORY DATA
# ------------------------------------------------------------

labs = pd.DataFrame({
    "patient_id": range(1001, 1011),
    "hemoglobin": [
        13.2, 10.1, 12.8, 9.7, 13.5,
        11.2, 8.9, 14.1, 10.5, 12.0
    ],
    "glucose": [
        92, 165, 108, 190, 88,
        145, 210, 96, 172, 118
    ],
    "lab_abnormal": [
        0, 1, 0, 1, 0,
        1, 1, 0, 1, 0
    ]
})

print("\nLABORATORY DATA")
print(labs)


# ------------------------------------------------------------
# 5. CREATE WEARABLE DEVICE DATA
# ------------------------------------------------------------

wearables = pd.DataFrame({
    "patient_id": range(1001, 1011),
    "heart_rate": [
        78, 96, 88, 105, 74,
        92, 112, 76, 101, 85
    ],
    "oxygen_sat": [
        98, 94, 97, 91, 99,
        95, 89, 98, 93, 96
    ],
    "temperature": [
        36.7, 37.2, 36.8, 38.1, 36.6,
        37.5, 38.4, 36.5, 37.8, 36.9
    ]
})

print("\nWEARABLE DATA")
print(wearables)


# ------------------------------------------------------------
# 6. CREATE DOCTOR CONSULTATION DATA
# ------------------------------------------------------------

consultations = pd.DataFrame({
    "patient_id": range(1001, 1011),
    "symptom_score": [
        2, 6, 4, 8, 1,
        5, 9, 2, 7, 3
    ],
    "consultation_minutes": [
        12, 20, 15, 25, 10,
        18, 30, 11, 22, 14
    ],
    "follow_up_required": [
        0, 1, 0, 1, 0,
        1, 1, 0, 1, 0
    ]
})

print("\nCONSULTATION DATA")
print(consultations)


# ------------------------------------------------------------
# 7. DATA QUALITY CHECKS
# ------------------------------------------------------------

sources = {
    "patients": patients,
    "appointments": appointments,
    "labs": labs,
    "wearables": wearables,
    "consultations": consultations
}

print("\nDATA QUALITY CHECKS")

for name, df in sources.items():

    print(
        f"{name}: "
        f"rows={len(df)}, "
        f"duplicates={df.duplicated().sum()}, "
        f"missing={df.isna().sum().sum()}"
    )


# ------------------------------------------------------------
# 8. INTEGRATE ALL DATA SOURCES
# ------------------------------------------------------------

data = patients.merge(
    appointments,
    on="patient_id",
    how="left"
)

data = data.merge(
    labs,
    on="patient_id",
    how="left"
)

data = data.merge(
    wearables,
    on="patient_id",
    how="left"
)

data = data.merge(
    consultations,
    on="patient_id",
    how="left"
)

print("\nINTEGRATED DATASET")
print("Shape:", data.shape)
print(data.head())


# ------------------------------------------------------------
# 9. FEATURE ENGINEERING
# ------------------------------------------------------------

data["age_group"] = pd.cut(
    data["age"],
    bins=[0, 39, 59, 200],
    labels=[
        "Young",
        "Middle",
        "Senior"
    ]
)

# Waiting-time flag
data["long_wait_flag"] = (
    data["waiting_minutes"] >= 45
).astype(int)

# Oxygen flag
data["low_oxygen_flag"] = (
    data["oxygen_sat"] < 94
).astype(int)

# Heart-rate flag
data["high_heart_rate_flag"] = (
    data["heart_rate"] > 100
).astype(int)

# Glucose flag
data["high_glucose_flag"] = (
    data["glucose"] > 140
).astype(int)

# Symptom flag
data["high_symptom_flag"] = (
    data["symptom_score"] >= 7
).astype(int)


# ------------------------------------------------------------
# 10. DEMONSTRATION RISK SCORE
# ------------------------------------------------------------

data["risk_score"] = (
    (data["age"] >= 65).astype(int)
    + data["lab_abnormal"]
    + data["low_oxygen_flag"]
    + data["high_heart_rate_flag"]
    + data["high_glucose_flag"]
    + data["high_symptom_flag"]
    + data["follow_up_required"]
)

data["risk_level"] = pd.cut(
    data["risk_score"],
    bins=[-1, 2, 4, 10],
    labels=[
        "Low",
        "Medium",
        "High"
    ]
)

print("\nPATIENT RISK ANALYTICS")

print(
    data[
        [
            "patient_id",
            "age",
            "department",
            "waiting_minutes",
            "glucose",
            "oxygen_sat",
            "symptom_score",
            "risk_score",
            "risk_level"
        ]
    ]
)


# ------------------------------------------------------------
# 11. STORE DATA IN SQLITE
# ------------------------------------------------------------

db_path = "hospital_analytics.db"

conn = sqlite3.connect(db_path)

data.to_sql(
    "patient_analytics",
    conn,
    if_exists="replace",
    index=False
)

print("\nSQLite database created successfully.")
print("Database:", db_path)


# ------------------------------------------------------------
# 12. READ DATA FROM SQLITE
# ------------------------------------------------------------

stored = pd.read_sql(
    "SELECT * FROM patient_analytics",
    conn
)

print("\nDATA READ FROM SQLITE")
print(stored.head())


# ------------------------------------------------------------
# 13. OVERALL AVERAGE WAITING TIME
# ------------------------------------------------------------

avg_wait = data["waiting_minutes"].mean()

print(
    f"\nAverage waiting time: "
    f"{avg_wait:.1f} minutes"
)


# ------------------------------------------------------------
# 14. WAITING TIME BY DEPARTMENT
# ------------------------------------------------------------

dept_wait = (
    data
    .groupby(
        "department",
        as_index=False
    )["waiting_minutes"]
    .mean()
    .sort_values(
        "waiting_minutes",
        ascending=False
    )
)

print("\nWAITING TIME BY DEPARTMENT")
print(dept_wait)


# ------------------------------------------------------------
# 15. WAITING TIME VISUALIZATION
# ------------------------------------------------------------

plt.figure(figsize=(8, 4))

plt.bar(
    dept_wait["department"],
    dept_wait["waiting_minutes"]
)

plt.title(
    "Average Waiting Time by Department"
)

plt.xlabel("Department")
plt.ylabel("Waiting Time (Minutes)")

plt.xticks(rotation=25)

plt.tight_layout()

plt.show()


# ------------------------------------------------------------
# 16. RISK LEVEL SUMMARY
# ------------------------------------------------------------

risk_summary = (
    data["risk_level"]
    .value_counts()
    .reindex(
        ["Low", "Medium", "High"]
    )
    .fillna(0)
)

print("\nRISK LEVEL SUMMARY")
print(risk_summary)


# ------------------------------------------------------------
# 17. RISK LEVEL VISUALIZATION
# ------------------------------------------------------------

plt.figure(figsize=(6, 4))

plt.bar(
    risk_summary.index.astype(str),
    risk_summary.values
)

plt.title(
    "Patients by Risk Level"
)

plt.xlabel("Risk Level")
plt.ylabel("Number of Patients")

plt.tight_layout()

plt.show()


# ------------------------------------------------------------
# 18. HIGH-RISK PATIENT VIEW
# ------------------------------------------------------------

high_risk = data.loc[
    data["risk_level"] == "High",
    [
        "patient_id",
        "age",
        "department",
        "risk_score",
        "oxygen_sat",
        "heart_rate",
        "glucose",
        "symptom_score"
    ]
].sort_values(
    "risk_score",
    ascending=False
)

print("\nHIGH-RISK PATIENT VIEW")
print(high_risk)


# ------------------------------------------------------------
# 19. PIPELINE MONITORING
# ------------------------------------------------------------

monitor = pd.DataFrame({
    "metric": [
        "source patient rows",
        "integrated rows",
        "null values",
        "duplicate rows",
        "high-risk records"
    ],
    "value": [
        len(patients),
        len(data),
        int(
            data.isna()
            .sum()
            .sum()
        ),
        int(
            data.duplicated()
            .sum()
        ),
        int(
            (
                data["risk_level"]
                == "High"
            ).sum()
        )
    ]
})

print("\nPIPELINE MONITORING")
print(monitor)


# ------------------------------------------------------------
# 20. EXPORT FINAL DATASET
# ------------------------------------------------------------

output_file = (
    "hospital_patient_analytics.csv"
)

data.to_csv(
    output_file,
    index=False
)

print(
    "\nCSV exported successfully:"
)

print(output_file)


# ------------------------------------------------------------
# 21. FINAL PROJECT SUMMARY
# ------------------------------------------------------------

print("\n" + "=" * 60)

print(
    "HOSPITAL PATIENT CARE ANALYTICS PIPELINE"
)

print(
    "Student: Chetan Kalagarla"
)

print(
    "Student ID: 2300033450"
)

print("=" * 60)

print(
    """
Pipeline completed:

1. Created synthetic hospital data
2. Performed data-quality checks
3. Integrated multiple data sources
4. Created analytical features
5. Generated demonstration risk scores
6. Stored data in SQLite
7. Analyzed waiting times
8. Analyzed patient risk levels
9. Generated visualizations
10. Performed pipeline monitoring
11. Exported analytics-ready CSV
"""
)

# Close database connection
conn.close()

print("Database connection closed.")
print("Project execution completed successfully.")
