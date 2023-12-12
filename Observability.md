# Grafana

```sh
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

## Graphite & StatsD

```sh
docker run -d --name graphite --restart=always -p 81:81 -p 8125:8125/udp hopsoft/graphite-statsd

docker container ls
docker exec -it graphite bash
apt-get update
apt-get install nano

cd /opt/graphite/conf
nano storage-schemas.conf

[shoehub]
pattern = ^shoehub\.
retentions = 20s:5h

docker stop graphite
docker start graphite
```

### Sending metrics to StatsD & Graphite
[GitHub](https://github.com/aussiearef/ShoeHub/tree/master)

**statsd.sh**
```sh
#!/bin/bash

echo "$2:1|c" | nc -w 1 -u $1 8125
```

```sh
chmod +x statsd.sh
~/statsd.sh 10.78.144.85 shoehub.test
watch -n 1 ~/statsd.sh 10.78.144.85 shoehub.test
```
# Prometheus
Run prometheus in a docker container and mount the prometheus config file from the persistent storage

```sh
docker run -p 9090:9090 --name prometheus -v /Users/magesh.thulasi/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

Contents of */Users/magesh.thulasi/prometheus/prometheus.yml*:

```sh
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

Once the docker container is up, you can navigate to prometheus at http://10.78.144.85:9090/
## Configure Blackbox

## Update Prometheus

## Create Grafana Dashboard

# Monitoring & Observability

- [Helm - Cheat sheet](https://helm.sh/docs/intro/cheatsheet/)
- [GitHub - Helm chart - Prometheus Community K8s Helm Charts](https://github.com/prometheus-community/helm-charts)
- [GitHub - Helm chart - kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [K8s observability - Grafana, Prometheus, Fluentd, Tempo, Loki](https://dev.to/thenjdevopsguy/implementing-open-source-monitoring-and-observability-in-kubernetes-1bgn)
- [YouTube - Setup Prometheus & Grafana](https://www.youtube.com/watch?v=QoDqxm7ybLc)
- [YouTube - Setup MongoDB Exporter](https://www.youtube.com/watch?v=mLPg49b33sA&t=53s)

- [Prometheus Operator - Definitive Guide](https://www.infracloud.io/blogs/prometheus-operator-helm-guide/)
- [YouTube - Jim Sheldon - Monitoring with Blackbox Exporter](https://www.youtube.com/watch?v=2TpQ3ETmhsw&t=484s)
- [Check out Synthetic Monitoring section](https://www.hashicorp.com/blog/hashicorp-vault-observability-monitoring-vault-at-scale)

## Install Prometheus & Grafana using Helm in a Kubernetes cluster
```sh
brew install helm

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm repo list

k create namespace monitoring
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -f prometheus.yaml --namespace monitoring
helm list --namespace monitoring
```

## Access Grafana and Prometheus from a browser

```sh
k get all -n monitoring
k --namespace monitoring get pods -l "release=kube-prometheus-stack"
k port-forward svc/kube-prometheus-stack-grafana 8080:80 -n monitoring

k port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring
```
- Access grafana at `http://localhost:8080` with credentials `admin/prom-operator`
- Access prometheus at `http://localhost:9090`

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.

#### Uninstallation and miscellaneous
```sh
helm uninstall kube-prometheus-stack --namespace monitoring
```

## Install Backbox Exporter

You can add the modules for blackbox.yml in the config section:
```json
config:
  modules:
    http_2xx:
      prober: http
      timeout: 5s
      http:
        valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
        follow_redirects: true
        preferred_ip_protocol: "ip4"
```

```shell
helm install prometheus-blackbox prometheus-community/prometheus-blackbox-exporter -f blackbox.yaml --namespace monitoring
```

