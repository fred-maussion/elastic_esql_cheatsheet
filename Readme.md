# ES|QL Query Cheatsheet

This repository aims to collect a comprehensive cheatsheet of common ES|QL queries. The purpose of this repository is to provide real life examples reference guide for analysts who work with Elasticsearch.

##  1. <a name='TableofContents'></a>Table of Contents
<!-- vscode-markdown-toc -->
* 1. [Table of Contents](#TableofContents)
* 2. [Why ES|QL?](#WhyESQL)
* 3. [What's inside this repository?](#Whatsinsidethisrepository)
* 4. [Use case](#Usecase)
	* 4.1. [Tcpdump Runtime Monitoring Across Servers](#TcpdumpRuntimeMonitoringAcrossServers)
	* 4.2. [Follow a transaction based on a correlation id.](#Followatransactionbasedonacorrelationid.)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

##  2. <a name='WhyESQL'></a>Why ES|QL?

ES|QL, the Elasticsearch query language, allows you to craft complex queries to retrieve specific data from your Elasticsearch indices. With ES|QL, you can filter, sort, and aggregate data using various operators and functions.

##  3. <a name='Whatsinsidethisrepository'></a>What's inside this repository?

This repository contains a curated list of ES|QL queries that cover common use cases, such as:

* Filtering and sorting data
* Aggregating and grouping results
* Using geospatial and text-based query types

You can expect to find examples of queries along with csv samples for common use cases like search suggestions, autocomplete, faceting, and more.


##  4. <a name='Usecase'></a>Use case

###  4.1. <a name='TcpdumpRuntimeMonitoringAcrossServers'></a>Tcpdump Runtime Monitoring Across Servers

__Description__ : Track and analyze the runtime of tcpdump processes across servers to monitor system activity and identify unusual execution patterns.

CSV Sample : [file](./esql_tcpdump_runtime_logs.csv)

__Query__

```sql
FROM test_esql 
| WHERE process.name == "tcpdump" AND (event.action == "start" OR event.action == "end")
| KEEP @timestamp, host.hostname, process.name, event.action, process.pid
| EVAL event_time = TO_DATETIME(@timestamp)
| STATS start_time = MIN(event_time), end_time = MAX(event_time) BY host.hostname, process.name, process.pid
| EVAL run_time = DATE_DIFF("minute", start_time, end_time)
| KEEP host.hostname, process.name, process.pid, start_time, end_time, run_time
```

Once done you can create a an alert that will notify you when the tcpdump process is running for more than 15 minutes.

```sql
FROM test_esql 
| WHERE process.name == "tcpdump" AND (event.action == "start" OR event.action == "end")
| KEEP @timestamp, host.hostname, process.name, event.action, process.pid
| EVAL event_time = TO_DATETIME(@timestamp)
| STATS start_time = MIN(event_time), end_time = MAX(event_time) BY host.hostname, process.name, process.pid
| EVAL run_time = DATE_DIFF("minute", start_time, end_time)
| KEEP host.hostname, process.name, process.pid, start_time, end_time, run_time
| WHERE run_time > 15
```

###  4.2. <a name='Followatransactionbasedonacorrelationid.'></a>Follow a transaction based on a correlation id.

__Description__ : Track and analyze a particular transaction for a given user and extract the current status.

CSV Sample : [file](./esql_follow_transaction_id.csv)

```sql
FROM follow_transaction_id 
| WHERE correlation_id == "c4d3e2f1-a5b4-c6d7-e8f9-a0b1c2d3e4f5" // if you want to focus a on particular transaction
| STATS start_time = MIN(@timestamp), end_time = MAX(@timestamp) by correlation_id, event_action, user_name, user_firstname, user_lastname, user_id 
| KEEP user_name, user_firstname, user_lastname, start_time, end_time, correlation_id, event_action
```