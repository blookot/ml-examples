# ml-examples
High level description of Elastic ML jobs

## Introduction
This only document describes (at a macroscopic level) how to create Machine Learning jobs with the Elastic stack.

In the following document, each job is described by a short title, the "detector" part of the job as well as the type of job (among multi-metric, population and advanced, as they appear in the UI).

The fields are named according to the [Elastic Common Schema](https://github.com/elastic/ecs).

You can get further details on the [Elastic ML recipe page](https://www.elastic.co/products/stack/machine-learning/recipes).

## Security

### Web Scraping
Description: data scraping used for extracting data from websites (Wikipedia)

**Source**: Web accces logs (Reverse proxy / LB) or Web server access logs

**Enrichment**: Bots list, User Agents parsing, Reverse DNS

**ML job**
* Nb of hits (RPS): `over user.id` (population)
* Nb of 404: `over user.id` (population)
* Downloaded volume (comparing per session/user id): `sum(network.outbound.bytes) over user.id` (population)


### Unusual visitor

* Unusual UA: `rare by user_agent.original` (advanced job)
* Unusual source IP: `rare by source.ip` (advanced job)
