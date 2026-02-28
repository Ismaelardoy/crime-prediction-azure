# DAX Measures & Data Modeling Logic

This document contains the core DAX (Data Analysis Expressions) formulas used in Power BI to build the Star Schema and calculate the predictive Time-Lag (T+1) metrics.

---

## 1. Predictive Logic: Time-Lag (T+1) Analysis

The core of the "Broken Windows Theory" validation relies on aligning current environmental complaints (T) with the crime rate of the following month (T+1).

**Next Month Crime (`Crimen_Mes_Siguiente`):**
Calculates the volume of high-risk crimes, shifted by one month into the future to match the current month's complaints on the same axis.

```dax
Crimen_Mes_Siguiente = 
CALCULATE(
    COUNT(TablaCrimenes[ID_Zona]), 
    DATEADD('Calendario'[Date], 1, MONTH)
)
```

---

## 2. Dynamic Star Schema Construction (Conformed Dimensions)

To ensure seamless cross-filtering between the two massive Fact Tables (`TablaCrimenes` and `TablaQuejas`), dynamic conformed dimensions were created using DAX.

### Dynamic Calendar (`Calendario`)

Automatically generates a continuous date table bridging the minimum and maximum dates found across both datasets.

```dax
Calendario = 
VAR FechaMin = MIN(MIN('TablaCrimenes'[Fecha]), MIN('TablaQuejas'[Fecha]))
VAR FechaMax = MAX(MAX('TablaCrimenes'[Fecha]), MAX('TablaQuejas'[Fecha]))
RETURN
CALENDAR(FechaMin, FechaMax)
```

### Borough Dimension (`Dim_Barrio`)

Extracts a unique, unified list of boroughs from both fact tables to act as a central filter.

```dax
Dim_Barrio = 
DISTINCT(
    UNION(
        VALUES('TablaCrimenes'[Barrio]),
        VALUES('TablaQuejas'[Barrio])
    )
)
```

### Zone Dimension (`Maestra_Zonas`)

Creates a unified mapping table for specific zones/precincts.

```dax
Maestra_Zonas = 
DISTINCT(
    UNION(
        SELECTCOLUMNS('TablaCrimenes', "ID_Zona", 'TablaCrimenes'[ID_Zona]),
        SELECTCOLUMNS('TablaQuejas', "ID_Zona", 'TablaQuejas'[ID_Zona])
    )
)
```

---

## 3. Base Metrics

Standard row counts for the baseline comparison.

```dax
Recuento de Crimenes = COUNTROWS(TablaCrimenes)
Recuento de Quejas   = COUNTROWS(TablaQuejas)
```