# Urban Crime Prediction with Azure & Open Data

> **Note:** The comprehensive final executive report included in the `docs/` folder is originally written in Spanish as it was presented for a university capstone project in Big Data Analytics. However, this README covers the complete technical architecture, methodologies, DAX logic, and business outcomes in English.

## Project Overview
Traditional policing relies heavily on a reactive model—waiting for a 911 call. This project aims to shift towards a **preventive model** by validating the "Broken Windows Theory". By crossing NYC Open Data sets (311 citizen complaints regarding urban decay, specifically graffiti, vs. NYPD high-risk crime data), this project proves that environmental deterioration acts as an early warning signal for severe crime.

## Architecture & Tech Stack
This project features a 100% cloud-native ELT architecture deployed on Microsoft Azure, emphasizing automation, scalability, and robust data modeling.

* **CI/CD & Infrastructure as Code (IaC):** ETL pipelines, linked services, and Data Flows are versioned and exported as **Azure Resource Manager (ARM) templates** from Azure Data Factory, demonstrating CI/CD readiness.
* **Data Lake / Ingestion:** Azure Blob Storage for raw flat files (.csv) from NYC OpenData.
* **Orchestration & Transformation:** **Azure Data Factory (Mapping Data Flows)** for parallel data cleansing, geospatial schema casting, and pruning of 48+ redundant columns using a low-code approach.
* **Data Warehouse:** **Azure SQL Database** serving as the centralized analytical repository.
* **Data Modeling & Visualization:** **Power BI** utilizing a **Star Schema** with conformed dimensions (Date and Zone/Borough) for seamless cross-filtering across massive datasets.

## Methodology: Time-Lag (T+1) Analysis
To validate the predictive hypothesis, a Time-Lag correlation technique was implemented. Instead of joining datasets on their exact natural dates, the model associates the volume of complaints from the current month ($T$) with the crime rates of the following month ($T+1$). 

**Key Finding:** The analysis reveals a nuanced reality. At a macro level (citywide), there is a visible synchrony between graffiti complaints and subsequent high-risk crimes, validating the core hypothesis. However, the interactive dashboard demonstrates that this correlation varies significantly at the micro-level (by borough). While the general trend holds for New York City as a whole, the pattern's strength and shape change depending on the specific neighborhood dynamics of places like The Bronx, Brooklyn, or Manhattan.

## Core DAX & Data Modeling
To ensure seamless cross-filtering between the two Fact Tables (`TablaCrimenes` and `TablaQuejas`), dynamic conformed dimensions were built from scratch using DAX. Here is a highlight of the predictive Time-Lag measure:

```dax
// Calculates the volume of high-risk crimes, shifted by one month into the future
Crimen_Mes_Siguiente = 
CALCULATE(
    COUNT(TablaCrimenes[ID_Zona]), 
    DATEADD('Calendario'[Date], 1, MONTH)
)
```

## Deployment & CI/CD
The project is designed for automated deployment using Azure DevOps. The `adf-arm-template` folder contains the exported ARM templates for the Data Factory, allowing for the recreation of the entire ETL infrastructure in any Azure subscription with a single click.
