layout: post
title: "Configuration flags for PostgreSQL instances"
date: 2023-09-01 22:45:00 -0300
categories: infrastructure postgresql

# Configuration info

[Cloud SQL Configuration flags](https://cloud.google.com/sql/docs/postgres/flags)
[PostgreSQL configuration resource](https://postgresqlco.nf/doc/en/param/max_connections/11/)
[Official documentation](https://www.postgresql.org/docs/11/runtime-config-resource.html)
[PGTune, helper for generating configuration](https://pgtune.leopard.in.ua/)

# Some important flags and general info

## max_locks_per_transaction
This parameter controls the average number of object locks allocated for each transaction; individual transactions can lock more objects as long as the locks of all transactions fit in the lock table. This is not the number of rows that can be locked; that value is unlimited. The default, 64, has historically proven sufficient, but you might need to raise this value if you have queries that touch many different tables in a single transaction, e.g., query of a parent table with many children. This parameter can only be set at server start.

## effective_cache_size
Sets the planner's assumption about the effective size of the disk cache that is available to a single query.
Recommendations are to set Effective_cache_size at 50% of the machine’s total RAM.

## maintenance_work_mem
Especifies the maximum amount of memory to be used by maintenance operations, such as VACUUM, CREATE INDEX, and ALTER TABLE ADD FOREIGN KEY
Total RAM * 0.05

## work_mem
Sets the base maximum amount of memory to be used by a query operation (such as a sort or hash table) before writing to temporary disk files
Sets the limit for the amount of non-shared RAM available for each query operation, including sorts and hashes. This limit acts as a primitive resource control, preventing the server from going into swap due to overallocation.
We can use the formula below to calculate the optimal work_mem value for the database server:
Total RAM * 0.25 / max_connections

## shared_buffers
Sets the amount of memory the database server uses for shared memory buffers
The shared_buffers parameter determines how much memory is dedicated to the server for caching data. 
25% of TOTAL ram or 1/3

## random_page_cost
Sets the planner's estimate of the cost of a nonsequentially fetched disk page
Reducing this value relative to seq_page_cost will cause the system to prefer index scans; raising it will make index scans look relatively more expensive.
If you believe a 90% cache rate is an incorrect assumption for your workload, you can increase random_page_cost to better reflect the true cost of random storage reads. 

## (What is?) WAL
Write-Ahead Logging (WAL) is a standard method for ensuring data integrity. A detailed description can be found in most (if not all) books about transaction processing. Briefly, WAL's central concept is that changes to data files (where tables and indexes reside) must be written only after those changes have been logged, that is, after log records describing the changes have been flushed to permanent storage. If we follow this procedure, we do not need to flush data pages to disk on every transaction commit, because we know that in the event of a crash we will be able to recover the database using the log: any changes that have not been applied to the data pages can be redone from the log records. (This is roll-forward recovery, also known as REDO.)

## wal_buffers
The amount of shared memory used for WAL data that has not yet been written to disk. The default setting of -1 selects a size equal to 1/32nd (about 3%) of shared_buffers, but not less than 64kB nor more than the size of one WAL segment, typically 16MB. 
On very busy, high-core machines it can be useful to raise this to as much as 128MB.
The contents of the WAL buffers are written out to disk at every transaction commit, so extremely large values are unlikely to provide a significant benefit. However, setting this value to at least a few megabytes can improve write performance on a busy server where many clients are committing at once.

## min_wal_size
Sets the minimum size to shrink the WAL to

## max_wal_size
Maximum size to let the WAL grow during automatic checkpoints. This is a soft limit; WAL size can exceed max_wal_size under special circumstances, such as heavy load, a failing archive_command or archive_library, or a high wal_keep_size setting. 
… except for databases that write more than 1GB/hour of data, in which case increase the size of the log so that it's at least an hour worth of logs

## max_worker_processes
Sets the maximum number of background processes that the system can support. This parameter can only be set at server start. The default is 8.
When changing this value, consider also adjusting max_parallel_workers, max_parallel_maintenance_workers, and max_parallel_workers_per_gather.
Increase to max_parallel_workers + other workers, such as workers for logical replication and custom background workers. Not more than your number of cores, though.

## log_min_duration_statement
Sets the minimum execution time above which all statements will be logged
-1 (the default) disables logging statement durations. 

## max_connections
Should be set to the maximum number of connections which you expect to need at peak load. Note that each connection uses shared_buffer memory, as well as additional non-shared memory, so be careful not to run the system out of memory. In general, if you need more than 200 connections, you should probably be making more use of connection pooling.

