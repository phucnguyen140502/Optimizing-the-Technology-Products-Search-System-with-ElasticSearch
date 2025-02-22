# Optimizing the Technology Products Search System with ElasticSearch

## Introduction
Our technology retail system offers products such as laptops, mobile phones, and other electronic devices, with over **5,000,000** items in total. Initially, all data was stored and queried through PostgreSQL. However, due to the massive volume of data, users experienced significant delays when searching for products based on their keyword inputs, which adversely impacted the user experience.
```sql
SELECT count(*)
FROM products_data pd;
```
![image](https://github.com/user-attachments/assets/506f4da6-84f2-4e59-ba7a-db2cd01d0fbe)


## Initial Challenges
- **Slow Query Performance:** When users search with keywords (e.g., "gaming laptop"), PostgreSQL must scan through millions of records, resulting in response times that can range from several seconds to even tens of seconds.
- **High System Load:** Complex queries on such a large dataset place a heavy load on the system, affecting other critical operations.
- **Limited Scalability:** As the data volume grows, maintaining query performance on PostgreSQL becomes increasingly challenging.


## The Solution: Integrating ElasticSearch for Search Queries
To address these issues, we enhanced the system by integrating ElasticSearch with Kafka for data synchronization. The key components of this solution are:

- **PostgreSQL:** Continues to serve as the core system for handling product **updates** and **inserts**. The original product data is maintained here.
- **Kafka:** Acts as the streaming channel that transmits data from PostgreSQL to ElasticSearch. Both PostgreSQL and Kafka are installed on the server with IP address **172.30.2.75**.
- **ElasticSearch:** Handles the indexing of data and optimizes full-text search queries, enabling rapid response times. ElasticSearch is installed on the server with IP address **172.30.2.76**.

### Workflow
1. **Data Ingestion:** Each time a product is updated or inserted in PostgreSQL, the change is recorded.
2. **Streaming via Kafka:** The new data is streamed into Kafka.
3. **Synchronization with ElasticSearch:** Kafka forwards the data to ElasticSearch, which updates its indices in near real-time.
4. **Fast Query Execution:** When a user performs a search, ElasticSearch processes the query and returns results in under one second.

## System Architecture
![image](https://github.com/user-attachments/assets/f5f5eb8c-809e-4778-8b1d-708cd845b91b)


- **PostgreSQL:** Manages the original product data and handles all update/insert operations.
- **Kafka:** Ensures continuous, reliable data streaming between PostgreSQL and ElasticSearch.
- **ElasticSearch:** Optimizes search queries using advanced indexing and full-text search techniques.

## Performance Comparison
### Before Implementation:
- **PostgreSQL Queries:** In some cases, searching via PostgreSQL can take tens of seconds (for example, **~23.8 seconds** in the screenshot) due to extensive data scanning.
- **User Experience Impact:** Such prolonged wait times diminish the overall user experience and lower customer satisfaction.
```sql
explain analyze
select *
from products_data pd
where product_name ilike '%Xiaomi%'or brand ilike '%Xiaomi%' 
or category ilike '%Xiaomi%' or cpu ilike '%Xiaomi%' or gpu ilike '%Xiaomi%';
```
![image](https://github.com/user-attachments/assets/0d54d563-7495-44e3-bad7-20105c082261)

### After Implementation:
- **Elasticsearch Queries:** With the help of optimized indexing and full-text search, query response times can drop to under 1 second (e.g., ~496 ms in the screenshoot).
- **Improved Performance and Scalability:** This solution can efficiently handle large data volumes, ensuring rapid responses even as the dataset grows.
- **Enhanced User Experience:** Fast and accurate search results significantly improve overall satisfaction, providing users with near-instant feedback.


![image](https://github.com/user-attachments/assets/17d7a203-480f-4d00-b28a-2188984c8bd4)

![image](https://github.com/user-attachments/assets/916d45c5-42ea-4dc6-a860-c19a9c8c0fdc)

## Conclusion
Integrating ElasticSearch through a Kafka-driven data streaming pipeline has effectively resolved the slow query performance in our technology retail system, which manages over 5 million products. The key benefits of this architecture include:

- **Reduced Query Time:**  
  - **PostgreSQL:** The query took around **3.28 seconds** (`Execution Time: 3283.395 ms`).  
  - **Elasticsearch:** The same search request was processed in about **496 ms** (`"took": 496`).  
  This shift from multiple seconds to under half a second dramatically improves the user experience.

- **Increased Efficiency:**  
  By offloading search queries to Elasticsearch, PostgreSQL can focus on update/insert operations. This optimization not only reduces the load on the core database but also speeds up overall data processing.

- **Scalability:**  
  The new architecture scales seamlessly with growing data volumes, ensuring that search performance remains high and system stability is maintained even under increased load.


| **Metric**                  | **PostgreSQL**                                                                 | **Elasticsearch**                           |
|-----------------------------|-------------------------------------------------------------------------------------------|--------------------------------------------------------|
| **Query Method**            | Sequential scan on `products_data` with filters on `brand`, `category`, `op`, etc.       | Full-text search on indexed documents                 |
| **Execution/Response Time** | **3283.395 ms** (as shown in the `Execution Time`)                                       | **496 ms** (from `"took": 496`)                        |
| **Observation**             | Sequential scans lead to higher latency when dealing with large datasets.               | Elasticsearch’s indexing and full-text search greatly reduce query time. |

This architectural enhancement not only improves query speed but also enhances user satisfaction and positions the system for future scalability.
