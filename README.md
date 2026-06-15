# Traffic Demand Prediction (Flipkart GRIDLOCK 2.0)

> **Achieved 91.09% prediction accuracy** using LightGBM, Spatial Clustering, Target Encoding, and Advanced Feature Engineering.

This project predicts road traffic demand using geospatial data, weather signals, road information, and time-based patterns.

The model predicts a demand value between **0 and 1**, where higher values represent higher traffic intensity.

---

## Project Stats

| Metric | Value |
|----------|----------|
| Model | LightGBM |
| Number of Trees | 2500 |
| Spatial Cluster Configurations | 6 |
| Target Encoding | 5-Fold Target Encoding |
| Prediction Range | 0 – 1 |
| Recency Weighting | Day 49 receives higher weight |

---

## Input Data

Each row contains:

- Geohash
- Timestamp
- Road Type
- Number of Lanes
- Large Vehicle Permission
- Landmark Presence
- Weather Condition
- Temperature
- Day Information

### Output

Predicted traffic demand between **0 and 1**.

---

## Why Geohash is Important

Geohash stores location information in a compact string format.

The pipeline decodes geohash into:

- Latitude
- Longitude

This allows the model to understand the actual location of each traffic record.

Additional hierarchical geohash features are created:

- `geo3`
- `geo4`
- `geo5`

These represent different geographic scales, from broad regions to highly localized areas.

---

## Feature Engineering

### Time Features

Timestamp is converted into total minutes from midnight.

Cyclic features are created:

- `time_sin`
- `time_cos`
- `hour_sin`
- `hour_cos`

These help the model understand that time is circular. For example, **23:45 and 00:00 are close together**.

### Temperature Handling

Missing temperatures are filled using:

1. Median temperature for the same geohash and day.
2. Global median temperature if still missing.

A missing-value indicator is also created:

```text
temp_missing
```

This informs the model whether the original temperature value was unavailable.

### Spatial Coordinates

Each geohash is decoded into:

- Latitude
- Longitude

Decoded coordinates are cached to avoid repeated geohash decoding.

### Interaction Features

Examples include:

- `geohash + timestamp`
- `geohash + RoadType`
- `geohash + Weather`
- `geo4 + timestamp`
- `RoadType + timestamp`
- `cluster + timestamp`

These features help capture location-time and road-time traffic patterns.

---

## Spatial Clustering

Nearby locations often exhibit similar traffic behavior.

The model uses clustering to group nearby geohashes into spatial regions.

### KMeans Clustering

Configurations:

- 10 clusters
- 25 clusters
- 50 clusters

These represent broad, medium, and fine-grained spatial regions.

### Agglomerative Clustering

Configurations:

- 10 clusters
- 25 clusters
- 50 clusters

This hierarchical method merges nearby locations into progressively larger clusters.

### Why DBSCAN Was Removed

DBSCAN was evaluated but produced weaker and noisier validation results.

Final clustering features:

```text
kmeans_10
kmeans_25
kmeans_50
agglo_10
agglo_25
agglo_50
```

**Total cluster configurations: 6**

---

## Target Encoding

Target encoding converts categorical groups into their historical average demand values.

The pipeline uses **5-fold target encoding** to reduce target leakage.

Applied to features such as:

- Geohash
- Geohash + Timestamp
- Geohash + RoadType
- Geohash + Weather
- Geo4 + Timestamp
- RoadType + Timestamp
- Cluster + Timestamp

This helps the model learn historical demand behavior across locations, road types, and time periods.

---

## Early Morning Signal

A feature called:

```text
day_geo_last
```

stores the last available demand value before or at **2 AM** for each day and geohash.

This captures useful early-day traffic signals that can improve later demand predictions.

---

## Label Encoding

Categorical variables are converted into numerical representations.

Applied to:

- Geohash features
- Road Type
- Weather
- Landmark Information
- Large Vehicle Permission
- Cluster Labels
- Interaction Features

---

## LightGBM Model

LightGBM is a gradient boosting framework that builds decision trees sequentially, with each tree correcting errors from previous trees.

### Model Parameters

| Parameter | Value |
|------------|------------|
| Number of Trees | 2500 |
| Learning Rate | 0.015 |
| Leaves per Tree | 63 |
| Max Depth | Unlimited |
| Min Samples per Leaf | 35 |
| Row Sampling | 85% |
| Column Sampling | 85% |
| L1 Regularization | 0.4 |
| L2 Regularization | 2.0 |
| Random Seed | 42 |

These settings prioritize generalization and reduced overfitting.

---

## Sample Weighting

More recent observations are given higher importance.

| Day | Weight |
|------|------|
| Day 49 | 3.0 |
| Other Days | 1.0 |

This helps the model focus more strongly on data closest to the test period.

---

## Prediction

After training, the model predicts traffic demand for test records.

Predictions are clipped to remain within:

```text
Minimum = 0
Maximum = 1
```

The final predictions are exported as a CSV submission file.

---

## Sanity Checks

The pipeline reports:

- Minimum Prediction
- Maximum Prediction
- Mean Prediction
- Standard Deviation
- Fraction of Near-Zero Predictions
- Fraction Above 0.5
- Fraction Above 0.8

These checks help identify abnormal prediction distributions.

---

## End-to-End Pipeline

1. Load train and test CSV files
2. Convert timestamps into numeric and cyclic features
3. Create geohash hierarchy features
4. Fill missing temperatures
5. Decode geohash into latitude and longitude
6. Create KMeans and Agglomerative clusters
7. Generate interaction features
8. Apply 5-fold target encoding
9. Add early-morning Day 49 signal
10. Label encode categorical features
11. Train LightGBM with sample weights
12. Predict traffic demand
13. Clip predictions to [0, 1]
14. Save submission CSV
15. Print prediction sanity checks

---

## Key Design Decisions

- Traffic demand is highly location-dependent, making geohash features essential.
- Multiple geohash resolutions capture both regional and local behavior.
- KMeans and Agglomerative clustering capture spatial traffic patterns.
- DBSCAN was removed due to poorer validation performance.
- Target encoding captures historical demand trends.
- Cyclic encoding properly represents repeating daily time patterns.
- Recent data receives higher weight to improve future prediction accuracy.
- LightGBM was selected for its strong performance on structured tabular datasets.

---

## Summary

This pipeline predicts traffic demand by combining:

- Exact location behavior
- Regional clustering
- Temporal patterns
- Road context
- Weather signals
- Historical demand encoding
- Recency-weighted training

The central idea is that traffic demand is driven by the interaction of **location, time, weather, and road characteristics**, and the model leverages all of these signals to generate accurate demand predictions.
