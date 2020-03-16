# Product Dynamics API

## Requests

The overall request layout looks like the following example:
```
{
      "DATASET_HANDLE" : "?",
      "MODEL" : "baseline",
      "ID" : [351],
      "START_DATE" : "2018-12-16",
      "STEPS" : 5,
      "RETURN_Y_TEST" : 1,
      "BENCHMARK" : 1
}
```

The below table explains each and every parameter

| **Key** | **Must** | **Descripion** | **Default** | **Default behavior** | **Non-default behavior** |
| --- | --- | --- | --- | --- | --- |
| `DATASET_HANDLE` | No | Select or query info about available datasets | &quot;?&quot; | Display default dataset info | Select particular non-default dataset |
| `MODEL` | YES | Select model or add baselines to results | &quot;baseline&quot; | Add most important baselines to output | Select particular Deep Learning model other than the default one |
| `ID` | YES | Select the target time-series | [] | No default. Must supply value | Select particular timeseries or a individual one  [10,102,103] |
| `START_DATE` | YES | Select the start-date for the time-series prediction (the date considered as the 1st day of predicted future) | &quot;&quot; | No default. Must supply value | Start date  &quot;YYYY-MM-DD&quot; |
| `STEPS` | YES | Number of &quot;steps&quot; to predict. Models are step-agnostics so &quot;step&quot; can mean anything (day, week, half-month, month, etc) depending solely business decision |   | No default. Must supply value | Number of steps |
| `RETURN_Y_TEST` | NO | Return in payload the actual ground-truth if the prediction is actually done on backlog data | 1 | Will return the info and also a &quot;Y\_PAST&quot; list of &quot;STEPS \* 2&quot; | - |
| `Y_PAST` | NO | Will return the past data. | 1 |   |   |
| `BENCHMARK` | NO | Return full benchmark information including data from baselines WARNING: This feature works only if real data is available for prediction period | 1 | Will return the benchmark info |   |


## Responses

| **Key** | **Purpose** | **Results description** | **Children** |
| --- | --- | --- | --- |
| `DATASET\_INFO` | The full info regarding default dataset info if requested | Start date&quot;present date&quot;Max future prediction dateNumber of available seriesTotal &quot;prepared&quot; days from first historical date to the future max prediction date | No |
| `RESULTS` | The full result including any baselines if available | Each model including the baselines as a dictionary item | Each model has its own &quot;DESC&quot; description and &quot;PRE&quot; list of predictied values based on &quot;START\_DATE&quot; and &quot;STEPS&quot; |
| `Y_TEST` | The &quot;STEPS&quot; ground truth (reality) if available and requested using &quot;RETURN\_Y\_TEST&quot; . | Actual values in the real time-series |   |
| `BENCHMARK` | The benchmarking information if it was requested with &quot;BENCHMARK&quot;.  WARNING: This feature works only if real data is available for prediction period | Multi-level benchmarking information | Yes |
| `BENCHMARK / INFO` | The meta-data that explains the benchmark indicators further used | Key-value paris. Key indicator is labeled as &quot;Main performance indicator&quot; |   |
| `BENCHMARK / BEST_BASELINES` | Top proposed baselines for the requested data series | List of strings. Baseline names – can be looked up in &quot;RESULTS&quot; |   |
| `BENCHMARK / STATS` | Given all the baselines and the top model we analyze the key performance indicators defined by &quot;INFO&quot; | One dict for every model. The KPI and stats | Key-value pairs |
| `BENCHMARK / SERIES` | Aggregated data (whole target period values) for all models (including baselines) for the given series | Key: dict or Key: valueModels with their values and also a key &quot;\_\_TARGET\_\_&quot; for the actual sum of the predicted period  | For all models we have info for each individual series:PRED\_QTYCONFIDENCEG&amp;I (Good and Important – see &quot;INFO&quot;)  |