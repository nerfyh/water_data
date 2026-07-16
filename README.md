# Water: Upper Ocmulgee Dissolved-Oxygen Dataset

Water is a multivariate time-series dataset compiled from dissolved-oxygen records made available by the U.S. Geological Survey (USGS). The model-ready file used in the SDPF experiments is [usgs_upper_ocmulgee_do10_15min.csv](./usgs_upper_ocmulgee_do10_15min.csv).

## Dataset summary

| Item | Value |
|---|---|
| Region | Upper Ocmulgee and South River, Georgia, USA |
| USGS hydrologic unit | HUC `03070103` |
| Measurement | Dissolved oxygen, unfiltered water, mg/L |
| USGS parameter code | `00300` |
| Time span | 2021-01-01 00:00:00 to 2024-12-31 23:45:00 UTC |
| Sampling interval | 15 minutes |
| Time steps | 140,256 |
| Variables | 10 monitoring stations |
| Missing values in released CSV | 0 |
| Suggested split | Chronological 6:2:2 |

The source records were obtained from the [USGS NWIS Instantaneous Values service](https://waterservices.usgs.gov/nwis/iv/). The released CSV is a processed benchmark panel compiled from those records, not a standalone dataset originally released by USGS under the name `Water`.

## Columns

The `date` column contains UTC timestamps. The remaining columns contain dissolved-oxygen measurements in mg/L.

| CSV column | USGS site number |
|---|---|
| `var_000` | `02207160` |
| `var_001` | `02207135` |
| `var_002` | `02208493` |
| `var_003` | `02203831` |
| `var_004` | `02203873` |
| `var_005` | `02203950` |
| `var_006` | `02203900` |
| `var_007` | `02203655` |
| `var_008` | `02208450` |
| `OT` | `02204037` |

`OT` is the conventional final-column name used by common long-term forecasting data loaders. It represents one of the ten station variables rather than a separately constructed target.

## Preprocessing

The USGS observations were aligned to a continuous 15-minute UTC grid without hourly aggregation. Missing values were completed independently for each station as follows:

1. Missing runs of at most 96 steps were linearly interpolated within the train, validation, and test segments separately.
2. Remaining gaps were filled using month-by-time-slot medians computed from the training segment.
3. Training time-slot and station medians were retained as fallbacks.

No cross-station filling was used. Source observations account for 99.1544% of the released values, while 0.8456% were completed by the procedure above. The model-ready CSV should therefore be treated as a completed benchmark panel rather than a raw observation archive.

## Loading

```python
import pandas as pd

data = pd.read_csv(
    "usgs_upper_ocmulgee_do10_15min.csv",
    parse_dates=["date"],
)
timestamps = data["date"]
values = data.drop(columns="date").to_numpy(dtype="float32")

assert values.shape == (140256, 10)
```

For the paper setting, split the rows chronologically into 84,153 training rows, 28,051 validation rows, and 28,052 test rows. Do not shuffle before splitting.

## Integrity

```text
SHA-256: c34a9869db8b37b9bde86fec09e21155fc8a2153a3db316c74ec516acb7df607
```

When using or redistributing the dataset, acknowledge USGS NWIS as the source of the underlying observations.
