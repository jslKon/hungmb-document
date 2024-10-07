# TimescaleDB in PostgresSQL

## Hypertables

Hypertables are PostgreSQL tables that automatically partition your data by time

Hypertables improved insert and query performance, and access to useful time-series features

## Hypertable partitioning

When you create and use a hypertable, it automatically partitions data by time, and optionally by space

Each hypertable is made up of child tables called chunks. Each chunk is assigned a range of time, and only contains data
from that range. If the hypertable is also partitioned by space, each chunk is also assigned a subset of the space
values

### Time partitioning

By default, each chunk covers 7 days

```
NOTE: BEST PRACTICE

We recommend setting the chunk_time_interval so that 25% of main memory can store one chunk

If you write approximately 2 GB of data per day and have 64 GB of memory, set the interval to 1 week
```

## Hypertable indexes

The default indexes are:

- On all hypertables, an index on time, descending
- On hypertables with space partitions, an index on the space parameter and time

For unique index -> must contain partition column (For e.g: time col)

## Timescale DB

