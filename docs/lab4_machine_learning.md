# Exercise 4 - Utilizing Machine Learning for Call Forecasting and Anomaly Detection

## Objective

Implement advanced machine learning capabilities in Splunk to predict call volumes, detect anomalies, and provide intelligent insights for contact center optimization.

## Overview

This exercise demonstrates how to leverage Splunk's Machine Learning Toolkit (MLTK) to:

- Build predictive models for call forecasting
- Detect unusual patterns and anomalies
- Enable data-driven decision making

## Prerequisites

1. **Install Splunk MLTK**
   - Navigate to **Apps** > **Find More Apps**
   - Search for "Splunk Machine Learning Toolkit"
   - Install and reload Splunk

---

## Part A: Build Call Forecasting Model Based on Historical Data

### Step 2: Prepare Historical Data for Forecasting

### Step 3: Build Time Series Forecasting Model

1. **Access MLTK Forecasting**

   - Navigate to **Machine Learning Toolkit** app
   - Click **Experiment** > **Forecast Time Series**
   - Click Create New Experiment
   - Select **Forecast Time Series** as experiment type and enter any experiment title

2. **Configure Experiment Settings**
   - Enter SPL

```spl
index="cce_demo"
| eval _time = strptime(DateTime, "%Y-%m-%d %H:%M:%S")
| timechart span=15m sum(CallsOffered) as TotalCallsOffered
| makecontinuous _time span=15m
| fillnull value=0
```

- Under Algorithm Select **Kalman Filter**
- Select **TotalCallsOffered** in Field to forecast
- Select **LLP5** as method
- Enter 700 for **Future Timespan** and **Holdback**
- Enter confidence interval as 95
- Enter 672 as period

<figure markdown>
  ![Forecast Configuration](./assets/forecast_configuration.png){ width="800" }
  <figcaption>Screenshot showing time series forecast configuration</figcaption>
</figure>

### Step 4: Generate and Validate Forecast

1. **Run Forecasting Model**

   - Click on **Forecast** Button
   - Observe forecasting results are shown as in image

<figure markdown>
  ![Forecast Results](./assets/forecast_results.png){ width="800" }
  <figcaption>Screenshot showing forecast results visualization</figcaption>
</figure>

### Step 5: Schedule Forecast Retraining

1. **Schedule Configuration**
   - Click on **Open in Search** Buton
   - Ensure the SPL shows the parameter selected
   - Click on **Save As** Report
   - Enter the title as "Scheduled Call Forecast Training"
   - In the next popup click on **Schedule**
   - Click on the **Schedule Report** and configure it to **Run Every Week** **Monday** **6:00**
   - Click Save

<figure markdown>
  ![Scheduled Training](./assets/scheduled_training.png){ width="800" }
  <figcaption>Screenshot showing scheduled model retraining configuration</figcaption>
</figure>

---

## Part B: Anomaly Detection for Call Patterns

### Step 1: Configure Numeric Outlier Detection

1. **Access MLTK Experiments**
   - Navigate to **Machine Learning Toolkit** > **Experiments**
   - Click **Create New Experiment**
   - Choose **Detect Numeric Outliers** as experiment type and enter any experiment title

<figure markdown>
  ![Outlier Experiment Creation](./assets/outlier_experiment_creation.png){ width="800" }
  <figcaption>Screenshot showing numeric outlier detection experiment creation</figcaption>
</figure>

### Step 2: Configure Experiment Settings

1. **Enter SPL Query for Anomaly Detection**

```spl
index="cce_demo"
| eval _time = strptime(DateTime, "%Y-%m-%d %H:%M:%S")
| timechart span=15m sum(CallsOffered) as CallsOffered by SkillGroup
| untable _time SkillGroup CallsOffered
| streamstats window=96 avg(CallsOffered) as avg stdev(CallsOffered) as stdev by SkillGroup
| eval upper_bound = avg + (3 * stdev)
| eval is_anomaly = if(CallsOffered > upper_bound AND CallsOffered > 50, 1, 0)
```

and click on Search

2. **Configure Outlier Detection Parameters**

- Select **CallsOffered** as the field to analyze for outliers
- Under **Threshold Method** select **Standard Deviation**
- Set threshold multiplier to **2**
- Select **SkillGroup** as the field to split by

<figure markdown>
  ![Outlier Detection Configuration](./assets/outlier_detection_config.png){ width="800" }
  <figcaption>Screenshot showing numeric outlier detection configuration</figcaption>
</figure>

### Step 3: Run Outlier Detection Analysis

1. **Execute Outlier Detection**
   - Click on **Detect Outliers** Button
   - Observe anomaly detection results showing skill groups with abnormal call patterns
   - Review the outlier scores and identified anomalies

<figure markdown>
  ![Outlier Detection Results](./assets/outlier_detection_results.png){ width="800" }
  <figcaption>Screenshot showing outlier detection results visualization</figcaption>
</figure>

### Step 4: Schedule Anomaly Detection Retraining

1. **Create Scheduled Report**
   - Click on **Open in Search** Button
   - Ensure the SPL shows the parameters selected in the experiment
   - Click on **Save As** Report
   - Enter the title as "Scheduled Call Pattern Anomaly Detection"
   - In the next popup click on **Schedule**
   - Click on the **Schedule Report** and configure it to **Run Every Day** at **2:00 AM**
   - Click Save

<figure markdown>
  ![Scheduled Anomaly Detection](./assets/scheduled_anomaly_detection.png){ width="800" }
  <figcaption>Screenshot showing scheduled anomaly detection retraining</figcaption>
</figure>

---

## Summary

### What You've Accomplished

- ✅ Implemented time series forecasting using Splunk MLTK for call volume prediction
- ✅ Built automated anomaly detection system using LocalOutlierFactor algorithm
- ✅ Created scheduled model retraining for both forecasting and anomaly detection

### Key Benefits

- **Predictive Intelligence**: Forecast call volumes 3 days in advance for better planning
- **Automated Anomaly Detection**: Real-time identification of unusual call patterns
- **Self-Maintaining Models**: Scheduled retraining keeps models current with data trends
- **Proactive Management**: Early warning system for operational issues
