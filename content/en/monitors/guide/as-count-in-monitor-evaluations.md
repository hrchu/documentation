---
title: as_count() in Monitor Evaluations
kind: guide
aliases:
  - /monitors/guide/as-count-monitor-evaluation
---

## Overview

Queries using **`as_count()`** and **`as_rate()`** modifiers are calculated in ways that can yield different results in monitor evaluations. Monitors involving arithmetic and at least 1 **`as_count()`** modifier use a separate evaluation path that changes the order in which arithmetic and time aggregation are performed.

## Error Rate Example

Suppose you want to monitor an error rate over 5 minutes using the metrics, `requests.error` and `requests.total`. Consider a single evaluation performed with these aligned timeseries points for the 5 min timeframe:

**Numerator**: `sum:requests.error{*}`

```
| Timestamp             | Value       |
| :-------------------- | :---------- |
| 2018-03-13 11:00:30   | 1           |
| 2018-03-13 11:01:30   | 2           |
| 2018-03-13 11:02:40   | 3           |
| 2018-03-13 11:03:30   | 4           |
| 2018-03-13 11:04:40   | 5           |
```

**Denominator**: `sum:requests.total{*}`

```
| Timestamp             | Value    |
| :-------------------- | :------- |
| 2018-03-13 11:00:30   | 5        |
| 2018-03-13 11:01:30   | 5        |
| 2018-03-13 11:02:40   | 5        |
| 2018-03-13 11:03:30   | 5        |
| 2018-03-13 11:04:40   | 5        |
```

### 2 ways to calculate

Refer to this query as **`classic_eval_path`**:

```
sum(last_5m): sum:requests.error{*}.as_rate() / sum:requests.total{*}.as_rate()
```

and this query as **`as_count_eval_path`**:

```
sum(last_5m): sum:requests.error{*}.as_count() / sum:requests.total{*}.as_count()
```

Compare the result of the evaluation depending on the path:

| Path                     | Behavior                                       | Expanded expression         | Result  |
| :----------------------- | :--------------------------------------------- | :-------------------------- | :------ |
| **`classic_eval_path`**  | Aggregation function applied _after_ division  | **(1/5 + 2/5 + ... + 5/5)** | **3**   |
| **`as_count_eval_path`** | Aggregation function applied _before_ division | **(1+2+...+5)/(5+5+...+5)** | **0.6** |

_Note that both evaluations above are mathematically correct. Choose a method that suits your intentions._

It may be helpful visualize the **`classic_eval_path`** as:

```       
sum(last_5m):error/total
```

and the **`as_count_eval_path`** as:

```
sum(last_5m):error
-----------------
sum(last_5m):total
```

In general, **`avg`** time aggregation with **`.as_rate()`** is reasonable, but **`sum`** aggregation with **`.as_count()`** is recommended for error rates. Aggregation methods other than **`sum`** (shown as _in total_ in-app) do not make sense to use with **`.as_count()`**.

**Note**: Aggregation methods other than sum (shown as in total in-app) cannot be used with `.as_count()`.

[Reach out to the Datadog support team][1] if you have any questions.


[1]: /help
