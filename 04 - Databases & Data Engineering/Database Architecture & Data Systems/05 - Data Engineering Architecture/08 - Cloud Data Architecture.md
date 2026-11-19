---

title: "Cloud Data Architecture"  
aliases:

- "Cloud Architecture for Data"
    
- "Cloud Data Systems"  
    type: learning-note  
    domain: technology  
    category: data-engineering-architecture  
    subject: cloud-data-architecture  
    level: intermediate  
    difficulty: intermediate  
    status: in-progress  
    created: 2026-11-12  
    updated: 2026-11-12  
    tags:
    
- cloud
    
- cloud-computing
    
- data-architecture
    
- data-engineering
    
- databases
    
- storage
    
- analytics
    
- scalability
    
- distributed-systems  
    concepts:
    
- cloud-computing
    
- cloud-data-architecture
    
- compute
    
- storage
    
- networking
    
- scalability
    
- elasticity
    
- managed-services
    
- serverless
    
- object-storage
    
- cloud-database
    
- data-lake
    
- data-warehouse
    
- lakehouse
    
- security
    
- cost-management  
    prerequisites:
    
- "[[01 - Data Architecture Overview]]"
    
- "[[04 - Data Pipelines]]"
    
- "[[05 - Data Warehouses]]"
    
- "[[06 - Data Lakes]]"
    
- "[[07 - Lakehouse Architecture]]"  
    related:
    
- "[[09 - Distributed Data Processing]]"
    
- "[[Data Governance]]"
    
- "[[Data Security]]"
    
- "[[02 - Database Scalability]]"
    
- "[[DOCKER_01_Beginner_Containers_Images_and_Dockerfile]]"  
    tools:
    
- SQL
    
- Python
    
- Docker
    
- PostgreSQL  
    learning_objectives:
    
- "Define Cloud Data Architecture."
    
- "Understand the main cloud infrastructure components."
    
- "Differentiate compute, storage, and networking."
    
- "Explain scalability and elasticity."
    
- "Understand managed services and serverless systems."
    
- "Explain cloud storage architectures."
    
- "Identify common cloud data pipelines."
    
- "Understand cloud security fundamentals."
    
- "Recognize cost management challenges."
    
- "Design a conceptual cloud data architecture."  
    review:  
    frequency: weekly  
    last_reviewed: null  
    next_review: null
    

---

# Cloud Data Architecture

## 1. Introduction

**Cloud Data Architecture** describes how data systems are designed and operated using cloud-based infrastructure and services.

Instead of maintaining all infrastructure locally, organizations can use remote computing resources for:

- Data storage.
    
- Databases.
    
- Data processing.
    
- Analytics.
    
- Machine learning.
    
- Networking.
    
- Security.
    

A simplified architecture looks like this:

```
                 CLOUD PLATFORM
+------------------------------------------------+
|                                                |
|   Storage        Compute        Databases      |
|      |               |               |         |
|      +---------------+---------------+         |
|                      |                         |
|               Data Processing                  |
|                      |                         |
|            Analytics and Applications          |
|                                                |
+------------------------------------------------+
```

The cloud does not replace data architecture principles.

Instead, it provides a different way to deploy and operate data systems.

---

# 2. Why Cloud Architecture?

Traditional infrastructure often requires organizations to manage physical resources.

For example:

```
Organization
     |
     v
Physical Servers
     |
     +--> Storage
     +--> Databases
     +--> Networking
     +--> Maintenance
```

Cloud infrastructure changes this model.

```
Organization
     |
     v
Cloud Services
     |
     +--> Storage
     +--> Compute
     +--> Databases
     +--> Analytics
```

This allows organizations to focus more on building systems and less on managing physical infrastructure.

---

# 3. Core Components

A cloud data architecture usually includes several fundamental components.

```
+--------------------------------------+
|          DATA SOURCES                |
+------------------+-------------------+
                   |
                   v
+--------------------------------------+
|          INGESTION                   |
+------------------+-------------------+
                   |
                   v
+--------------------------------------+
|          STORAGE                     |
+------------------+-------------------+
                   |
                   v
+--------------------------------------+
|          PROCESSING                  |
+------------------+-------------------+
                   |
                   v
+--------------------------------------+
|          CONSUMPTION                 |
|     BI | Analytics | Applications    |
+--------------------------------------+
```

The specific technologies may change, but these architectural responsibilities remain similar.

---

# 4. Compute

**Compute** refers to the resources used to execute programs and process data.

Examples include:

- Virtual machines.
    
