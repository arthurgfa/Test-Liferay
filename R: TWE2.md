# Cluster Replication in Ehcache (EE)

This post wraps up the 2010 performance and optimization work and summarizes what was built over the last few months. It introduces an EE only feature: a new cluster replication mechanism for Ehcache.

Since this feature is only available in the EE version of the portal, the focus here is on the problem and the general approach, not on low-level implementation details.

By default, Ehcache uses an RMI-based, point-to-point model to replicate cache events across nodes. This works for small clusters, but it does not scale well. As the number of nodes grows, each node has to send the same update to many others, which quickly turns into a network bottleneck.

In large platforms such as Liferay, this setup also creates a large number of replication threads. Most of these threads spend most of their time idle, but they still consume memory and add unnecessary CPU overhead.

To improve this, Liferay relies on ClusterLink with multicast support provided by JGroups. This replaces many one-to-one messages with a single broadcast, which is much easier to scale.

The number of threads is also kept under control by using a small, shared pool to dispatch cache events. Because all events go through the same channel, repeated updates to the same cache entry can be grouped and sent once, reducing network traffic.

Recent versions of Ehcache also support JGroups-based replication, which helps with network scalability. However, they still create many threads and do not support grouping updates.

EE customers who want to use this feature can reach out to our support for more details