# SLO dashboard
- [SLO implementation](https://mattjmcnaughton.com/post/slo-implementation-part-1/)
- [SLO dashboard](https://grafana.com/grafana/dashboards/11222-slo-dashboard/)

Prometheus supports 4 main [metric types](https://prometheus.io/docs/concepts/metric_types/): counter, gauge, histogram and summary. We will use the counter and the histogram for our SLO dashboard
## Availability - using counter metric
For the availability component, we need to track:
- Number of failed requests
- Total number of requests

```sql

sum(rate(caddy_http_response_status_count_total{service="blog",status!~"5.."}[1h])) /
sum(rate(caddy_http_response_status_count_total{service="blog"}[1h]))
```

## Latency - using histogram metric
For the latency SLO, we need to track `the ever changing response time of server requests`. 

We could use a gauge, which provides a snapshot of current state (i.e. the response time of the last request), but histogram is better suited since it captures a snapshot of the current state, but also supports percentile calculations. For example, what is the 99% percentile response time for the requests.

```sql
histogram_quantile(0.99, sum(rate(caddy_http_request_duration_seconds_bucket[1h])) by (le))
```

### Scratch Pad
probe_success{instance=~"$target"}
sum(rate(probe_success{instance=~"$target",job="blackbox-external-targets"}[1h]))

probe_http_duration_seconds{instance=~"$target"}
probe_duration_seconds

probe_http_status_code
probe_ssl_earliest_cert_expiry

avg_over_time(probe_success{instance=~"$target"}[1d])
quantile_over_time(0.9, probe_success[1d])
node_memory_MemAvailable_bytes

---

Metrics have labels to provide dimensional data model.
Metric types refer to different ways in which exporters represent the metric data they provide. These are counter

Merge the following for AWS, GCP and FedRamp
- Slack channels
- Code repositories
- Work with the following individuals to solve the access issue between AWS, GCP and FedRamp
	- Rory, Joe and Keith (and Henry)
	- Short term - May be added them to access groups
	- Long term - Have one profile for all 3 groups - like INF-IPC profile

- General awareness on resiliency
	- What is resiliency?
		- The ability to recover quickly from service disruption. It is one of the 6 pillars of AWS Well-Architected Framework
		- https://aws.amazon.com/resilience/
	- Steps to consider for reducing failure
		- Backoff and retry
		- Circuit breaker
		- Graceful degradation
		- Throttling
		- Load shedding
	- What is HA, single point of failure, DR?
	- Make it a standard instead of individual way
	- Come up with a presentation with Rory
	- Ramki will share the spreadsheet with list of services - for each one, is it resilient?

---

Slide deck links:
- [Icons](https://logosear.ch/search.html?q=email)
- https://medium.com/@akashjoffical08/monitor-uptime-of-endpoints-in-k8s-using-blackbox-exporter-f80166a328e9
- https://www.squadcast.com/blog/infrastructure-monitoring-using-kube-prometheus-operator
- https://picluster.ricsanfre.com/docs/prometheus/

PDX outage:
- Reputation
- #1: *WD system MUST be Resilient, Secure and Cost-efficient*
- Move to public cloud
- Practice DR regularly
- Look at each service and validate #1
	- Define RPO and RTO for each and make sure they are resilient
- Get the document from Victor Zakharyev

Sean Dugan, Rory Mitchell
INFRA-SYSTEMS is the group for AWS

Access options for both AWS and GCP
- Review the on-boarding document: [https://docs.google.com/document/d/1IZoATGT_yJwRz2ABoKOUleCs4BhCSemtE7WNUOCOoE0/edit](https://docs.google.com/document/d/1IZoATGT_yJwRz2ABoKOUleCs4BhCSemtE7WNUOCOoE0/edit "https://docs.google.com/document/d/1IZoATGT_yJwRz2ABoKOUleCs4BhCSemtE7WNUOCOoE0/edit")
- Discuss options for developers to access both AWS and GCP
	- Rory proposed an option to use INFRA-SYSTEMS profile to have all the GCP groups
	- Any other options to consider?
