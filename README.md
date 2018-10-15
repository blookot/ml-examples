# Elastic Machine Learning examples
Helping users define jobs of Machine Learning (ML) using the Elastic stack.

## Introduction
This document describes (at a macroscopic level) how to create [Machine Learning jobs](https://www.elastic.co/products/stack/machine-learning) with the Elastic stack.

In the following document, each job is described by:
* a short title, 
* the "detector" part of the job, with `partition_field_name` that is the keyword Elastic uses to "split" the data (as you can find in multi-metric jobs), `over` for population jobs or `by` that you see in the advanced jobs,
* the type of job (among multi-metric, population and advanced, as they appear in the UI).

In order to focus the ML job on a subset of data (for example auth failures or one network protocol in particular), 2 features can be used: saved searches in Discover (the easy one) or the Datafeed query field of the advanced job where the following syntax can be used:
* `{ "match": { "@to.keyword": "xxx" }}`
* `{ "term": { "network.protocol": { "value": "dns", "boost": 1 }}}`
* `{ "term": { "event.action": { "value": "reject", "boost": 1 }}}`

The fields are named according to the [Elastic Common Schema](https://github.com/elastic/ecs).

You can get further details on the [Elastic ML recipe page](https://www.elastic.co/products/stack/machine-learning/recipes).

## Security
Identify security attacks.

### Data exfiltration
Description: stealing data out of an organization network

**Source**: Proxy logs, DNS requests logs, packet capture (via packetbeat)

**Enrichment**: Malicious domains, top visited URLs (Alexa for instance), Reverse DNS... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Outgoing volume by source: `sum(network.outbound.bytes) over source.ip` (population) and `sum(network.outbound.bytes) partition_field_name=source.ip` (multi-metric)
* Unusual domains: `rare() by destination.domain` (advanced)
* Volume (nb of requests or outgoing bytes) by domain: `sum(network.outbound.bytes) over destination.domain` (population) and `count() over destination.domain` (population)
* Number of subdomains by domain: `distinct_count(destination.subdomain) over destination.domain` (population)
* Entropy of subdomains by domain: `high_info_content(destination.subdomain) over destination.domain` (advanced), see the [DNS exfiltration recipe](https://www.elastic.co/products/stack/machine-learning/recipes/dns-data-exfiltration-tunneling) for more info

### Web Scraping
Description: data scraping used for extracting data from websites (Wikipedia)

**Source**: Web accces logs (Reverse proxy / LB) or Web server access logs

**Enrichment**: Bots list, User Agents parsing, Reverse DNS, GeoIP... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Nb of hits (RPS): `count() over user.id` (population) or `count() over user.id partition_field_name=http.response.status_code` (advanced)
* Downloaded volume (comparing per session/user id): `sum(network.outbound.bytes) over user.id` (population)

### Unusual website visitor
Split visitors between users and non human (robots, etc)

**Source**: Web accces logs (Reverse proxy / LB) or Web server access logs

**Enrichment**: Bots list, User Agents parsing, Reverse DNS, GeoIP... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Unusual UA: `rare() by user_agent.original` (advanced job)
* Unusual source IP: `rare() by source.ip` (advanced job)
* Comparing hits over users: `high_count() partition_field_name=http.response.status_code over user.id` (advanced job)

### Port scanning
Detect a port scan activity on the network

**Source**: packets (packetbeat) or netflow (using the [Logstash netflow module](https://www.elastic.co/guide/en/logstash/current/netflow-module.html))

**Enrichment**: CMDB mostly. See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Nb destinations by source: `high_distinct_count(destination.ip) partition_field_name=source.ip` (advanced job)
* Nb destination ports by source: `high_distinct_count(destination.port) partition_field_name=source.ip` (advanced job)

### Authentication brute force
Brute force attacks on authentication

**Source**: authentication logs (from application, SSH, VPN, etc)

**Enrichment**: AD, CMDB, GeoIP... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Failed logins by source: `count() over source.ip partition_field_name=event.action` (population job), see the [ML recipe](https://www.elastic.co/products/stack/machine-learning/recipes/detect-suspicious-login-activity-volume) for more info.

### Unusual login
Unusual login time or location

**Source**: authentication logs (from application, SSH, VPN, etc)

**Enrichment**: AD, CMDB, GeoIP (adding the `user.location` below). See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Login time per user and device: `time_of_day() by user.id partition_field_name=host.id` (advanced job)
* Login location per user and device: `rare() by user.location partition_field_name=host.id` (advanced job)

### Unusual process
Unusual process started on a device

**Source**: processes started (from Windows events using winlogbeat and auditbeat) and process runing (via metricbeat)

**Enrichment**: AD, CMDB... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Unusual processes: `rare() by process.pid partition_field_name=host.id` (advanced job), see the [ML recipe](https://www.elastic.co/products/stack/machine-learning/recipes/detect-suspicious-process-activity-host) for more information.
* rare arguments: `high_info_content(process.args) partition_field_name=process.name` (advanced job)

## Application monitoring (perf, errors...)
Track issues of performance or errors in applications.

### Performance
Track performance issue (response time). Response time decrease can also be abnormal (to be correlated with errors and response codes).

**Source**: application logs (from the app itself), Web accces logs (Reverse proxy / LB) or Web server access logs

**Enrichment**: CMDB, GeoIP, reverse DNS... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Average response time: `avg(http.response.time) partition_field_name=url.host.name,http.response.status_code` (multi-metric job)
* Maximum response time: `max(http.response.time) partition_field_name=url.host.name,http.response.status_code` (multi-metric job)

### Visits
Track visitors, nb of hits, session time, etc

**Source**: application logs (from the app itself), Web accces logs (Reverse proxy / LB) or Web server access logs

**Enrichment**: CMDB, GeoIP, reverse DNS... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML jobs**:
* Session duration: `avg(session.duration)` (simple job)
* Hits over users: `count() over user.id` (population job)
* Hits by response code: `count() partition_field_name=url.host.name,http.response.status_code` (multi-metric job)


## Infra monitoring

in/out bound traffic : high mean bytes in/out
CPU by process
memory by process