- Containers.
    
- Managed processing services.
    
- Serverless functions.
    

Conceptually:

```
Application
     |
     v
+-------------+
|   Compute   |
|             |
| CPU         |
| Memory      |
| Processing  |
+-------------+
```

Compute resources perform the work required by applications and data pipelines.

---

# 5. Storage

**Storage** is responsible for preserving data.

Cloud architectures can use different storage systems depending on the use case.

Common categories include:

- Object storage.
    
- Block storage.
    
- File storage.
    
- Database storage.
    

For data engineering, **object storage** is particularly important.

```
Object Storage
     |
     +--> CSV
     +--> JSON
     +--> Parquet
     +--> Images
     +--> Logs
```

This type of storage is commonly used as the foundation for Data Lakes.

---

# 6. Compute and Storage Separation

An important concept in modern cloud architecture is the separation of:

> **Compute** and **Storage**

Conceptually:

```
        +-------------+
        |   Compute   |
        +-------------+
               |
               v
        +-------------+
        |   Storage   |
        +-------------+
```

Different compute resources can access shared storage.

This can provide flexibility because organizations can scale processing independently from data storage.

---

# 7. Scalability

**Scalability** is the ability of a system to handle increased workload.

For example:

```
Small Workload
     |
     v
+----------+
| 1 Server |
+----------+
```

As demand increases:

```
Large Workload
     |
     v
+----------+  +----------+  +----------+
| Server 1 |  | Server 2 |  | Server 3 |
+----------+  +----------+  +----------+
```

A scalable architecture can increase available resources when required.

---

# 8. Vertical and Horizontal Scaling

There are two common approaches to scaling.

## Vertical Scaling

Vertical scaling means increasing the capacity of a single machine.

```
Before:

+----------+
| Server   |
| 4 CPU    |
+----------+

After:

+----------+
| Server   |
| 16 CPU   |
+----------+
```

This is often called:

> Scaling up.

---

## Horizontal Scaling

Horizontal scaling means adding more machines.

```
Before:

+----------+
| Server 1 |
+----------+

After:

+----------+  +----------+  +----------+
| Server 1 |  | Server 2 |  | Server 3 |
+----------+  +----------+  +----------+
```

This is often called:

> Scaling out.

Distributed data systems frequently rely on horizontal scaling.

---

# 9. Elasticity

**Elasticity** is the ability to automatically increase or decrease resources according to demand.

Example:

```
High Demand
     |
     v
More Compute Resources

Low Demand
     |
     v
Fewer Compute Resources
```

This differs slightly from scalability.

|Concept|Main Question|
|---|---|
|Scalability|Can the system handle growth?|
|Elasticity|Can resources adjust dynamically to demand?|

Cloud environments are particularly useful for elastic workloads.

---

# 10. Managed Services

A **managed service** is a system where the infrastructure provider handles part of the operational responsibility.

For example, instead of manually installing and maintaining a database server:

```
Traditional Database

Organization
    |
    +--> Server setup
    +--> Database installation
    +--> Updates
    +--> Backups
    +--> Monitoring
```

A managed database reduces some of these responsibilities.

```
Managed Database

Organization
    |
    +--> Configure database
    +--> Manage data
    +--> Manage application
```

The infrastructure provider manages more of the underlying system.

---

# 11. Serverless Architecture

**Serverless** does not mean that servers do not exist.

It means that the user does not directly manage the servers required to execute the workload.

Conceptually:

```
Code
 |
 v
Serverless Platform
 |
 v
Execution
```

The platform manages infrastructure and may automatically allocate resources.

Serverless systems can be useful for:

- Event processing.
    
- APIs.
    
- Scheduled tasks.
    
- Lightweight data transformations.
    

---

# 12. Cloud Databases

Cloud environments can provide different database models.

Examples include:

- Relational databases.
    
- Document databases.
    
- Key-value databases.
    
- Analytical databases.
    

A conceptual architecture:

```
Applications
     |
     v
+------------------+
| Cloud Database   |
+------------------+
     |
     v
Persistent Data
```

The database model should still be selected according to the application's requirements.

Cloud deployment does not eliminate the need for good data modeling.

---

# 13. Cloud Data Warehouses

Cloud platforms can provide scalable environments for analytical data warehouses.

Conceptually:

```
Data Sources
     |
     v
Data Pipeline
     |
     v
Cloud Data Warehouse
     |
     v
BI and Analytics
```

A cloud warehouse can support:

- Large analytical queries.
    
- Multiple users.
    
- Historical datasets.
    
