---
title: Metric submission with DogStatsD
kind: documentation
description: Overview of the features of DogStatsD, including data types and tagging.
aliases:
  - /developers/faq/reduce-submission-rate
  - /developers/faq/why-is-my-counter-metric-showing-decimal-values
  - /developers/faq/dog-statsd-sample-rate-parameter-explained
further_reading:
- link: "developers/dogstatsd"
  tag: "Documentation"
  text: "Introduction to DogStatsD"
- link: "developers/metrics/metric_type"
  tag: "Documentation"
  text: "Discover Datadog metric types."
---

While StatsD accepts only metrics, DogStatsD accepts all three of the major Datadog data types: metrics, events, and Service Checks. This section shows typical use cases for Metrics split down by metric types, and introduces [Sampling Rate](#sample-rates) and [Metrics tagging](#metrics-tagging) options which are specific to DogStatsD.

[Count](#count), [Gauge](#gauge), and [Set](#set) metric types are familiar to StatsD users. Histograms are specific to DogStatsD. Timers, which exist in StatsD, are a sub-set of histograms in DogStatsD. Additionally you can also submit [Histogram](#histogram) and [Distribution](#distribution) metric types using DogStatsD.

**Note**: Depending of the submission method used, the submission metric type and the actual metric type stored within Datadog might differ

After having [installed DogStatsD][1], find below the functions available to submit your metrics to Datadog depending of their metric type.

## Count

| Method                                       | Description                                               | Storage type                                                                                                                                             |
| :-----                                       | :-------                                                  | :---                                                                                                                                                     |
| `increment(MetricName, SampleRate, Tags)`    | Used to increment a count metric.                         | Stored as a `RATE` type in Datadog. Each value in the stored timeseries is a time-normalized delta of the counter's value over that StatsD flush period. |
| `decrement(MetricName, SampleRate, Tags)`    | Used to decrement a count metric.                         | Stored as a `RATE` type in Datadog. Each value in the stored timeseries is a time-normalized delta of the counter's value over that StatsD flush period. |
| `count(MetricName, Value, SampleRate, Tags)` | Use to increment a count metric from an arbitrary `Value` | Stored as a `RATE` type in Datadog. Each value in the stored timeseries is a time-normalized delta of the counter's value over that StatsD flush period. |

with the following parameters:

| Parameter    | Type            | Required | Description                                                                                                                                                                |
| --------     | -------         | -----    | ----------                                                                                                                                                                 |
| `MetricName` | String          | yes      | Name of the metric to submit.                                                                                                                                              |
| `Value`      | Double          | yes      | Value associated to your metric.                                                                                                                                           |
| `SampleRate` | Double          | no       | Sample rate, between `0` (no sample) and `1` (all datapoints are dropped), to apply to this particular metric. See the [Sample Rate section](#sample-rates) to learn more. |
| `Tags`       | List of Strings | no       | List of Tags to apply to this particular metric. See the [Metrics Tagging](#metrics-tagging) section to learn more.                                                        |

Note: Count type metrics can show a decimal value within Datadog since they are normalized over the flush interval to report a per-second units.

Find below short code snippets depending of your language that you can run to emit a `COUNT` metric type-stored as `RATE` metric type-into Datadog. Learn more about the [`COUNT` type in the metric types documentation][2].

{{< tabs >}}
{{% tab "Python" %}}

After having [setup DogStatsD on your host][1] run the following code to submit a DogStatsD `COUNT` metric type,

```python
from datadog import statsd
import time

while(1):
  statsd.increment('example_metric.increment')
  statsd.decrement('example_metric.decrement')
  statsd.count('example_metric.count', 2)
  sleep(10)
```

[1]: /developers/dogstatsd/?tab=python#setup
{{% /tab %}}
{{% tab "Ruby" %}}

```ruby
require 'datadog/statsd'
statsd = Datadog::Statsd.new

while true do
    statsd.increment('example_metric.increment')
    statsd.decrement('example_metric.decrement')
    statsd.count('example_metric.count', 2)
    sleep 10
end
```

{{% /tab %}}
{{% tab "Go" %}}


{{% /tab %}}
{{% tab "Java" %}}

{{% /tab %}}
{{% tab ".NET" %}}

{{% /tab %}}
{{% tab "PHP" %}}

{{% /tab %}}
{{< /tabs >}}

With this code, the data is available to graph in Datadog. Here's an example:

TO DO: Run the script and add a screenshoot

**Notes**:

* StatsD counters are normalized over the flush interval to report per-second units. To increment or measure values over time, see [gauges](#gauges).
* For counters coming from another source that are ever-increasing and never reset (for example, the number of queries from MySQL over time), Datadog tracks the rate between flushed values. To get raw counts within Datadog, apply a function to your series such as _cumulative sum_ or _integral_. [Read more about Datadog functions][3].

## Gauge

| Method | Datadog Storage type |
| :----- | :------- |
|`gauge(MetricName, Value, SampleRate, Tags)`| Stored as a `GAUGE` type in Datadog. Each value in the stored timeseries is the last gauge value submitted for that metric during the StatsD flush period.|

with the following parameter:

| Parameter    | Type            | Required | Description                                                                                                                                                                |
| --------     | -------         | -----    | ----------                                                                                                                                                                 |
| `MetricName` | String          | yes      | Name of the metric to submit.                                                                                                                                              |
| `Value`      | Double          | yes      | Value associated to your metric.                                                                                                                                           |
| `SampleRate` | Double          | no       | Sample rate, between `0` (no sample) and `1` (all datapoints are dropped), to apply to this particular metric. See the [Sample Rate section](#sample-rates) to learn more. |
| `Tags`       | List of Strings | no       | List of Tags to apply to this particular metric. See the [Metrics Tagging](#metrics-tagging) section to learn more.                                                        |

Find below short code snippets depending of your language that you can run to emit a `GAUGE` metric type-stored as `GAUGE` metric type-into Datadog. Learn more about the [`GAUGE` type in the metric types documentation][4].

{{< tabs >}}
{{% tab "Python" %}}

After having [setup DogStatsD on your host][1] run the following code to submit a DogStatsD `GAUGE` metric type into Datadog:

```python
from datadog import statsd
import time

i = 0
while(1):
  i += 1
  statsd.gauge('example_metric.gauge', i )
  sleep(10)
```

[1]: /developers/dogstatsd/?tab=python#setup
{{% /tab %}}
{{% tab "Ruby" %}}

```ruby
# Record the amount of free memory every ten seconds.
while true do
    statsd.gauge('system.mem.free', get_free_memory())
    sleep(10)
end
```

{{% /tab %}}
{{% tab "Go" %}}


{{% /tab %}}
{{% tab "Java" %}}

{{% /tab %}}
{{% tab ".NET" %}}

{{% /tab %}}
{{% tab "PHP" %}}

{{% /tab %}}
{{< /tabs >}}

With this code, the data is available to graph in Datadog. Here's an example:

TO DO: Run the script and add a screenshoot/

## Set

|Method | Datadog Storage type |
|:---|:---|
|`set(MetricName, Value, SampleRate, Tags)`| Stored as `GAUGE` type in Datadog. Each value in the stored timeseries is the count of unique values submitted to StatsD for a metric over that flush period.|

with the following parameter:

| Parameter    | Type            | Required | Description                                                                                                                                                                |
| --------     | -------         | -----    | ----------                                                                                                                                                                 |
| `MetricName` | String          | yes      | Name of the metric to submit.                                                                                                                                              |
| `Value`      | Double          | yes      | Value associated to your metric.                                                                                                                                           |
| `SampleRate` | Double          | no       | Sample rate, between `0` (no sample) and `1` (all datapoints are dropped), to apply to this particular metric. See the [Sample Rate section](#sample-rates) to learn more. |
| `Tags`       | List of Strings | no       | List of Tags to apply to this particular metric. See the [Metrics Tagging](#metrics-tagging) section to learn more.                                                        |

Find below short code snippets depending of your language that you can run to emit a `SET` metric type-stored as `GAUGE` metric type-into Datadog. Learn more about the [`SET` type in the metric types documentation][5].

{{< tabs >}}
{{% tab "Python" %}}

```python
from datadog import statsd
import time
import random

while(1):
  statsd.set('example_metric.set', random.randint(0, 10))
  sleep(10)
```

{{% /tab %}}
{{% tab "Ruby" %}}

```ruby
def login(self, user_id)
    # Log the user in ...
    statsd.set('users.uniques', user_id)
end
```

{{% /tab %}}
{{% tab "Go" %}}


{{% /tab %}}
{{% tab "Java" %}}

{{% /tab %}}
{{% tab ".NET" %}}

{{% /tab %}}
{{% tab "PHP" %}}

{{% /tab %}}
{{< /tabs >}}

With this code, the data is available to graph in Datadog. Here's an example:

TO DO: Run the script and add a screenshoot

## Histogram

| Method             | Datadog Storage type                                                                                  |
| :---               | :---                                                                                      |
| `histogram(MetricName, Value, SampleRate, Tags)` | Since multiple metrics are submitted, metric types stored depend of the metric. The two types stored are `GAUGE`, `Rate` See the [histogram metric type][6] documentation to learn more. |

with the following parameter:

| Parameter    | Type            | Required | Description                                                                                                                                                                |
| --------     | -------         | -----    | ----------                                                                                                                                                                 |
| `MetricName` | String          | yes      | Name of the metric to submit.                                                                                                                                              |
| `Value`      | Double          | yes      | Value associated to your metric.                                                                                                                                           |
| `SampleRate` | Double          | no       | Sample rate, between `0` (no sample) and `1` (all datapoints are dropped), to apply to this particular metric. See the [Sample Rate section](#sample-rates) to learn more. |
| `Tags`       | List of Strings | no       | List of Tags to apply to this particular metric. See the [Metrics Tagging](#metrics-tagging) section to learn more.                                                        |

Histograms are specific to DogStatsD. Find below short code snippets depending of your language that you can run to emit a `HISTOGRAM` metric type-stored as `GAUGE` and `RATE` metric types-into Datadog. Learn more about the [`HISTOGRAM` type in the metric types documentation][6].

{{< tabs >}}
{{% tab "Python" %}}

```python
from datadog import statsd
import time
import random

while(1):
  statsd.set('example_metric.histogram', random.randint(0, 10))
  sleep(10)
```

{{% /tab %}}
{{% tab "Ruby" %}}

```ruby
start_time = Time.now
results = db.query()
duration = Time.now - start_time
statsd.histogram('database.query.time', duration)

# The `time` helper is a short-hand for timing blocks of code.
statsd.time('database.query.time') do
  return db.query()
end
```

{{% /tab %}}
{{% tab "Go" %}}


{{% /tab %}}
{{% tab "Java" %}}

{{% /tab %}}
{{% tab ".NET" %}}

{{% /tab %}}
{{% tab "PHP" %}}

{{% /tab %}}
{{< /tabs >}}

The above instrumentation produces the following metrics:

| Metric                               | Description                               |
| ------------------------------------ | ----------------------------------------- |
| `example_metric.histogram.count`          | Number of times this metric was sampled   |
| `example_metric.histogram.avg`            | Average time of the sampled values        |
| `example_metric.histogram.median`         | Median sampled value                      |
| `example_metric.histogram.max`            | Maximum sampled value                     |
| `example_metric.histogram.95percentile`   | 95th percentile sampled value             |

And the data is available to graph in Datadog. Here's an example:

TO DO: Run the script and add a screenshoot

### Timers

Timers in DogStatsD are an implementation of Histograms (not to be confused with timers in the standard StatsD). They measure timing data only: for example, the amount of time a section of code takes to execute. Choose your language below to see how to implement them according to your needs:

{{< tabs >}}
{{% tab "Python" %}}

In Python, timers are created with a decorator:

```python
from datadog import statsd
import time
import random

@statsd.timed('example_metric.timer')
def my_function():
  sleep(random.randint(0, 10))
```

or with a context manager:

```python
from datadog import statsd
import time
import random

def my_function():

  # First some stuff you don't want to time
  sleep(1)

  # Now start the timer
  with statsd.timed('example_metric.timer'):
    # do something to be measured
    sleep(random.randint(0, 10))
```

{{% /tab %}}
{{% tab "Ruby" %}}


{{% /tab %}}
{{% tab "Go" %}}


{{% /tab %}}
{{% tab "Java" %}}

{{% /tab %}}
{{% tab ".NET" %}}

{{% /tab %}}
{{% tab "PHP" %}}

{{% /tab %}}
{{< /tabs >}}

In either case, as DogStatsD receives the timer data, it calculates the statistical distribution of render times and sends the following metrics to Datadog:

| Metric                               | Description                               |
| ------------------------------------ | ----------------------------------------- |
| `example_metric.histogram.count`          | Number of times this metric was sampled   |
| `example_metric.histogram.avg`            | Average time of the sampled values        |
| `example_metric.histogram.median`         | Median sampled value                      |
| `example_metric.histogram.max`            | Maximum sampled value                     |
| `example_metric.histogram.95percentile`   | 95th percentile sampled value             |

And the data is available to graph in Datadog. Here's an example:

TO DO: Run the script and add a screenshoot

Remember: under the hood, DogStatsD treats timers as histograms. Whether you use timers or histograms, you are sending the same data to Datadog.

## Distribution

**This feature is in BETA. [Contact Datadog support][7] for details on how to have it enabled for your account.**

| Method | Datadog Storage type |
| :----- | :------- |
| `distribution(MetricName, Value, Tags)` | Stored as a `Distribution` type in Datadog. See the dedicated [Distribution documentation][8] to learn more. |

with the following parameter:

| Parameter    | Type            | Required | Description                                                                                                                                                                |
| --------     | -------         | -----    | ----------                                                                                                                                                                 |
| `MetricName` | String          | yes      | Name of the metric to submit.                                                                                                                                              |
| `Value`      | Double          | yes      | Value associated to your metric.                                                                                                                                           |
| `SampleRate` | Double          | no       | Sample rate, between `0` (no sample) and `1` (all datapoints are dropped), to apply to this particular metric. See the [Sample Rate section](#sample-rates) to learn more. |
| `Tags`       | List of Strings | no       | List of Tags to apply to this particular metric. See the [Metrics Tagging](#metrics-tagging) section to learn more.                                                        |

Distributoins are specific to DogStatsD. Find below short code snippets depending of your language that you can run to emit a `DISTRIBUTION` metric type-stored as `DISTRIBUTION` metric type-into Datadog. Learn more about the [`DISTRIBUTION` type in the metric types documentation][9].

{{< tabs >}}
{{% tab "Python" %}}

```python
from datadog import statsd
import time
import random

while(1):
  statsd.set('example_metric.distribution', random.randint(0, 10))
  sleep(10)
```

{{% /tab %}}
{{% tab "Ruby" %}}

```ruby
start_time = Time.now
results = Net::HTTP.get('https://google.com')
duration = Time.now - start_time
statsd.distribution('dist.dd.website.latency', duration)
```

{{% /tab %}}
{{% tab "Go" %}}


{{% /tab %}}
{{% tab "Java" %}}

{{% /tab %}}
{{% tab ".NET" %}}

{{% /tab %}}
{{% tab "PHP" %}}

{{% /tab %}}
{{< /tabs >}}

The data is now available to graph in Datadog:


The above instrumentation calculates the following data: `sum`, `count`, `average`, `minimum`, `maximum`, `50th percentile` (median), `75th percentile`, `90th percentile`, `95th percentile` and `99th percentile`. Distributions are not only for measuring times. They can be used to measure the distribution of *any* type of value, such as the size of uploaded files, or classroom test scores, for example.

## Metric Submission options

### Sample rates

Since the overhead of sending UDP packets can be too great for some performance intensive code paths, DogStatsD clients support sampling, i.e. only sending metrics a percentage of the time. It's not useful in all cases, but can be interesting if you sample many metrics, and your DogStatsD client is not on the same host as the DogStatsD server. This is a trade off: you decrease traffic but slightly lose in precision/granularity.

Before sending the metric to Datadog, DogStatsD uses the `sample_rate` to correct the metric value depending of the metric type, i.e. to estimate what it would have been without sampling:

| Metric Type | Sample rate correction |
| ----------- | ----------------- |
| `Count` | Values received are multiplied by (1/sample_rate), because it's reasonable to suppose in most cases that for 1 datapoint received, `1/sample_rate` were actually sampled with the same value. |
| `Gauge` | No correction. The value received is kept as it is. |
| `Set` | Bo correction. The value received is kept as it is. |
| `Histogram` | The `histogram.count` statistic is a counter metric, and receives the correction outlined above. Other statistics are gauge metrics and aren't "corrected." |

See the [Datadog Agent aggregation code][10] to learn more about this behavior.

The following code only sends points half of the time:

{{< tabs >}}
{{% tab "Python" %}}

```python
while True:
  do_something_intense()
  statsd.increment('loop.count', sample_rate=0.5)
```

{{% /tab %}}
{{% tab "Ruby" %}}

```ruby
while true do
  do_something_intense()
  statsd.increment('loop.count', :sample_rate => 0.5)
end
```

{{% /tab %}}
{{% tab "Go" %}}


{{% /tab %}}
{{% tab "Java" %}}

{{% /tab %}}
{{% tab ".NET" %}}

{{% /tab %}}
{{% tab "PHP" %}}

{{% /tab %}}
{{% tab "C++" %}}

{{% /tab %}}
{{< /tabs >}}

### Metrics Tagging

Add tags to any metric you send to DogStatsD. For example, compare the performance of two algorithms by tagging a timer metric with the algorithm version:


{{< tabs >}}
{{% tab "Python" %}}

```python

@statsd.timed('algorithm.run_time'x)
def algorithm_one():
    # Do fancy things here ...

@statsd.timed('algorithm.run_time', tags=['algorithm:two'])
def algorithm_two():
    # Do fancy things (maybe faster?) here ...
```

{{% /tab %}}
{{% tab "Ruby" %}}

{{% /tab %}}
{{% tab "Go" %}}


{{% /tab %}}
{{% tab "Java" %}}

{{% /tab %}}
{{% tab ".NET" %}}

{{% /tab %}}
{{% tab "PHP" %}}

{{% /tab %}}
{{% tab "C++" %}}

{{% /tab %}}
{{< /tabs >}}

#### Host tag key

The host tag is assigned automatically by the Datadog Agent aggregating the metrics. Metrics submitted with a host tag not matching the Agent hostname lose reference to the original host. The host tag submitted overrides any hostname collected by or configured in the Agent.

**Note**: Because of the global nature of Distributions, extra tools for tagging are provided. See the [Distribution Metrics][11] page for more details.

## Further reading

{{< partial name="whats-next/whats-next.html" >}}
[1]: /developers/dogstatsd
[2]: /developers/metrics/metrics_type/?tab=count#metric-type-definition
[3]: /graphing/functions/#apply-functions-optional
[4]: /developers/metrics/metrics_type/?tab=gauge#metric-type-definition
[5]: /developers/metrics/metrics_type/?tab=set#metric-type-definition
[6]: /developers/metrics/metrics_type/?tab=histogram#metric-type-definition
[7]: /help
[8]: /graphing/metrics/distributions
[9]: /developers/metrics/metrics_type/?tab=distribution#metric-type-definition
[10]: https://github.com/DataDog/dd-agent/blob/master/aggregator.py
[11]: /graphing/metrics/distributions