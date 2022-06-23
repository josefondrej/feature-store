## Basic Feast API

**Field**

```
Field(
    name='rating', 
    dtype=int
)
```

**Entity**

```
Entity(
    name='driver', 
    value_type=str, 
    join_keys=['driver_id']
)
```

**FeatureView**

Tells us where (`source`) to find some data and in what format and what are the names of the features (`entities`
, `schema`).

```
FeatureView(
    name='driver_rating', 
    entities=['driver'],  # Optional
    schema=[Field(...), Field(...)],
    source=BigQuerySource(table='feast-oss.demo-data.global_stats'), 
    ttl=timedelta(hours=2) 
) 
```

`source` is a table of format

| timestamp  | driver_id | rating |
|------------|-----------|--------|
| 2022-01-05 | driver-01 | 1      |
| 2022-01-06 | driver-05 | 4      |
| ...        | ...       | ...    |

The `timestamp` field is required. The event timestamp describes the event time at which a feature was observed or
generated.
The `ttl` argument tells us how long do features in this view "live" -- for how long are they relevant for us.

### How it Works

Different `FeatureViews` (and other things, e.g. `FeatureService`) are defined in some github repository.
This is the **Feature Repo**. We then get the feature store by calling

```
feature_store = FeatureStore(repository=...)
```

We then use the feature store in the following way. When we have some entity dataframe, which can look like

| timestamp  | driver_id | 
|------------|-----------|
| 2022-01-05 | driver-01 |
| 2022-01-06 | driver-05 |
| ...        | ...       |

we can then join different feature views onto this dataframe (in point-in-time correct manner) like this:

```
training_df = feature_store.get_historical_features(
    entity_df=entity_df,
    features = [
        'driver_hourly_stats:conv_rate', <-- format is 'feature_view_name:feature_name' 
        'driver_hourly_stats:acc_rate',
        'driver_hourly_stats:avg_daily_trips'
    ],
).to_df()
```