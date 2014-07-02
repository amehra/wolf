#Wolf
Trade foreign exchange with ease.

### Introduction
Wolf Trading Platform is an interactive platform that supports real-time financial data visualization, as well as historical data lookup. It executes simple trading rules in real time and is simple to integrate with the brokerage services. It is currently available [here](http://ec2-54-183-118-188.us-west-1.compute.amazonaws.com/wolf/graph.new). This interface was tested on Google Chrome Version 35.0.1916.153.

![alt text](https://github.com/wolf/images/architecture.png "Architecture of Wolf")

Wolf uses the following modules:

  1. Data.provider is responsible for a real-time Forex feed to the system. To have that free of charge it uses data aggregated by [Hist Data](histdata.com) every month. The data feed is therefore one month old, e.g., a tick that happened in June at exactly '2014-06-03 15:32:21.451 EST' is fed to the system on July at exactly '2014-07-03 15:32:21.451 EST'. This approach helps generating traffic similar to what expensive data providers would provide.
  2. Restful.rule.submission is responsible for getting user-defined orders from investors. They are defined as simple trading rues. This module allows investors to specify simple trading rules to the system, they are then delivered to the rule.engine and executed if the condition specified in the order is met.
  3. Data.router is responsible for getting events generated by producers: "ticks" from data.provider and orders from restful.rule.submission and deliver them to the appropriate consumers. It is built on top of [Kafka](https://kafka.apache.org/) distributed queue.
  4. Rule.engine is one of the consumers of events provided by data.router. It matches up rules with the current state of the market and executes the trades when the conditions specified in rules are met. It is built on top of [Storm](http://storm.incubator.apache.org/) event processor.
  5. Data.aggregator.rt is responsible for aggregating the latest ticks in real time. It is built on top of [Cassandra](http://cassandra.apache.org/) database.
  6. Data.aggregator.batch is responsible for aggregating a wide time range of ticks in batch, averaging them, and serving aggregated views to data.aggregator.rt. It is built on top of [HDFS](http://hadoop.apache.org/docs/r1.2.1/hdfs_design.html), [Camus](https://github.com/linkedin/camus), and [Hive](https://hive.apache.org/).
  7. Restful.cache.service.rt separates queries to the data.aggregator.rt database from clients. It uses in-memory cache to serve requests in real time.
  8. Restful.cache.service.batch works exactly like restful.cache.service.rt but for the other types of queries.

### Getting started

I am currently working on the deployment scripts, all the modules should be easy to deploy using the following commands:
```
git clone https://github.com/slawekj/wolf.git
./wolf/<module name>/bin/install.sh
./wolf/<module name>/bin/run.sh
```
Modules should be deployed in the following order:
  1. data.router
  2. data.provider
  3. rule.engine
  4. data.aggregator.rt
  5. data.aggregator.batch
  6. restful.cache.rt
  7. restful.cache.batch
  8. restful.rule.submission
  9. web.interface

Installation scripts are tested on Ubuntu 12.04 distribution, which is available [here](http://releases.ubuntu.com/12.04/ubuntu-12.04.4-server-amd64.iso). You should have git installed:
```
sudo apt-get install git
```
