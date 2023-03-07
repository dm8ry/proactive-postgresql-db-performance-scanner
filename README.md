
Proactive PostgreSQL DB Performance Scanner

Deep night. The phone rings. The critical situation in the system. CPU is reached one hundred percent. There is an urgent need to find a solution.

I think this scenario is familiar to many engineers.

On mission-critical systems, it is important to be proactive in preventing the situation that the key parameters of the system reach maximum values that directly impact performance, stability, and reliability.

Like in medicine, we want to find a cure as soon as the first symptoms appear.

Databases are an essential key element of modern systems.

This blog is about monitoring PostgreSQL database(s).

It describes an approach that makes it easy to identify queries that use the system inefficiently, help to find a root cause for the performance issues, and assist to understand typical workload patterns and performance bottlenecks. The found patterns and queries can be improved and the system will work efficiently and resiliently.

The Proactive PostgreSQL Database(s) Performance Scanner is a script that connects to a database and runs a set of probes that can be extended if desired. All the probes are queries to a database, that are unified by structure.

It includes:

the threshold value,
description of the check,
to which issue this check is associated,
recommendation on how to troubleshoot the issue
SQL query to perform the check
an additional optional SQL query in case there is a need for more evidence
If some check exceeds the threshold value, then the corresponding report will be generated in the following standard form:

description of the check
datetime
environment details
issue
details about the issue
additional evidence
recommendation

The script has the following structure:

the function that executes the probes (mainProcessor)
the function that checks input parameters (helpFunction)
set the number of characters the queries will be cut. It's useful to make output readable in case queries are too long.
populate the environment details.
check the PostgreSQL version. It is useful in case different queries/checks should be performed depending on the version of the DB engine.
check the pg_stat_statements extension is enabled. The script is using it.
expandable set of probes.
The monitoring script has the option to run different types of queries depending on the version of the PostgreSQL database being checked. It's useful when the database metadata structure depends on the version.

If there are several PostgreSQL databases that need to be monitored, the Proactive PostgreSQL Database(s) Performance Scanner script can be run in a loop.

The basic version of the script contains sample checks that can be used for monitoring. It includes probes related to connection utilization, long non-optimal queries, high CPU utilization by queries, etc. It can be extended to any other metrics and indicators that it is important to monitor and check.

```Example of how to run the Proactive PostgreSQL DB Performance Scanner:
proactive_pg_db_performance_scanner.sh -h db_host -p 5432 -U postgres -d postgres
Examples of output:
Check in the pg_stat_statements DB queries that take more than 5000 ms
DateTime: 20230105_112233
Environment: Host:db_host; Port:5432; DB_Username:postgres; DB_Name: postgres
Issue: Long-running queries
Details:
 userid | dbid  |       db_name        | total_time | calls | mean  |             query                                            | chk 
--------+-------+----------------------+------------+-------+-------+----------------------------------------------------------------------
  11111 | 11112 |    my_database_1     |  55555.00  |     1 | 55555 | select * from my_table where a='12345'                       | vwv 
  11111 | 11112 |    my_database_1     |  33333.00  |     1 | 33333 | update my_table set a='12345'                                | vwv 
  11111 | 11112 |    my_database_1     |  11111.00  |     1 | 11111 | delete from my_table where a='12345'                         | vwv 
(3 rows)
Recommendation: Check why the query/queries take so much time. It may be a heavy non-optimized query. Maybe it's an unusual application pattern.```

```Check the queries that occupy more than 15 % of a CPU
DateTime: 20230106_115523
Environment: Host:db_host; Port:5432; DB_Username:postgres; DB_Name: postgres
Issue: Query/queries that utilize significant portion of CPU
Details:
 userid | dbid  |       db_name       |  total_time  |  calls  |    mean  | cpu_portion_pctg |                      query             | chk 
--------+-------+---------------------+--------------+---------+----------+------------------+----------------------------------------+-----
  11111 | 11112 |    my_database_1    | 888799911.12 | 9999999 |    88.88 |            80.00 | select * from my_table where a='12345' | wvw
  11111 | 11112 |    my_database_1    |     99999.99 |       1 | 99999.99 |            20.00 | update my_table set a='12345'          | wvw
(2 rows)
Recommendation: Check why the query/queries take a significant portion of the CPU. Maybe it takes significant time. Maybe it's running too frequently. Try to analyze why this DB query takes a significant part of the CPU.```

```The query/queries that allocates/allocate a significant number of connection slots (Threshold=300)
DateTime: 20230106_120551
Environment: Host:db_host; Port:5432; DB_Username:postgres; DB_Name: postgres
Issue: The most of connection slots are occupied by single query
Details:
 pctg  |        query                                          | num_of_allocated_connection_slots_by_the_query | tot_allocated_slots | chk 
-------+-------------------------------------------------------+------------------------------------------------+---------------------+-----
 55.50 | select * from my_table where a='12345'                |                                            555 |                1000 | wvw
 33.30 | update my_table set a='12345'                         |                                            333 |                1000 | wvw
(2 rows)
Recommendation: Check why a single pattern of queries allocates so many connection slots. It may be application logic, or an unusual application pattern issue.```
