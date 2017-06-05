---
layout: post
title: "BigQuery: Google style OLAP for your business"
---

I've come across [BigQuery](https://cloud.google.com/bigquery/) (BQ) at work a few times recently - mostly in the context of our data engineering team enabling data analysts to ask questions of our data. I've also seen a few cases of teams thinking of leveraging it in some job oriented operational tasks due to its low latency querying capabilities. Needless to say this piqued my interest and I got hold of 2 papers by Google talking about the secret sauce behind their tech. One is a about BQ itself as available through [Google Cloud Platform](https://cloud.google.com/) (GCP); the other is about the internal Google tool [Dremel](https://en.wikipedia.org/wiki/Dremel_(software)) that BQ is based on.

## BigQuery

White Paper: [An Inside Look at Google BigQuery](https://cloud.google.com/files/BigQueryTechnicalWP.pdf)

BigQuery allows users to run ad-hoc SQL-like queries against extremely large data sets to get results in seconds.

> "Dremel Can Scan 35 Billion Rows Without an Index in Tens of Seconds"

So it actually scans ALL the data and doesn't use any indexes.

### It does this by:

- Using columnar storage - individual records / rows are split into their constituent columns and these columns are stored on different storage volumes.
    - This minimizes traffic - only the columns needed are read
    - It compresses better (than row storage - 10x vs 3x) - since columns have similar values; this is especially true for columns with low cardinality
    - However this is worse for updates - which is fine since Dremel is meant for read-only [OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing) / [BI](https://en.wikipedia.org/wiki/Business_intelligence) use cases
- Tree architecture - It uses a parallel, tree-based, distributed scatter-gather approach to execute queries and retrieve the data

### Comparison to [MapReduce](https://en.wikipedia.org/wiki/MapReduce) (MR):

- Unlike MR, Dremel is meant for interactive analysis (ad-hoc querying)
- MR is meant for large scale batch processing (complex [unstructured] data and processing logic - e.g. Machine Learning / data mining)

### Datawarehouse solutions for OLAP / BI typically fall into the following buckets:

- R(elational)OLAP
    - Uses indexes that need to be pre-defined for all potential ways in the which the data might be queried
    - Needs specialized hardware for large data
- M(ultildimensional)OLAP
    - Needs predefined dimensions
    - Potentially expensive BI engineers needed to design data marts
    - Brittle to change
- Full scan of data (what BQ does)
    - No indexes / data design needed
    - True ad-hoc querying
    - Achieved through high disk IO throughput
        - In memory DB or flash storage - a lot of traditional OLAP appliances use this but it can get expensive (100s of 1000s of $) fast
        - Columnar storage (BQ does this)
        - Parallel disk IO
            - Traditional OLAP does this using proprietary hardware / specialized storage units which may also get expensive
            - BQ does this by using Google's datacenter infrastructure as a part of GCP - the query is sent to a tree of (potentially 1000s of) machines that co-ordinate and do the work (in parallel) and then collect the results and return to the client

### Getting data into BQ:

- Upload data to [GCS](https://cloud.google.com/storage/)
- Import files into BQ using a command-line tool, Web UI or API

### Punchline(s) from the paper (emphasis mine):

> "BigQuery requires ***no capacity planning, provisioning, 24x7 monitoring or operations, nor does it require manual security patch updates***. You simply upload datasets to Google Cloud Storage of your account, import them into BigQuery, and let Google’s experts manage the rest. This ***significantly reduces your total cost of ownership (TCO)*** for a data handling solution."

The paper also provides an example of the above:

> "***Wikipedia query example*** we explored at the beginning of this paper. If you execute the query on BigQuery, it would cost you just ***$0.32 for each query plus $4.30 per month*** for Google Cloud Storage"

Earlier in the paper they mention:

> "This “wikipedia” table holds all the change history records on Wikipedia’s
article content and consists of ***314 millions of rows – that’s 35.7GB***."

## Dremel

Paper: [Dremel: Interactive Analysis of Web-Scale Datasets](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36632.pdf)

- > "scalable, interactive ad-hoc query system for analysis of read-only nested data"
- > "The system scales to thousands of CPUs and petabytes of data, and has thousands of users at Google"
- > "dealing with stragglers and failures is essential for achieving fast execution and fault tolerance"
- > "The data used in web and scientific computing is often nonrelational. Hence, a flexible data model is essential in these domains. … Normalizing and recombining such data at web scale is usually prohibitive"
- Unlike [Pig](https://en.wikipedia.org/wiki/Pig_(programming_tool)) and [Hive](https://en.wikipedia.org/wiki/Apache_Hive), Dremel executes queries natively without converting them to MapReduce jobs
- It uses column striped storage with a nested data model to query data in-situ (in place, unlike [ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load))
- An algorithm for encoding this nested ([protobuf](https://en.wikipedia.org/wiki/Protocol_Buffers) serialized) data in a columnar format is provided
- Has a SQL-like query language which also allows peeking at paths into records
- Tree based query distribution (similar to Info Retrieval / Search problems!) with query rewriting and leaves talking to storage layer (e.g. [GFS](https://en.wikipedia.org/wiki/Google_File_System)) / local storage. The query leaves do the processing.
- A query dispatcher:
    - Does prioritization and load balancing. There are often more tablets to be processed than ‘slots’ (a slot is a thread running on a leaf node)
    - Reschedules processing taking too long (referred to above as the 'stragglers')
- Each node has an internal execution tree / plan with other optimizations
- Has a threshold for the min % of tablets to be scanned before returning a result - this is very useful since at this scale tail latencies tend to dominate ([The Tail at Scale](http://www.cs.duke.edu/courses/cps296.4/fall13/838-CloudPapers/dean_longtail.pdf)) - the sample queries given in the paper finish processing 99% of tablets under 1-2s
- The paper details a whole bunch of experimental improvements showcasing the positive effect of design choices made and emphasizing Dremel’s interactivity

