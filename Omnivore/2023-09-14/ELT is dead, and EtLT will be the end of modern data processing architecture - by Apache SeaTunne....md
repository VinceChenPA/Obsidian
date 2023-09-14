---
id: 6d250082-db7f-45e9-b0e0-3e4fd75d730e
---

# ELT is dead, and EtLT will be the end of modern data processing architecture | by Apache SeaTunnel | Jul, 2023 | Dev Genius
#Omnivore

[Read on Omnivore](https://omnivore.app/me/elt-is-dead-and-et-lt-will-be-the-end-of-modern-data-processing--18a91dfde18)
[Read Original](https://blog.devgenius.io/elt-is-dead-and-etlt-will-be-the-end-of-modern-data-processing-architecture-154b87c1cce0)

## Highlights

> EtLT Architecture Overview:
> 
> ![](https://proxy-prod.omnivore-image-cache.app/700x361,s1-CP2vCYp7NdHvXPJj72NIRNq8oe6wSmHhDy9cNgdQI/https://miro.medium.com/v2/resize:fit:1400/1*KaBoPzCzEz1MZ_U2f6k0VA.png)
> 
> Image Source: The author’s own picture
> 
> EtLT splits the traditional ETL and ELT structures and combines real-time and batch processing to accommodate real-time data warehouses and AI application requirements.
> 
> * E(xtract) — Extraction: EtLT supports traditional on-premises databases, files, and software as well as emerging cloud databases, SaaS software APIs, and serverless data sources. It can perform real-time CDC on database binlog and real-time stream processing (e.g., Kafka Streaming) and also handle bulk data reading (multi-threaded partition reading, rate limiting, etc.).
> * t(ransform) — Normalization: In addition to ETL and ELT, EtLT introduces a small “t,” focusing on data normalization. It rapidly converts complex and heterogeneous extracted data sources into structured data that can be readily loaded into the target data storage. It deals with real-time CDC by splitting, filtering, and changing field formats, supporting both batch and real-time distribution to the final Load stage.
> * L(oad) — Loading: The loading stage is no longer just about data loading but also involves adapting data source structures and content to suit the target data destination (Data Target). It should handle data structure changes (Schema Evolution) in the source and support efficient loading methods such as Bulk Load, SaaS loading (Reverse ETL), and JDBC loading. EtLT ensures support for real-time data and data structure changes, along with fast batch data loading.
> * (T)ransform — Conversion: In cloud data warehouses, on-premises data warehouses, or new data federations, business logic is processed. This is typically achieved using SQL, either in real-time or batch mode, to transform complex business rules accurately and quickly into data usable by business or AI applications.
> 
> In the EtLT architecture, different user roles have distinct responsibilities:
> 
> * EtL Phase: Primarily handled by data engineers who convert complex and heterogeneous data sources into data that can be loaded into the data warehouse or data federation. They do not need to have an in-depth understanding of enterprise metric calculation rules but must be proficient in transforming various source and unstructured data into structured data. Their focus is on data timeliness and the accuracy of transforming source data into structured data.
> * T Phase: Led by data analysts, business SQL developers, and AI engineers who possess a deep understanding of enterprise business rules. They convert business rules into SQL statements to perform analysis and statistics on the underlying structured data, ultimately achieving data analysis within the enterprise and enabling AI applications. Their focus is on data logic relationships, data quality, and meeting business requirements for final data results.
> 
> ##  [⤴️](https://omnivore.app/me/elt-is-dead-and-et-lt-will-be-the-end-of-modern-data-processing--18a91dfde18#375848a5-b8fb-49a8-bba5-a5d5ff55d071)  ^375848a5