- Business Intelligence workloads.
    

---

# 14. Cloud Data Lakes

Cloud object storage can provide a foundation for a Data Lake.

```
+---------------------------+
|      CLOUD STORAGE        |
|                           |
|  Raw Data                 |
|  CSV                      |
|  JSON                     |
|  Parquet                  |
|  Logs                     |
+---------------------------+
```

Processing systems can access the stored information when necessary.

```
Data Lake
    |
    +--> SQL Processing
    +--> Python Processing
    +--> Distributed Processing
    +--> Machine Learning
```

This separation can support large-scale analytical architectures.

---

# 15. Cloud Lakehouse Architecture

A Lakehouse can also be deployed using cloud infrastructure.

```
              CLOUD
                |
                v
        +---------------+
        | Object Storage|
        +-------+-------+
                |
                v
        +---------------+
        | Managed Tables|
        +-------+-------+
                |
                v
      +---------+---------+
      |         |         |
      v         v         v
     BI        SQL       ML
```

The cloud provides scalable infrastructure, while the Lakehouse provides the data architecture.

---

# 16. Cloud Data Pipelines

A typical cloud pipeline may contain several stages.

```
Sources
   |
   v
Ingestion
   |
   v
Raw Storage
   |
   v
Transformation
   |
   v
Curated Storage
   |
   v
Analytics
```

Each stage can potentially use independent services.

For example:

```
API
 |
 v
Ingestion Service
 |
 v
Object Storage
 |
 v
Python / SQL Processing
 |
 v
Analytical Dataset
```

This modular design makes it easier to adapt the architecture as requirements change.

---

# 17. Batch Processing in the Cloud

Batch workloads process data at scheduled intervals.

Example:

```
Every Day
   |
   v
Extract Data
   |
   v
Cloud Storage
   |
   v
Transform
   |
   v
Update Analytics
```

Batch processing is useful when immediate processing is not required.

Examples include:

- Daily reports.
    
- Historical data loads.
    
- Scheduled transformations.
    

---

# 18. Streaming in the Cloud

Streaming architectures process data continuously or with low latency.

```
Events
   |
   v
Streaming Platform
   |
   v
Processing
   |
   v
Storage / Analytics
```

Possible use cases include:

- Sensor data.
    
- Application events.
    
- Financial transactions.
    
- Operational monitoring.
    

Streaming architectures usually introduce additional complexity.

They should be used when low-latency processing is genuinely required.

---

# 19. Security

Cloud data architectures must protect both systems and information.

Important areas include:

## Authentication

Who is attempting to access the system?

## Authorization

What actions is that user allowed to perform?

## Encryption

How is sensitive data protected?

## Network Security

Which systems are allowed to communicate?

## Audit Logs

Who accessed or modified important information?

Security should be considered throughout the architecture.

```
Data
 |
 +--> Access Control
 +--> Encryption
 +--> Monitoring
 +--> Audit
```

---

# 20. Identity and Access Management

Cloud environments often use centralized identity systems to control access.

Conceptually:

```
User
 |
 v
Identity System
 |
 v
Permissions
 |
 +--> Storage
 +--> Database
 +--> Compute
```

A useful principle is:

> Grant only the permissions required to perform a specific responsibility.

This is commonly called the **principle of least privilege**.

---

# 21. Reliability

Cloud architecture should consider what happens when components fail.

A resilient architecture may include:

- Backups.
    
- Replication.
    
- Redundancy.
    
- Monitoring.
    
- Recovery procedures.
    

Conceptually:

```
Primary System
      |
      +---- Failure ----+
                       |
                       v
                 Backup / Replica
```

A reliable architecture anticipates failure rather than assuming that systems will never fail.

---

# 22. Monitoring and Observability

Cloud systems can generate large amounts of operational information.

Important signals include:

- CPU usage.
    
- Memory usage.
    
- Storage consumption.
    
- Query performance.
    
- Pipeline failures.
    
- Application errors.
    

Observability helps answer questions such as:

> What is happening inside the system?

A simplified model is:

```
System
  |
  +--> Metrics
  +--> Logs
  +--> Events
```

Monitoring helps teams identify problems before they become major failures.

---

# 23. Cost Management

Cloud resources are flexible, but flexibility can create cost challenges.

Common cost factors include:

- Storage volume.
    
- Compute time.
    
- Data transfer.
    
- Number of active services.
    
- Query frequency.
    

For example:

```
More Data Processing
        |
        v
More Compute Time
        |
        v
Higher Cost
```

