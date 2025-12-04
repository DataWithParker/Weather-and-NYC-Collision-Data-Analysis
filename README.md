# NYC Weather & Traffic Crash Analysis (2020–2025)

*Course: IST 718 – Big Data Analytics - Syracuse University*

This project investigates how weather and environmental conditions influence the number of hourly traffic crashes across New York City boroughs (2020–2025). Using large-scale crash and weather datasets aligned in PySpark, a Poisson regression model was built to quantify how conditions like rain, snow, visibility, and time-of-day shape crash frequency.

---

## **Problem Statement**
**How do weather and environmental conditions influence the number of crashes per hour in each NYC borough (2020–2025)?**

This project aims to identify which conditions most strongly increase crash risk and how hourly patterns vary across boroughs.

---

## **Dataset Overview**

### Original NYC Crash Data
- **Source:** https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95/about_data
- **Description:** The Motor Vehicle Collisions crash table contains details on the crash event. Each row represents a crash event. The Motor Vehicle Collisions data tables contain information from all police reported motor vehicle collisions in NYC. The police report (MV104-AN) is required to be filled out for collisions where someone is injured or killed, or where there is at least $1000 worth of damage.
- **Size:** 2,212,912 rows, 29 columns
- **Date Range:** *2012-07-01 to 2025-10-11*

#### **Crash Data Used in Model**
- **Years Used:** 2020–2025  
- **Rows after cleaning:** 398,266
- **Key Fields**
  - `crash_datetime`
  - `borough`
  - `zip_code`
  - `crash_nearest_hour` *(engineered; aligns with weather)*
---

### **Weather Data**
- **Visual Crossing API**: https://www.visualcrossing.com
- **Hourly weather observations matched to each crash**
- **Key Fields**
  - `weather_datetime`
  - `feelslike`, `humidity`, `dew`
  - `precip`, `snow`, `snowdepth`
  - `windspeed`, `winddir`, `pressure`
  - `visibility`, `cloudcover`
  - `conditions` (text label)

---

## **Feature Engineering**

### **1. Conditions to Binary Flags**
The original `conditions` column contained **12 unique text labels** (e.g., “Rain”, “Rain, Overcast”, “Overcast, Rain”).  
These were consolidated into **5 interpretable binary indicators**:
- `cond_rain`
- `cond_snow`
- `cond_cloudy`
- `cond_overcast`
- `cond_clear`

**Reason:** Poisson regression requires numeric input, and grouping similar conditions improves interpretability & model stability.

---

### **2. Time-Based Features**
Extracted from `crash_datetime`:
- `hour_of_day`
- `day_of_week`
- `month_of_year`

**Reason:** Traffic exposure varies by time (rush hour, weekends, seasonality).

---

### **3. Crash–Weather Alignment**
Each crash was matched to the nearest hourly weather reading using PySpark window operations.

---

## **Methodology**

### **Model Used: Poisson Regression**
Chosen because crash counts are:

- Non-negative integers  
- Skewed (many low-count hours, few high-count hours)  
- Best modeled as rate-based events  

### **PySpark MLlib**
All modeling and feature engineering were completed in **PySpark**, enabling scalable processing of ~400k+ hourly crash–weather pairs.

---

## **Model Fit Indicators**

| Metric | Value | Interpretation |
|--------|--------|----------------|
| **Null Deviance** | 480,379 | Error with no predictors |
| **Residual Deviance** | 458,604 | Error after adding predictors |
| **Deviance Reduction** | Significant | Predictors explain real variation |
| **Overdispersion Ratio** | **1.12** | Close to 1, therefore Poisson reasonable |

**Summary:**  
The deviance drop indicates the predictors are meaningful. The overdispersion ratio shows the Poisson assumptions fit the data well.

---

## **Key Results: Risk Multipliers**

Risk multipliers convert Poisson coefficients (originally in log) into rate changes using exp(), so they are easy to interpret.

### **Top Positive Predictors**
| Feature | Crash Rate Change |
|--------|---------------------|
| **cond_rain** | **+12%** |
| **cond_snow** | **+10%** |
| **snow** | **+5%** |
| **hour_of_day** | **+2.3%** |
| **day_of_week** | **+1.7%** |

### **Risk-Reducing Predictors**
| Feature | Crash Rate Change |
|--------|---------------------|
| **cond_overcast** | **−25%** |
| **cond_clear** | **−20%** |
| **precip** | **−9.5%** |
| **visibility** | **−2.7%** |

**Interpretation:**  
Rain and snow shows a slight increase in the crash rate, while clear/overcast weather and higher visibility tend to reduce crash likelihood.

---

## **Visual Findings**

### **Poisson Distribution per Borough**
- Brooklyn (λ ≈ 3.18) has the highest curve shifted toward 3–4 crashes per hour, indicating it experiences the most frequent crashes.

### **Poisson Distribution vs. Empirical Crash Data**
- Borough crash patterns (Queens, Brooklyn, Bronx, etc.) follow Poisson-shaped curves.
- Empirical crash distribution matches the Poisson prediction closely.
- Confirms Poisson regression is statistically appropriate.

### **Observed vs. Predicted Scatter Plot**
- Model predicts typical crash rate (0–10 crashes) well.
- Underestimates rare high-crash hours (>10), a known limitation of Poisson models.

---

## **Conclusions**

- Weather conditions have a slight influence on crash counts.
- Rain (+12%) and snow (+10%) are the strongest drivers of increased crash frequency.
- Time-of-day and day-of-week patterns reflect predictable traffic behavior.
- PySpark enabled efficient processing and modeling of hundreds of thousands of hourly crash–weather pairs.
- Poisson regression is validated by:
  - Deviance reduction  
  - Overdispersion ratio (~1)  
  - Empirical distribution alignment  

**Overall:**  
This model effectively quantifies how weather and time conditions shape crash risk across NYC.

