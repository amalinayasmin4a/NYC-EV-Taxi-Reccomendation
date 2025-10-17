NYC-EV-Taxi-Recommendation

Based on the NYC Taxi Performance Data, this project evaluates the operational and pricing performance of NYC taxis to identify opportunities for efficiency improvement and support a cost-effective transition to electric vehicles (EVs).

Data Cleaning Process
1. Data Import & Initial Inspection

Imported the dataset NYC_TLC_Cleaned_for_Report.xlsx.

Conducted initial inspection using:

df.info(), df.head(), df.describe()


Checked dataset dimensions (df.shape) and verified data types.

2. Data Cleaning
a. Datetime Conversion

Converted pickup and dropoff columns to datetime format:

df['lpep_pickup_datetime'] = pd.to_datetime(df['lpep_pickup_datetime'], errors='coerce')
df['lpep_dropoff_datetime'] = pd.to_datetime(df['lpep_dropoff_datetime'], errors='coerce')

b. Trip Duration Calculation

Calculated trip duration in minutes:

df['trip_duration_minutes'] = (df['lpep_dropoff_datetime'] - df['lpep_pickup_datetime']).dt.total_seconds() / 60


Removed invalid or zero-duration trips:

df = df[df['trip_duration_minutes'] > 0]

3. Handling Missing Values

Checked for missing data using df.isna().sum().

Filled missing categorical data (e.g., RatecodeID) using mode or logical imputations.

4. Outlier Detection & Removal
a. Distance–Fare Anomalies

Removed inconsistent entries:

df = df[(df['trip_distance'] > 0.1)]
df = df[~((df['trip_distance'] < 2) & (df['fare_amount'] > 40))]
df = df[~((df['trip_distance'] > 5) & (df['fare_amount'] > 100))]

b. Fare-per-Mile Ratio Filter

Created a ratio feature:

df['fare_per_mile'] = df['fare_amount'] / df['trip_distance']


Removed extreme values:

df = df[(df['fare_per_mile'] >= 1.5) & (df['fare_per_mile'] <= 2 * df['fare_per_mile'].median())]

c. Trip Duration Outliers

Filtered unrealistic durations:

df = df[df['trip_duration_minutes'] <= 180]

d. Speed-Based Filtering (Optional)

Calculated average speed and removed implausible values:

df['avg_speed_mph'] = df['trip_distance'] / (df['trip_duration_minutes'] / 60)
df = df[(df['avg_speed_mph'] > 1) & (df['avg_speed_mph'] < 100)]

5. Feature Engineering

Created additional features for business and EV analysis:

fare_per_mile → pricing efficiency

avg_speed_mph → operational efficiency

distance_category → trip classification for EV suitability

Example function:

def categorize_distance(mi):
    if mi < 2: return "0–2 mi"
    elif mi < 5: return "2–5 mi"
    elif mi < 10: return "5–10 mi"
    elif mi < 20: return "10–20 mi"
    elif mi < 40: return "20–40 mi"
    else: return "40+ mi"

df['distance_category'] = df['trip_distance'].apply(categorize_distance)

6. Data Validation

Validated cleaning results visually using:

Scatterplot: trip_distance vs fare_amount

Boxplot: fare_per_mile by distance_category

Histogram: distribution of trip_duration_minutes

Geospatial checks: validated pickup locations using NYC Taxi Zone mapping

7. Clean Data Export

Saved the final cleaned and validated dataset:

df.to_excel("NYC_TLC_Cleaned_EV_Ready.xlsx", index=False)

Summary of Results

Produced a clean, analysis-ready dataset with valid and consistent records.

Removed anomalies, unrealistic fares, and trip irregularities.

Added business-relevant analytical features for EV adoption analysis and fare optimization.

Visualization

For business insights, the following visualizations were used:

Bar plots to analyze temporal and categorical trends.

Pie charts to represent distribution and proportion.

Heatmaps and geospatial maps for location-based performance insights.
