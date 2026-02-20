# PostHog Logs Data Loss - February 20, 2026

On February 19th, PostHog's Logs ClickHouse database experienced a bug which caused it to delete a significant proportion of our Logs Product data in our US Cloud region

## Summary

For PostHog's new Logs product, we store data in a ClickHouse cluster. When we started the Logs a few months ago we made the decision to run Logs on its own ClickHouse cluster instead of adding it to our main ClickHouse cluster where we store analytics events and other PostHog data. This was so that we could move quicker, use later versions, and optimise the database for Logs specific access patterns, as well as isolating the impact of bugs or load between different products

This new cluster uses S3 disks in ClickHouse and data parts are automatically uploaded to S3 after 24 hours - this is to allow us to scale to the enormous amount of volume required for Logs, our own internal log consumption is over 500MB/s, or about 1PB/month (compressed it's about ~100TB/month)

A bug in ClickHouse caused it to unexpectedly start deleting almost all of the data parts in S3. The database is replicated with two replicas, however very early on in the project we had set "Zero Copy Replication" on in the cluster. This is an experimental feature that ClickHouse do not recommend in production, for exactly this reason: A bug that should have caused a single replica to be deleted instead deleted the data for both replicas.

## Timeline

All times in UTC.

- **Feb 19: 10:54**: A routine mutation was run to materialize an index
- **Feb 19: 11:02**: The mutation finished, this triggered a bug in ClickHouse's zero-copy replication which caused one of the replicas to erroneously believe all of the data parts in the database were no longer referenced
- **Feb 19: 11:02-19:40**: During this time the replicas were busy diligently deleting all of the data stored in S3 for the entire database. As data is only moved to S3 after 24 hours, and the vast majority of our queries are for recent data, no automated alarms were triggered as the volume of query errors was relatively low
- **Feb 19: 19:40**: One of the nodes in the cluster crashes and restarts, it fails to start up due to the large volume of missing data it can't find. Engineers investigate and after some checks discover that the vast majority of S3-backed data is missing
- **Feb 19: 21:45**: It is determined that the data is most likely unrecoverable - disaster recovery procedures start
- **Feb 19: 22:15**: Decision is made to cut over to a new table and restore data from Kafka, where we have 3 days of retention
- **Feb 19: 23:00**: We have switched over to the new table (without zero-copy replication) and caught up on recent messages.
- **Feb 20: 10:05**: Data backfill from Kafka history begins
- **Feb 20: 12:21**: Data backfill completes

## Root Cause Analysis

### Zero Copy replication bug

The decision to use zero-copy replication was taken extremely early in the Logs product development when it was an experimental internal-only tool.

Once Logs was released to external users this decision should have been revisited, but wasn't. Due to experiencing no issues at all during several months of internal usage, settings that had been set at the beginning were largely unvisited and unchanged.

Zero-copy replication has been largely unmaintained for the last 4 years and still contains critical bugs, which includes deleting the entire database erroneously. Because Zero Copy replication utilizes a shared storage medium (S3) for multiple replicas, when the logic on one node failed and issued delete commands for the underlying S3 objects, those files were removed for the entire cluster immediately. There was no redundancy layer between the database application logic and the storage layer.

### Lack of Detection

We lacked specific monitoring for the integrity of "cold" data stored in S3. Our alerts are optimized for ingestion lag, query latency, and error rates on active queries. Since users rarely query logs older than 24 hours, and the deletion process happened silently in the background without throwing application-level errors, the system remained "green" on our dashboards until the node restart forced a consistency check.

## Lessons Learned

### What Went Well

*   **Service Isolation:** The decision to host the Logs product on a completely separate ClickHouse cluster from the main Analytics product was validated. Despite the severity of this incident, our core Product Analytics and Session Replay features remained 100% unaffected.
*   **Kafka Retention Strategy:** Configuring Kafka with 3 days of retention saved us from total data loss for recent activity.

### What Went Poorly

*   **Configuration Lifecycle Management:** Experimental configurations (Zero Copy Replication) intended for MVP/Alpha stages were allowed to persist into production
*   **Silent Failure:** The system deleted petabytes of data over an 8-hour window without a single alarm firing. We were blind to the deletion of historical data because we only monitor the health of *incoming* data and *hot* data.
*   **Backup Strategy:** Relying solely on the database replication for data durability (when using shared storage) created a single point of failure. We did not have S3 Versioning enabled on the bucket, which would have allowed us to "undelete" the files removed by the application.

### Key Takeaways

1.  **Immediate Configuration Audit:** Disable Zero Copy Replication on all clusters immediately. Conduct a full audit of the Logs ClickHouse configuration and ensure no experimental features are used in production.
2.  **Implement S3 Object Protection:** Enable S3 Versioning on the underlying storage buckets. This ensures that even if the database application issues a destructive command due to a bug, the underlying data objects can be recovered.
3.
