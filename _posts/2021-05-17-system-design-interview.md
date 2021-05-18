---
title: "How to answer system design question?"
layout: post
date: 2020-05-03
headerImage: false
tag:
- system design
- interview
blog: true
author: gdf
description: "Procedures to follow when interview system design question"
---

Steps to solve system design interview questions

- Step 1: Clarify the function requirement
    - Functional requirement, what functions do we want to achieve
    - Non-Functional requirement, performance of the system. Ex, Latency requirement, robustness, scalability, resilience, extendability
    - Other requirements
- Step 2: Ask about the capacity estimation and constraints
    - Estimate traffic (Read and Write TPS)
    - Potential Disk usage, Potential Memory usage. 
    - Estimate Bandwidth 
- Step 3: Design System APIs. 
    - What are the Parameters for the APIs
    - What are the APIs we want to support (CURD)
- Step 4: Database Design
    - Database schema
    - Size of a single record
    - What is the primary key, what are the attributes?
    - DataType of each attributes
    - What are the indexes?
    - SQL or non-SQL
- Step 5: Basic System Design and Algorithm
    - List out the pros and cons for different design options
    - Draw Component diagram. 
- Step 6: Data Partitioning and Replication
    - How can we partition the database if we have a large amount of data? 
        - Hash-Based partitioning
- Step 7: Cache
    - Which cache eviction policy should we use? Least Recently Used (LRU)
    - How much memory will we use?
    - How to update the replica of cache?
- Step 8: Load Balancer
    - Most simple version is the round robin approach. 
    - But we can also route traffic according to the avg latency for a single host. Less latency host can get more traffic. 
- Step 9: Purging or cleanup unused data. 
    - Clean data on the fly as part of a request
    - Schedule service to clean up data once in a while
- Step 10: Security and permissions request
    - Who can have permission to access certain resources. 