Good cloud architecture considers both technical performance and economic efficiency.

---

# 24. Common Cost Optimization Principles

## Use the Right Resource Size

Avoid allocating more capacity than necessary.

## Scale When Needed

Use additional resources only when workload requires them.

## Manage Data Lifecycle

Move older information to appropriate storage tiers.

## Optimize Queries

Inefficient queries can increase processing costs.

## Monitor Usage

Measure how infrastructure is actually being used.

The goal is not simply to minimize cost.

The goal is to balance:

> Performance + Reliability + Cost

---

# 25. Conceptual Cloud Data Architecture

A complete conceptual architecture may look like this:

```
+--------------------------------------+
|            DATA SOURCES              |
| Databases | APIs | Files | Events    |
+-------------------+------------------+
                    |
                    v
+--------------------------------------+
|             INGESTION                |
| Batch | Streaming | APIs             |
+-------------------+------------------+
                    |
                    v
+--------------------------------------+
|             STORAGE                  |
| Object Storage | Databases           |
+-------------------+------------------+
                    |
                    v
+--------------------------------------+
|            PROCESSING                |
| SQL | Python | Distributed Compute   |
+-------------------+------------------+
                    |
                    v
+--------------------------------------+
|             DATA LAYER               |
| Warehouse | Lake | Lakehouse         |
+-------------------+------------------+
                    |
                    v
+--------------------------------------+
|            CONSUMPTION               |
| BI | Analytics | Applications | ML   |
+--------------------------------------+
```

Security, monitoring, governance, and cost management apply across the entire architecture.

---

# 26. Example: Fleet Data in the Cloud

Imagine a fleet management system with several data sources.

```
Vehicles Database
       |
Maintenance System
       |
Fuel Records
       |
Telemetry
       |
       v
   CLOUD INGESTION
       |
       v
   CLOUD STORAGE
       |
       v
 DATA PROCESSING
       |
       v
 ANALYTICAL DATA
       |
   +---+---+---+
   |   |   |   |
   v   v   v   v
  BI Analytics ML Reports
```

This architecture could support different workloads without requiring every system to run on the same physical infrastructure.

---

# 27. Cloud Architecture and Containers

Containers package an application together with its dependencies.

Conceptually:

```
Application
    |
    v
+------------------+
|    Container     |
|                  |
| Application      |
| Dependencies     |
| Configuration    |
+------------------+
```

Containers can improve consistency between environments.

This connects directly with tools such as:

- Docker.
    
- Container orchestration platforms.
    

A container does not automatically make an application scalable.

Scalability depends on the complete architecture.

---

# 28. Common Architecture Mistakes

## 28.1 Moving Everything to the Cloud Without a Plan

Cloud services should support actual requirements.

## 28.2 Ignoring Cost

Poorly managed resources can create unnecessary expenses.

## 28.3 Ignoring Security

Data protection should not be added only after deployment.

## 28.4 Overengineering

A small application may not need a complex distributed architecture.

## 28.5 Ignoring Monitoring

A system that cannot be observed is difficult to operate.

## 28.6 Treating Managed Services as Magic

Managed services reduce operational work, but architecture and data quality remain important.

## 28.7 Choosing Technology Before Understanding the Problem

A good architecture begins with requirements.

---

# 29. Architecture Design Questions

Before designing a cloud data system, ask:

### Data

- What data exists?
    
- How much data is generated?
    
- How fast does it arrive?
    

### Processing

- Is batch processing sufficient?
    
- Is low latency required?
    

### Storage

- How long must data be retained?
    
- Which formats are appropriate?
    

### Users

- Who consumes the data?
    
- Do they need reports, SQL access, or machine learning?
    

### Reliability

- What happens if a component fails?
    

### Security

- Who should access the data?
    

### Cost

- What resources are actually necessary?
    

These questions should guide technology decisions.

---

# 30. Practical Exercises

## Exercise 1: Identify Cloud Components

For a simple data analytics system, identify:

- Compute.
    
- Storage.
    
- Database.
    
- Processing.
    
- Consumption.
    

Draw the data flow.

---

## Exercise 2: Scale a System

A data pipeline initially processes:

```
10,000 records per day
```

Later, it must process:

```
10,000,000 records per day
```

Explain the possible architectural changes.

Consider:

- Storage.
    
- Compute.
    
- Processing.
    
- Monitoring.
    

---

## Exercise 3: Fleet Cloud Architecture

Design a conceptual cloud architecture for:

- Vehicle inventory.
    
- Maintenance records.
    
