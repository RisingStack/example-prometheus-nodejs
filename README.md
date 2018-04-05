# Example Prometheus Monitoring

## Goal

Setup monitoring with [Prometheus](https://prometheus.io) and [Grafana](https://grafana.com).

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

Open Prometheus: [http://localhost:9090](http://localhost:9090/graph)

### Example Queries

#### Throughput

#### Error rate

Range[0,1]: number of 5xx requests / total number of requests

```
sum(increase(http_request_duration_ms_count{code=~"^5..$"}[1m])) /  sum(increase(http_request_duration_ms_count[1m]))
```

##### Request Per Minute

```
sum(rate(http_request_duration_ms_count[1m])) by (service, route, method, code)  * 60
```

#### Response Time

#### Apdex

[Apdex](https://en.wikipedia.org/wiki/Apdex) score approximation:  
`100ms` target and `300ms` tolerated response time

```
(
  sum(rate(http_request_duration_ms_bucket{le="100"}[1m])) by (service)
+
  sum(rate(http_request_duration_ms_bucket{le="300"}[1m])) by (service)
) / 2 / sum(rate(http_request_duration_ms_count[1m])) by (service)
```

> Note that we divide the sum of both buckets. The reason is that the histogram buckets are cumulative. The le="100" bucket is also contained in the le="300" bucket; dividing it by 2 corrects for that. - [Prometheus docs](https://prometheus.io/docs/practices/histograms/#apdex-score)

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
avg(nodejs_external_memory_bytes / 1024 / 1024) by (service)
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

[Open Grafana: http://localhost:3000](http://localhost:3000)

```
Username: admin
Password: admin
```

### Setting datasource

Create a Grafana datasource with this settings:
+ name: DS_PROMETHEUS
+ type: prometheus
+ url: http://localhost:9090
+ access: direct

Or use this curl request:
```sh
curl 'http://admin:admin@localhost:3000/api/datasources' -H 'Content-Type: application/json;charset=UTF-8' -H 'Accept: application/json, text/plain, */*' --data-binary '{"name":"DS_PROMETHEUS","type":"prometheus","url":"http://localhost:9090","access":"direct","jsonData":{"keepCookies":[]},"secureJsonFields":{}}' --compressed
```

### Setting dashboard

Grafana Dashboard to import: [/grafana-dashboard.json](/grafana-dashboard.json)

Or use this curl request:
```sh
curl 'http://admin:admin@localhost:3000/api/dashboards/import' -H 'Accept-Encoding: gzip, deflate' -H 'Content-Type: application/json;charset=UTF-8' -H 'Accept: application/json, text/plain, */*' --data-binary '%{copy and paste grafana-dashboard.json}' --compressed
```

### Grafana Dashboard

![Grafana - Response Time](/images/grafana-response-time.png)
![Grafana - Throughput](/images/grafana-throughput.png)

## Acknowledgements

This example is sponsored by [Trace by RisingStack](https://trace.risingstack.com).
