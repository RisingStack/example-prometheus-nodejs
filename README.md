# Example Prometheus Monitoring

## Goal

Setup monitoring with [Prometheus](https://prometheus.io) and [Grafana](https://grafana.com/).

## Steps

1. Run sample server: `npm install` and `node server`
2. Run Prometheus: see below
3. Visit your running Prometheus and run queries
4. Run Grafana: see below
5. Add Prometheus data source (Url: `http://localhost:9090`, Access: `direct`)
6. Import `grafana-dashboard.json` dashboard
7. Create your own dashboard from the Prometheus queries

## Requirements

- Docker

### Run

Modify: `/prometheus-data/prometheus.yml`, replace `192.168.0.10` with your own host machine's IP.
Host machine IP address: `ifconfig | grep 'inet 192'| awk '{ print $2}'`

```sh
docker run -p 9090:9090 -v "$(pwd)/prometheus-data":/prometheus-data prom/prometheus -config.file=/prometheus-data/prometheus.yml
```

[Open Prometheus: http://http://localhost:9090](http://http://localhost:9090/graph)

### Example Queries

#### Throughput

##### Request Per Minute

```
sum(rate(http_request_duration_ms_count[1m])) by (service, route, method, code)  * 60
```

#### Response Time

In milliseconds.

##### 95th Response Time

```
histogram_quantile(0.95, sum(rate(http_request_duration_ms_bucket[1m])) by (le, service, route, method))
```

##### Median Response Time:

```
histogram_quantile(0.5, sum(rate(http_request_duration_ms_bucket[1m])) by (le, service, route, method))
```

##### Average Response Time

```
avg(rate(http_request_duration_ms_sum[1m]) / rate(http_request_duration_ms_count[1m])) by (service, route, method, code)
```

#### Memory Usage

##### Average Memory Usage

In Megabyte.

```
avg(nodejs_external_memory_bytes / 1024) by (service)
```

### Reload config

Necessary when you modified prometheus-data.

```sh
curl -X POST http://localhost:9090/-/reload
```

### Prometheus Data

```
avg(rate(http_request_duration_ms_sum[1m]) / rate(http_request_duration_ms_count[1m])) by (service, route, method, code)
```

![Prometheus - Data](/images/prometheus-data.png)

### Prometheus Alerts

States of active alerts: `pending`, `firing`

![Prometheus - Alert Pending](/images/prometheus-alert-pending.png)
![Prometheus - Alert Firing](/images/prometheus-alert-firing.png)

## Grafana

### Run

```sh
docker run -i -p 3000:3000 grafana/grafana
```

[Open Grafana: http://http://localhost:3000](http://http://localhost:3000)

```
Username: admin
Password: admin
```

Grafana Dashboard to import: [/grafana-dashboard.json](/grafana-dashboard.json)

### Grafana Dashboard

![Grafana - Response Time](/images/grafana-response-time.png)
![Grafana - Throughput](/images/grafana-throughput.png)

## Acknowledgements

This example is sponsored by [Trace by RisingStack](https://trace.risingstack.com).
