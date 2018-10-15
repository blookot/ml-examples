# ml-examples
High level description of Elastic ML jobs

## Introduction
This document describes (at a macroscopic level) how to create Machine Learning jobs with the Elastic stack.

In the following document, each job is described by a short title, the "detector" part of the job as well as the type of job (among multi-metric, population and advanced, as they appear in the UI).

The fields are named according to the [Elastic Common Schema](https://github.com/elastic/ecs).

You can get further details on the [Elastic ML recipe page](https://www.elastic.co/products/stack/machine-learning/recipes).

## Security

### Data exfiltration
Description: stealing data out of an organization network

**Source**: Proxy logs, DNS requests logs, packet capture (via packetbeat)

**Enrichment**: Malicious domains, top visited URLs (Alexa for instance), Reverse DNS... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML job**
* Outgoing volume by source: `sum(network.outbound.bytes) over source.ip` (population) and `sum(network.outbound.bytes) partition_field_name=source.ip` (multi-metric)
* Unusual domains: `rare by destination.domain` (advanced)
* Volume (nb of requests or outgoing bytes) by domain: `sum(network.outbound.bytes) over destination.domain` (population) and `count() over destination.domain` (population)
* Number of subdomains by domain: `distinct_count(destination.subdomain) over destination.domain` (population)
* Entropy of subdomains by domain: `info_content(destination.subdomain) over destination.domain` (advanced), see the [DNS exfiltration recipe](https://www.elastic.co/products/stack/machine-learning/recipes/dns-data-exfiltration-tunneling) for more info

### Web Scraping
Description: data scraping used for extracting data from websites (Wikipedia)

**Source**: Web accces logs (Reverse proxy / LB) or Web server access logs

**Enrichment**: Bots list, User Agents parsing, Reverse DNS, GeoIP... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML job**
* Nb of hits (RPS): `count() over user.id` (population) or `count() over user.id partition_field_name=http.response.status_code` (advanced)
* Downloaded volume (comparing per session/user id): `sum(network.outbound.bytes) over user.id` (population)

### Unusual website visitor
Split visitors between users and non human (robots, etc)

**Source**: Web accces logs (Reverse proxy / LB) or Web server access logs

**Enrichment**: Bots list, User Agents parsing, Reverse DNS, GeoIP... See the [Logstash transformation capabilities](https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html) for more info.

**ML job**
* Unusual UA: `rare by user_agent.original` (advanced job)
* Unusual source IP: `rare by source.ip` (advanced job)
* Unusual source IP: `high_count partition_field_name=http.response.status_code over user|client_ip` (advanced job)

### Port scanning

### Priviledge escalation

### Authentication brute force

### Unusual login
Unusual login time or location

### Unusual process


## Application monitoring (perf, errors...)

## Infra monitoring

