# Overview

This project analyzes Chicago taxi trips for the year 2023 to uncover insights related to revenue trends, operational efficiency, demand patterns, payment methods, and temporal fluctuations. The analysis involves extracting data using BigQuery, performing ETL processes in Python, and preparing data for visualization in Tableau.

![Chicago-taxi-dashboard](https://github.com/user-attachments/assets/74bd8294-7f35-4975-9c44-20d85348e3fc)

# Analysis

For a detailed breakdown of insights, please see [Insights](./insights.md).

## Key Highlights:

Data Source: Chicago Taxi Trips Public Dataset ([dataset](https://console.cloud.google.com/marketplace/product/city-of-chicago-public-data/chicago-taxi-trips?hl=en&organizationId=0&project=velvety-outcome-444601-r0))

Technologies: Google BigQuery, SQL, Python (Pandas), Google Colab, Tableau

Focus Areas: Revenue/Profitability, Demand Patterns, Payment Methods, Geospatial Efficiency, Temporal Trends


# Data Extraction (BigQuery)
We accessed the raw data from Google BigQuery. Monthly subsets of the 2023 dataset were queried, downloaded as CSVs, and stored in Google Drive.

## Sample SQL Query (for January 2023):
```
sql SELECT unique_key, taxi_id, trip_start_timestamp, trip_end_timestamp, trip_seconds, trip_miles, pickup_census_tract, dropoff_census_tract, pickup_community_area, dropoff_community_area, fare, tips, tolls, extras, trip_total, payment_type, company, pickup_latitude, pickup_longitude, pickup_location, dropoff_latitude, dropoff_longitude, dropoff_location
FROM bigquery-public-data.chicago_taxi_trips.taxi_trips
WHERE trip_start_timestamp >= '2023-01-01 00:00:00' AND trip_start_timestamp < '2023-02-01 00:00:00'; 
```

We repeated this query for each month (February through December 2023).

## 1) Load Data
In Google Colab, we authenticated to Google Drive, listed all monthly CSV files, and combined them into a single Pandas DataFrame.

## 2) ETL (Extract, Transform, Load) and Data Segmentation
After loading the raw data, we performed ETL steps to clean and preprocess the dataset. We then created separate datasets for different analytical segments, ensuring transformations for one segment do not affect the others.

### 2.1 Revenue and Profitability Analysis
Purpose: Understand how trip distance and time relate to revenue, and examine tipping behavior.

```
revenue_data = all_data.copy()
revenue_data = revenue_data.dropna(subset=['trip_total', 'trip_miles', 'trip_seconds'])
revenue_data = revenue_data[(revenue_data['trip_miles'] > 0) & (revenue_data['trip_total'] > 0)]

# Create new metrics to assess profitability and tipping:
# 1. revenue_per_mile: How much revenue is generated per mile traveled.
# 2. revenue_per_minute: How efficiently revenue is generated over time (per minute).
# 3. tip_percentage: The proportion of the fare that comes from tips, reflecting customer tipping behavior.

revenue_data['revenue_per_mile'] = revenue_data['trip_total'] / revenue_data['trip_miles']
revenue_data['revenue_per_minute'] = revenue_data['trip_total'] / (revenue_data['trip_seconds'] / 60)
revenue_data['tip_percentage'] = (revenue_data['tips'] / revenue_data['fare']) * 100
```

By introducing these new metrics, we can compare trips not only by their total revenue but also by how efficiently that revenue is earned (distance vs. time) and gauge customer tipping patterns. These insights help identify which types of trips are more profitable and where improvements or marketing strategies might be beneficial.

### 2.2 Demand Patterns by Time and Location
Purpose: Identify when and where demand is highest.

```
python demand_data = all_data.copy()
demand_data = demand_data.dropna(subset=['trip_start_timestamp'])
demand_data['trip_start_timestamp'] = pd.to_datetime(demand_data['trip_start_timestamp'])
demand_data = demand_data.dropna(subset=['pickup_latitude', 'pickup_longitude'])
demand_data = demand_data[demand_data['pickup_latitude'].between(-90, 90) & demand_data['pickup_longitude'].between(-180, 180)] demand_data['hour_of_day'] = demand_data['trip_start_timestamp'].dt.hour demand_data['day_of_week'] = demand_data['trip_start_timestamp'].dt.day_name() demand_data['month'] = demand_data['trip_start_timestamp'].dt.month_name() 
```

### 2.3 Payment Method Analysis
Purpose: Compare cash vs. card to see how payment influences revenue and tipping.

```
python payment_data = all_data.copy()
payment_data = payment_data.dropna(subset=['payment_type', 'fare', 'tips'])
payment_data = payment_data[payment_data['fare'] > 0]

payment_summary = payment_data.groupby('payment_type').agg({ 'tips': 'mean', 'trip_total': 'sum', 'fare': 'mean' }).reset_index() payment_summary['avg_tip_percentage'] = (payment_summary['tips'] / payment_summary['fare']) * 100 
```

### 2.4 Geospatial Trip Efficiency
Purpose: Assess how trip efficiency varies by location.

```
python geospatial_data = all_data.copy()
geospatial_data = geospatial_data.dropna(subset=['trip_miles', 'fare', 'pickup_latitude', 'pickup_longitude']) geospatial_data = geospatial_data[(geospatial_data['trip_miles'] > 0) & (geospatial_data['pickup_latitude'].between(-90, 90)) & (geospatial_data['pickup_longitude'].between(-180, 180))] geospatial_data['efficiency_ratio'] = geospatial_data['fare'] / geospatial_data['trip_miles'] geospatial_data['trip_duration_minutes'] = geospatial_data['trip_seconds'] / 60
```

### 2.5 Temporal Revenue Trends
Purpose: Examine daily and hourly revenue patterns.

```
python temporal_data = all_data.copy()
temporal_data = temporal_data.dropna(subset=['trip_start_timestamp','trip_total'])
temporal_data['trip_start_timestamp'] = pd.to_datetime(temporal_data['trip_start_timestamp'])
temporal_data['trip_date'] = temporal_data['trip_start_timestamp'].dt.date

daily_revenue = temporal_data.groupby('trip_date')['trip_total'].sum().reset_index()
hourly_revenue = temporal_data.groupby(temporal_data['trip_start_timestamp'].dt.hour)['trip_total'].sum().reset_index()
```

## 3) Download Prepared Data
After ETL and segmentation, we saved the cleaned and enriched datasets for use in Tableau.

```
python revenue_data.to_csv('/content/drive/My Drive/Revenue_Data.csv', index=False)
demand_data.to_csv('/content/drive/My Drive/Demand_Data.csv', index=False)
payment_summary.to_csv('/content/drive/My Drive/Payment_Summary.csv', index=False)
geospatial_data.to_csv('/content/drive/My Drive/Geospatial_Data.csv', index=False)
daily_revenue.to_csv('/content/drive/My Drive/Daily_Revenue.csv', index=False)
hourly_revenue.to_csv('/content/drive/My Drive/Hourly_Revenue.csv', index=False)
```

# Next Steps
Visualization in Tableau: Import these CSVs into Tableau to create interactive dashboards.
Analysis and Insights: Explore revenue patterns, identify high-demand zones, compare payment methods, and examine temporal trends.


Analysis Link: [Insights](./insights.md)
