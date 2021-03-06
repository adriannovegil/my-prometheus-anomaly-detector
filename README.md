# Anomaly Detection in Prometheus Metrics

This repository contains the prototype for a Prometheus Anomaly Detector (PAD). The PAD is a framework to deploy a metric prediction model to detect anomalies in prometheus metrics.

Prometheus is the chosen application to do monitoring across multiple products and platforms. Prometheus metrics are time series data identified by metric name and key/value pairs.

This application leverages machine learning algorithms such as __Fourier__ and __Prophet__ models to perform time series forecasting and predict anomalous behavior in the metrics. The predicted values are compared with the actual values and if they differ from the default threshold values, it is flagged as an anomaly.

## Use Case

The use case for this framework is to assist teams in real-time alerting of their system/application metrics.

The time series forecasting performed by the models can be used by developers to update/enhance their systems to tackle the anomalies in the future.   

## Configurations

 * `FLT_PROM_URL` - URL for the prometheus host, from where the metric data will be collected
 * `FLT_PROM_ACCESS_TOKEN` - OAuth token to be passed as a header, to connect to the prometheus host (Optional)
 * `FLT_METRICS_LIST` - List of metrics that are to be collected from prometheus and train the prophet model.
<br> Example: `"up{app='openshift-web-console', instance='172.44.0.18:8443'}; up{app='openshift-web-console', instance='172.44.4.18:8443'}; es_process_cpu_percent{instance='172.44.17.134:30290'}"`, multiple metrics can be separated using a semi-colon `;`.
<br>If one metric and label configuration matches more than one timeseries, all the timeseries matching the configuration will be collected.
 * `FLT_RETRAINING_INTERVAL_MINUTES` - This specifies the frequency of the model training, or how often the model is retrained. (Default: `15`)
<br> Example: If this parameter is set to `15`, it will collect the past 15 minutes of metric data every 15 minutes and append it to the training dataframe.
 * `FLT_ROLLING_TRAINING_WINDOW_SIZE` - This parameter limits the size of the training dataframe to prevent Out of Memory errors. It can be set to the duration of data that should be stored in memory as dataframes. (Default `15d`)
<br> Example: If set to `1d`, every time before training the model using the training dataframe, the metric data that is older than 1 day will be deleted.

## Run detection

If you want to execute the application, you can do the following:

```
 $ docker-compose up
```

## Implementation

The current setup is as follows:

 * __Data__: Prometheus metrics scraped from specified hosts/targets
 * __Models being trained__:
  * __Fourier__: It is used to map signals from the time domain to the frequency domain. It represents periodic time series data as a sum of sinusoidal components (sine and cosine)
  * __Prophet__[https://facebook.github.io/prophet/]: Procedure developed by Facebook for forecasting time series data based on an additive model where non-linear trends are fit with yearly, weekly, and daily seasonality, plus holiday effects. It works best with time series that have strong seasonal effects and several seasons of historical data. The following are the forecasted values:
    * `yhat`: Predicted time series value
    * `yhat_lower`: Lower bound of uncertainity interval
    * `yhat_upper`: Upper bound of uncertainity interval
 * __Visualization__ - Grafana dashboards are created to visualize the predicted  metrics
 * __Alerts__ - Prometheus alerts are configured based on predicted metric values

![Thoth Dgraph anomaly detection - blog post (2)](https://user-images.githubusercontent.com/7343099/64876403-081c9f80-d61d-11e9-84df-266c91a75dde.jpg)
