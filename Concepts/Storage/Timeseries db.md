### Time Series Databases (TSDBs) Overview:

#### What are they?
Time Series Databases (TSDBs) are specialized databases optimized for storing, querying, and analyzing time-stamped data points, such as IoT sensor readings, financial tick data, or system logs.

#### Internal Implementation:
- **Optimized for sequential writes**: Data is appended with timestamps.
- **Compression techniques**: Efficient storage using columnar formats, delta encoding, or time-based chunking.
- **Indexing**: Time-based indexing ensures fast reads for queries over ranges of time.
- **Partitioning**: Data is often partitioned by time to manage retention and aging.

#### When to Use:
- **High-frequency data ingestion**: Use TSDBs when you have frequent time-based events.
- **Time-based queries**: Ideal when most queries involve time ranges, such as monitoring system health metrics, financial markets, or IoT sensor data.

#### Trade-offs:
- **Write-heavy workloads**: TSDBs excel in write-heavy environments but might not be ideal for complex, multi-dimensional queries (unless hybridized with other systems).
- **Retention**: They often have built-in data retention policies, expiring older data to manage storage limits.
  
#### Pros:
- **Efficient ingestion** of time-series data.
- **High read performance** for time-range queries.
- **Optimized compression** for better storage efficiency.

#### Cons:
- **Not ideal for general-purpose queries**: They are optimized for time-based queries, not complex relational queries.
- **Limited transaction support**: Some TSDBs may not offer full ACID compliance.

Popular examples include InfluxDB, Prometheus, and TimescaleDB.