- Fuel consumption.
    
- Horometer readings.
    
- Incident information.
    

Include:

```
Sources → Ingestion → Storage → Processing → Analytics
```

---

## Exercise 4: Cost Analysis

Identify three possible sources of unnecessary cloud costs in a large data architecture.

For each one, propose a mitigation strategy.

---

## Exercise 5: Managed vs. Self-Managed

Compare:

```
Self-Managed Database
```

with:

```
Managed Cloud Database
```

Consider:

- Maintenance.
    
- Control.
    
- Scalability.
    
- Operational responsibility.
    

---

# 31. Knowledge Check

1. What is Cloud Data Architecture?
    
2. What is compute?
    
3. What is cloud storage?
    
4. What is object storage?
    
5. What is scalability?
    
6. What is the difference between vertical and horizontal scaling?
    
7. What is elasticity?
    
8. What is a managed service?
    
9. What does serverless mean?
    
10. Why can compute and storage be separated?
    
11. How can a Data Lake use cloud infrastructure?
    
12. What is a cloud data pipeline?
    
13. What is the difference between batch and streaming?
    
14. Why is security important in cloud systems?
    
15. What is the principle of least privilege?
    
16. Why is monitoring important?
    
17. What factors affect cloud cost?
    
18. How do containers relate to cloud architecture?
    
19. What are common cloud architecture mistakes?
    
20. Why should requirements guide technology decisions?
    

---

# 32. Glossary

|Term|Definition|
|---|---|
|Cloud Computing|Delivery of computing resources through remote infrastructure|
|Cloud Data Architecture|Architecture for storing, processing, and consuming data in cloud environments|
|Compute|Resources used to execute programs and process data|
|Storage|Systems used to preserve data|
|Object Storage|Storage model designed to manage data as objects|
|Scalability|Ability to handle increased workload|
|Vertical Scaling|Increasing the capacity of one machine|
|Horizontal Scaling|Adding more machines|
|Elasticity|Dynamic adjustment of resources to workload|
|Managed Service|Service where the provider manages part of the infrastructure|
|Serverless|Computing model where infrastructure is managed by the platform|
|Data Pipeline|System for moving and transforming data|
|Monitoring|Observing system behavior and performance|
|Observability|Ability to understand internal system behavior through generated signals|
|Least Privilege|Granting only the permissions required for a task|
|Replication|Maintaining copies of data or systems|
|Redundancy|Additional components used to improve reliability|

---

# 33. Connections to Other Topics

## Previous notes

- [[01 - Data Architecture Overview]]
    
- [[04 - Data Pipelines]]
    
- [[05 - Data Warehouses]]
    
- [[06 - Data Lakes]]
    
- [[07 - Lakehouse Architecture]]
    

## Next note

- [[09 - Distributed Data Processing]]
    

## Related concepts

- [[02 - Database Scalability]]
    
- [[Data Governance]]
    
- [[Data Security]]
    
- [[DOCKER_01_Beginner_Containers_Images_and_Dockerfile]]
    
- [[Distributed Systems]]
    

## Practical applications

- [[SIAPV Data Architecture]]
    
- [[Fleet Data Analytics]]
    
- [[Data Engineering Fundamentals]]
    

---

# 34. Key Takeaways

1. Cloud Data Architecture uses remote infrastructure to store, process, and manage data systems.
    
2. The main infrastructure components include compute, storage, and networking.
    
3. Compute performs processing, while storage preserves data.
    
4. Modern architectures can scale compute and storage independently.
    
5. Scalability allows systems to handle growth.
    
6. Elasticity allows resources to adjust dynamically to demand.
    
7. Managed services reduce infrastructure management responsibilities.
    
8. Serverless platforms allow users to focus more directly on code and workloads.
    
9. Cloud infrastructure can support Data Warehouses, Data Lakes, and Lakehouses.
    
10. Cloud data pipelines connect sources, storage, processing, and analytical consumption.
    
11. Both batch and streaming workloads can be supported.
    
12. Security must include authentication, authorization, encryption, and auditing.
    
13. Monitoring and observability are essential for operating reliable systems.
    
14. Cloud architecture requires active cost management.
    
15. Technology decisions should always begin with business and technical requirements.
    

---

> [!summary]  
> **Cloud Data Architecture** provides a flexible infrastructure for modern data systems. Its main value comes from combining scalable **compute, storage, managed services, security, monitoring, and data processing** into an architecture that can adapt to changing workloads. The cloud provides the infrastructure; good data architecture determines how that infrastructure is used.