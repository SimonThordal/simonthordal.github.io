---
layout: post
category : neo4j
tagline: "Finding common neighbors in Neo4j"
tags : [neo4j, link-prediction, common-neighbors]
---

{% include JB/setup %}

# Finding common neighbors in neo4j

The number of neighbors shared by two nodes $x$ and $y$ is the most basic and often among the best predictors of a connection between $x$ and $y$.
Usually this number is formulated as the Common Neighbors index whih is just the size of the intersection of the 1-neighborhoods of $x$ and $y$

$$
CN(x,y) = | \Gamma(x) \cup \Gamma(y) |
$$
This index is pretty brilliant since it is easy to find and it is often a surprisingly good predictor. It also forms the basis for a bunch of other indices such as Adamic/Adar and the Jaccard index.

Finding the number of common neighbors is luckily very easy in Neo4j. Let's assume we have a network made of research papers where we just added the node, "Caloric values of electron soups" and we want all the potential citations for this paper along with the common neighbors score for the citations.
First order of the day is finding the set of all the papers cited by our source node and then all the papers that are linked to the members of this set

```
MATCH (source)--(neighbor)--(target)
WHERE NOT (source) = (target)
OPTIONAL MATCH (source) - [con] - target
RETURN DISTINCT source.node_id AS source_id, target.verdict_id AS target_id, count(neighbor) AS common_neighbors, con IS NOT NULL as is_connected
```

The first `MATCH`statement in the query finds all the paths in the network that consists of 3 nodes which is the same as finding every pair of source and target node that share common neighbors. The following `WHERE` statement ensures that the target and source node are not the same, since we'd otherwise end up with a bunch of self-loops in the data.
These two statements are in themselves enough to find the number of common neighbors between papers in the network, but it is also interesting to know whether a link actually exists between the source and target so we can evaluate the performance of the number of common neighbors as a predictor. This is accomplished by the `OPTIONAL MATCH` statement which tries to find links between the source node and the target node and returns null if no link exists.
Finally we can return the results of the query. We use `RETURN DISTINCT` to ensure that each source and target node is only returned once (if we didn't then we would get a row for each neighbor as well) and we find the number of common neighbors as the count of neighbor nodes for each pair of source and target nodes. The boolean `IS NOT NULL` statement at the end of the line returns true if a link was found between the source and target and false otherwise.
