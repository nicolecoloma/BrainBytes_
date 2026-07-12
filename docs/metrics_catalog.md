BrainBytes Metrics Catalog

The BrainBytes backend exposes Prometheus metrics through the /metrics endpoint. These metrics provide visibility into application performance, system resource usage, API traffic, and AI tutoring response times. They are intended for monitoring with Prometheus and visualization in Grafana.

Available Metrics
Metric Name	Type	Description	Labels	Example Query
brainbytes_http_requests_total	Counter	Total number of HTTP requests received by the backend.	method, route, status	sum(rate(brainbytes_http_requests_total[5m]))
brainbytes_active_sessions	Gauge	Number of chat requests currently being processed.	None	brainbytes_active_sessions
brainbytes_ai_response_duration_seconds	Histogram	AI response latency for tutoring requests. Supports percentile and average latency calculations.	subject	histogram_quantile(0.95, rate(brainbytes_ai_response_duration_seconds_bucket[5m]))
process_cpu_seconds_total	Counter	Total CPU time consumed by the backend process (default Node.js metric).	None	rate(process_cpu_seconds_total[5m])
process_resident_memory_bytes	Gauge	Current memory usage of the backend process (default Node.js metric).	None	process_resident_memory_bytes
Metrics by Category
Category	Metrics	Purpose
System	process_cpu_seconds_total, process_resident_memory_bytes	Monitor backend resource utilization and runtime health.
Application	brainbytes_http_requests_total, brainbytes_active_sessions	Track API traffic, request volume, and active workloads.
Business	brainbytes_ai_response_duration_seconds	Measure the performance of the AI tutoring service.
PromQL Query Reference

The following queries can be used in Prometheus or Grafana dashboards.

Monitoring Goal	PromQL	Description
Request rate	sum(rate(brainbytes_http_requests_total[5m]))	Requests processed per second over the last five minutes.
Error rate	sum(rate(brainbytes_http_requests_total{status=~"5.."}[5m])) / sum(rate(brainbytes_http_requests_total[5m]))	Percentage of requests returning 5xx errors.
Requests by route	sum by (route) (rate(brainbytes_http_requests_total[5m]))	Traffic distribution across API endpoints.
Active sessions	brainbytes_active_sessions	Current number of active tutoring requests.
AI p95 latency	histogram_quantile(0.95, rate(brainbytes_ai_response_duration_seconds_bucket[5m]))	Response time experienced by 95% of users.
Average AI latency	rate(brainbytes_ai_response_duration_seconds_sum[5m]) / rate(brainbytes_ai_response_duration_seconds_count[5m])	Average AI response time.
AI latency by subject	histogram_quantile(0.95, sum by (le, subject) (rate(brainbytes_ai_response_duration_seconds_bucket[5m])))	Compare response latency across subjects.
Backend availability	up{job="backend"}	Backend health (1 = up, 0 = down).
Memory usage	process_resident_memory_bytes	Current backend memory consumption.
CPU usage	rate(process_cpu_seconds_total[5m])	Average CPU utilization over five minutes.
Alert Rules

Alert rules notify operators when key performance thresholds are exceeded.

Alert	Trigger	Severity	Response
HighAIResponseLatency	p95 AI response time > 5 seconds for 2 minutes	Warning	Check Groq API availability, backend performance, and network latency.
BackendDown	up{job="backend"} == 0 for 1 minute	Critical	Restart the backend service and verify successful recovery.
HighErrorRate	5xx error rate > 5% for 5 minutes	Warning	Review backend logs and investigate the source of server errors.
Traffic Simulation

Generate sample traffic using:

node traffic-simulator.js [normal|peak|error|quiet]
Scenario	Configuration	Expected Result
Normal	3 requests/batch, 2-second delay, 5% errors	Stable traffic with low latency and minimal errors.
Peak	15 requests/batch, 0.5-second delay, 10% errors	Increased request rate, higher active sessions, and possible latency spikes.
Error	5 requests/batch, 1-second delay, 60% errors	Triggers the HighErrorRate alert.
Quiet	1 request/batch, 5-second delay, 2% errors	Simulates low-traffic periods with minimal resource usage.
