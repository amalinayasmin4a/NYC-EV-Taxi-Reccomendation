# NYC-EV-Taxi-Reccomendation
Based on the NYC Taxi Performance Data, I have created a deck to evaluate their performance and operational base to then suggest solution to optimize their efficiency
#Cleaning
1️. Data Import & Initial Inspection

Imported the dataset NYC_TLC_Cleaned_for_Report.xlsx.

Used pandas to inspect with:

df.info(), df.head(), df.describe()


Checked basic dimensions (df.shape) and data types.

2️. Data Cleaning
a. Datetime Conversion

Converted pickup and dropoff times to datetime objects:

df['lpep_pickup_datetime'] = pd.to_datetime(df['lpep_pickup_datetime'], errors='coerce')
df['lpep_dropoff_datetime'] = pd.to_datetime(df['lpep_dropoff_datetime'], errors='coerce')

b. Trip Duration Calculation

Created trip duration in minutes:

df['trip_duration_minutes'] = (df['lpep_dropoff_datetime'] - df['lpep_pickup_datetime']).dt.total_seconds() / 60


Filtered invalid durations:

df = df[df['trip_duration_minutes'] > 0]

3️. Handling Missing Values

Checked missing data using:

df.isna().sum()


Filled or imputed critical categorical columns (e.g., RatecodeID) using mode or logical assumptions.

4️. Outlier Detection & Removal
a. Distance–Fare Anomalies

Removed cases where:

Short distance but high fare

Long distance but low fare

df = df[(df['trip_distance'] > 0.1)]
df = df[~((df['trip_distance'] < 2) & (df['fare_amount'] > 40))]
df = df[~((df['trip_distance'] > 5) & (df['fare_amount'] > 100))]

b. Fare-per-Mile Ratio Filter

Added a new ratio:

df['fare_per_mile'] = df['fare_amount'] / df['trip_distance']


Filtered extreme values:

df = df[(df['fare_per_mile'] >= 1.5) & (df['fare_per_mile'] <= 2 * df['fare_per_mile'].median())]

c. Trip Duration Outliers

Removed unrealistic trips:

df = df[df['trip_duration_minutes'] <= 180]

d. Speed-Based Filtering (optional)

Calculated average speed:

df['avg_speed_mph'] = df['trip_distance'] / (df['trip_duration_minutes'] / 60)


Removed impossible speeds (<1 mph or >100 mph):

df = df[(df['avg_speed_mph'] > 1) & (df['avg_speed_mph'] < 100)]

5️. Feature Engineering

Created new analytical columns for business insights:

fare_per_mile → pricing efficiency

avg_speed_mph → operational efficiency

distance_category → EV suitability classification

Example:

def categorize_distance(mi):
    if mi < 2: return "0–2 mi"
    elif mi < 5: return "2–5 mi"
    elif mi < 10: return "5–10 mi"
    elif mi < 20: return "10–20 mi"
    elif mi < 40: return "20–40 mi"
    else: return "40+ mi"

df['distance_category'] = df['trip_distance'].apply(categorize_distance)

6️. Data Validation

Validated results visually:

Scatterplot: trip_distance vs fare_amount

Boxplot: fare_per_mile by distance_category

Histogram: distribution of trip_duration_minutes

Geographic check (via PULocationID → NYC Taxi Zones mapping)

7️.  Clean Data Export

Saved the clean, analysis-ready dataset:

df.to_excel("NYC_TLC_Cleaned_EV_Ready.xlsx", index=False)

*Summary of Results*



1.  Cleaned and validated X rows of quality data
2. Removed anomalies and extreme fares
3. Added business-relevant features for EV analysis and pricing optimization
*Visualization*
for Business Insight Visualization there is Bar Plot for trend, Pie Chart for proportion, and Maps for heatmap
