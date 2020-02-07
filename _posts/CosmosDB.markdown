---
layout: post
title:  "Engineering habits"
subtitle: "Compartmetalizing the helpful way"
date:   2019-12-24 22:30:00
categories: [random]
---


Azure Cosmos DB allows developers to choose among the five well-defined consistency models: strong, bounded staleness, session, consistent prefix and eventual. https://docs.microsoft.com/en-us/azure/cosmos-db/consistency-levels-tradeoffs

- Strong -> Linerizable
- Bounded staleness -> The reads might lag behind writes by at most "K" versions (i.e., "updates") of an item or by "T" time interval. Bounded staleness offers total global order except within the "staleness window."
- Session ->  Within a single client session reads are guaranteed to honor the consistent-prefix (assuming a single “writer” session), monotonic reads, monotonic writes, read-your-writes, and write-follows-reads guarantees. Clients outside of the session performing writes will see eventual consistency.
- Consistent prefix -> Updates that are returned contain some prefix of all the updates, with no gaps. Consistent prefix consistency level guarantees that reads never see out-of-order writes.
- Eventual -> There's no ordering guarantee for reads. In the absence of any further writes, the replicas eventually converge.



- Consistency guarantees in practice
In practice, you may often get stronger consistency guarantees. Consistency guarantees for a read operation correspond to the freshness and ordering of the database state that you request. Read-consistency is tied to the ordering and propagation of the write/update operations.

When the consistency level is set to bounded staleness, Cosmos DB guarantees that the clients always read the value of a previous write, with a lag bounded by the staleness window.

When the consistency level is set to strong, the staleness window is equivalent to zero, and the clients are guaranteed to read the latest committed value of the write operation.

For the remaining three consistency levels, the staleness window is largely dependent on your workload. For example, if there are no write operations on the database, a read operation with eventual, session, or consistent prefix consistency levels is likely to yield the same results as a read operation with strong consistency level.

If your Azure Cosmos account is configured with a consistency level other than the strong consistency, you can find out the probability that your clients may get strong and consistent reads for your workloads by looking at the Probabilistically Bounded Staleness (PBS) metric. This metric is exposed in the Azure portal, to learn more, see Monitor Probabilistically Bounded Staleness (PBS) metric.
