---
title: "Things to consider when it comes to performance testing"
layout: post
date: 2022-07-14
headerImage: false
tag:
- performance testing
- backend system
blog: true
author: gdf
description: "This document intends to give an introduction of the performance testing in backend system"
---

## Performance Goals To Consider
- Request/Query per second (RPS)
- Latency
- CPU usage
- Memory usage
- Network usage
- Disk I/O
- Robustness under load 
    - Check Memory leak
    - Check DB Deadlock
    - Check DB heavy/inefficient queries
    - Check DB fail over
    - Check Cluster autoscaling (DB, Search, Cache)
    - Check region/AZ down
    - Check services/DB/Cache/Search recovery
- Simulate the real world scenarios

## Formula

RPS * Latency (In second) = # of concurrent users

Basically, there are two ways to control the load. One is to set the number of RPS, another is to set the number of concurrent users.

||With latency increases|
|---|---|
|RPS fixed|# of concurrent users increase|
|# of concurrent users fixed|RPS decrease|

## How will system performed with the increase of concurrent users

![image](/assets/images/posts/performance.png)

## What should we do to achieve our goals

- Pick the load testing framework to use. (Ex: Locust, JMeter, Gatling, Artillery)
- Develop load test cases. 
- Testing environment preparation: Ideally, the testing environment should have same data quality and system setting as production environment. 
    - (Optional) Seeding test data
    - Service deployment (Dev/Staging/Pre-prod)
- Run the load test and identify the heavy load & buckle zone for the system. 
- Load test data visualization/analysis. 
- Service/DB/Memory/Cache metrics collection/analysis. 
    - Latency
    - CPU/Memtory/Disk IO
    - Error log
    - Response error rate
- (Optional) Integrate the load test in CI/CD pipeline. 
