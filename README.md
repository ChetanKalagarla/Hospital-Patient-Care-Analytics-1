<div align="center">

# 🏥 Hospital Patient Care Analytics Pipeline

### 📊 End-to-End Data Engineering Case Study

**Chetan Kalagarla**  
**Student ID: 2300033450**

<br>

<img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white">
<img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white">
<img src="https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white">
<img src="https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white">
<img src="https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white">
<img src="https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge&logo=matplotlib&logoColor=white">

</div>

---

## 📌 Project Overview

A multi-specialty hospital collects data from multiple systems such as:

- 👤 Patient Registration
- 📅 Appointment Scheduling
- 🧪 Laboratory Reports
- ⌚ Wearable Health Devices
- 🩺 Doctor Consultation

Managing these sources separately makes it difficult to understand patient flow, waiting times, health indicators and operational performance.

This project demonstrates an **end-to-end data engineering pipeline** that collects, validates, integrates, transforms, stores and analyzes hospital data.

---

## 🎯 Project Objectives

The main objectives of this project are:

| Objective | Description |
|:---|:---|
| 🔗 Data Integration | Combine data from multiple hospital systems |
| 🧹 Data Quality | Identify missing values and duplicate records |
| ⚙️ Data Transformation | Convert raw data into useful analytical features |
| 🗄️ Data Storage | Store processed data using SQLite |
| ⏱️ Operational Analytics | Analyze patient waiting times |
| 🚨 Risk Analytics | Generate simple demonstration risk levels |
| 📊 Visualization | Present important analytical results |
| 🔍 Monitoring | Track basic pipeline-quality metrics |

---

## 🏗️ System Architecture

```text
                 🏥 HOSPITAL DATA SOURCES
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   Registration     Appointments      Laboratory
        │                │                │
        └────────────────┼────────────────┘
                         │
                 ⌚ Wearables
                         │
                 🩺 Consultation
                         │
                         ▼
                 📥 DATA INGESTION
                         │
                         ▼
              🧹 DATA QUALITY CHECK
                         │
                         ▼
              ⚙️ DATA TRANSFORMATION
                         │
                         ▼
                🔗 DATA INTEGRATION
                         │
                         ▼
                🗄️ SQLITE STORAGE
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       ⏱️ OPERATIONAL          🚨 RISK
          ANALYTICS             ANALYTICS
              │                     │
              └──────────┬──────────┘
                         ▼
                  📊 INSIGHTS
