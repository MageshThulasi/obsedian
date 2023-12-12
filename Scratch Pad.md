# 11/27/23
- Two identities
	- Tier 0 -  first.last@gcp.com is one id - Prod LDAP
	- Tier 1,2 and 3 - first.last@wd.com is another id (first.last@wdgcpdev.com) - Via okta
- GCP is tree structure
	- one super admin login
	- Projects is equal to one account in AWS

https://confluence.workday.com/display/InfoSec/GCP+IAM+-+Full+story
https://confluence.workday.com/display/INFRA/*+Directory+Services+at+Workday

---
# HAProxy
- [Exported Metrics](https://www.haproxy.com/documentation/hapee/latest/observability/metrics/prometheus/#exported-metrics)
- [Link](https://grafana.produs.us.wdpharos.io/connections/datasources) > Navigate to Home | Connections | Data Sources
	- Cortex > Explore
	- `haproxy_*_requests`
	- Use Haproxy-Ingress-Test dashboard
	- [SLO sample dashboard](https://grafana.produs.us.wdpharos.io/d/9e3a243814377e06d5695e1162aa7428/pharos-metrics-slis-slos?orgId=1&refresh=30s)
- [HAProxy to Monitor Application Health](https://www.haproxy.com/blog/use-your-load-balancer-to-monitor-application-health)
	- Success vs Failed HTTP Responses: `rate(haproxy_server_http_responses_total{stack="usw2-ipc-1", code=~"2xx|4xx|5xx", proxy="vault"}[$__rate_interval])`
	- Server Response Times: `haproxy_backend_response_time_average_seconds{stack="euc1-ipc-1", proxy="vault"}`
		- Servers UP/Down: `haproxy_backend_active_servers{stack="euc1-ipc-1", proxy="vault"}`
		- `haproxy_backend_status{stack="euc1-ipc-1", proxy="vault"} == 1`
		- Show only these two for the UP/DOWN status: az="ap-southeast-2a", instance="i-0312d930ad5fa2ddc"
	- Latency
		- haproxy_server_response_time_average_seconds
		- avg(histogram_quantile(0.99,rate(haproxy_server_response_time_average_seconds{stack="euc1-ipc-1", proxy="vault"}[$__rate_interval])))
- [Terminology](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring)
	- https://developer.harness.io/docs/service-reliability-management/get-started/slo-dashboard/#:~:text=The%20SLO%20dashboard%20provides%20a,service's%20overall%20health%20and%20performance.
	- SLO - a statement of desired performance
	- SLI - a measurement of performance
		- Request count - the number of requests per minute that result in 2xx or 5xx responses
			- Availability SLI - ratio of the number of successful responses vs the number of all responses
		- Response latencies - the latency for HTTP 2xx responses
			- Latency SLI - ratio of the number of calls below a latency threshold to the number of all calls
	- Error budget - starts at 1 - SLO and declines as the actual performance misses the SLO
```shell
curl http://localhost:9100/metrics > metrics.log

sudo netstat -tulpn | grep haproxy
curl http://localhost:8443/stats > stats.log

# To view the haproxy stats socket
sudo ls -la /var/lib/haproxy/stats
sudo file /var/lib/haproxy/stats
sudo echo "show stat" | sudo nc -U /var/lib/haproxy/stats
sudo watch -n 1 'echo "show stat" | sudo nc -U /var/lib/haproxy/stats | cut -d "," -f 1,2,5-11,18,24,36,50,62 | column -s , -t'

sudo watch -n 1 'echo "show stat" | sudo nc -U /var/lib/haproxy/stats | grep "vault" | cut -d "," -f 1,2,5-11,18,24,36,50,62 | column -s , -t'
```
```json
rate(haproxy_backend_requests_denied_total{job="haproxy_exporter",stack="$stack",role=~".*$role",instance=~"$instance_id"}[$__rate_interval])
```

## Types of metrics
### Counters
Always increases. Ex: Request count, Error count
#### Query
Per second rate of request averaged over the last 5 minutes
rate(request_count[5m])

Note that the rate function only works for counters as it relies on the metric value always going up and never down 

### Gauges
Can we used for metrics that goes up or down. Ex: Temperature, CPU or Memory usage, Number of requests in progress, queue size
#### Query
Average queue size over the last 5 minutes
avg_over_time(queue_size[5x])
### Histograms
Measures the frequency of value observations that fall into specific predefined buckets. Ex: Measure request duration for a specific HTTP request school using Histograms. Rather than storing every duration for every request, Prometheus will make an approximation by storing the frequency of requests that fall into particular buckets. The default bucket values are: .005, .01, .025, .05, .075, .1, .25, .5, .75, 1, 2.5, 5, 7.5 and 10. This is very much tuned to measuring request durations below 10 seconds. 

When should we use Histograms? When you want to take many measurements of a vault to later calculate averages or percentiles. When you are not bothered about the exact values and are happy with an approximation and when you know what the range of values will be upfront, so you can use default bucket definitions or create custom buckets.

Use cases: Request duration, response size
#### Query
Use the following to calculate the average request duration within the last five minutes.
rate(request_duration_sum[5m]) / rate(request_duration_count[5m])

The histogram metric also allows us to calculate the Centaurs. You can use the built-in histogram quantile function. We can calculate the 95th percentile or 0.95 quantile in the last 5 minutes.
histogram_quantile(.95, sum(rate(request_duration_bucket[5m])) by (le))
### Summaries
Summaries and histograms share a lot of similarities. Histograms superseded summaries in Prometheus and the recommendation is to use histograms where possible.

Differences: With histograms, quantiles are calculated on the Prometheus server. With summaries, the quantiles are calculated on the application server. Therefore, summary data cannot be aggregated from a number of application instances. 

Histograms require upfront bucket definition so suit the use case of where you have a good idea about the spread of your values. Summaries are a good option if you need to calculate accurate quantiles that can't be sure of what the range of values will be up front.

When should we use summaries?  When you want to take many measurements of a value, to later calculate averages or percentiles. When you don't know what the range of values will be up front.

Use cases: Request duration, response size
