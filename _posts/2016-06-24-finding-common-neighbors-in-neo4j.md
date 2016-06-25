---
layout: post
category : neo4j
tags : [neo4j, link-prediction, common-neighbors]
---

{% include JB/setup %}

The number of neighbors shared by two nodes $$x$$ and $$y$$ is the most basic and often among the best predictors of a connection between a source node, $$x$$, and the target node, $$y$$.
Usually this number is formulated as the Common Neighbors index which is just the size of the intersection of the 1-neighborhoods of $$x$$ and $$y$$

\\[
CN(x,y) = | \Gamma(x) \cup \Gamma(y) |
\\]

This index is pretty brilliant since it is easy to find and it is often a surprisingly good predictor. It also forms the basis for a lot of other indices such as Adamic/Adar and the Jaccard index.

Finding the number of common neighbors is luckily very easy in Neo4j. If we have some arbitrary network what we want to do is to find the number of common neighbors between every possible pair of source and target nodes, regardless of whether a link exists between them or not. The first order of the day will be to find the 1-neighborhood of every source node and then say that nodes in the 1-neighborhood of the neighbors are potential target nodes.t

```
MATCH (source)--(neighbor)--(target)
WHERE NOT (source) = (target)
```

The first `MATCH`statement in the query finds all the paths in the network that consists of 3 nodes which is the same as finding every pair of source and target node that share common neighbors. The following `WHERE` statement ensures that the target and source node are not the same, since we'd otherwise end up with a bunch of self-loops in the data.


These two statements are in themselves enough to find the number of common neighbors between nodes in the network, but it is also interesting to know whether a link actually exists between the source and target so we can evaluate the performance of the number of common neighbors as a predictor. This is accomplished by the `OPTIONAL MATCH` statement which tries to find links between the source node and the target node and returns null if no link exists.

```
OPTIONAL MATCH (source) - [con] - target
```

Finally we can return the results of the query. We use `RETURN DISTINCT` to ensure that each source and target node is only returned once (if we didn't then we would get a row for each neighbor as well) and we find the number of common neighbors as the count of neighbor nodes for each pair of source and target nodes. The boolean `IS NOT NULL` statement at the end of the line returns true if a link was found between the source and target and false otherwise.

```
RETURN DISTINCT source.node_id AS source_id, target.verdict_id AS target_id, count(neighbor) AS common_neighbors, con IS NOT NULL as is_connected
```

Together this leads to the following query

```
MATCH (source)--(neighbor)--(target)
WHERE NOT (source) = (target)
OPTIONAL MATCH (source) - [con] - target
RETURN DISTINCT source.node_id AS source_id, target.verdict_id AS target_id, count(neighbor) AS common_neighbors, con IS NOT NULL as is_connected
```

## Common Neighbors in directed, temporal networks

While the above query works fine for an undirected network a little more work is necessary for directed networks. If for example we're talking about a network of scientific papers an outgoing link means a citation from the source to the target paper. In general we can assume that a paper only can cite papers that are older than itself, so the paper publication date is encoded in a property called `created_at`.
This changes the `MATCH` statement to the following

```
MATCH (source)->(neighbor)--(target)
WHERE NOT (source) = (target)
AND source.created_at > target.created_at
```

with the additional clause making sure that paper doing the citing is newer than the cited paper. Adding direction to the link between the source and target makes sense since you won't know who will cite you when you are writing the paper.

All in all this leads to the following statement

```
MATCH (source)->(neighbor)--(target)
WHERE NOT (source) = (target)
AND source.created_at > target.created_at
OPTIONAL MATCH (source) - [con] - target
RETURN DISTINCT source.node_id AS source_id, target.verdict_id AS target_id, count(neighbor) AS common_neighbors, con IS NOT NULL as is_connected
ORDER BY common_neighbors DESCENDING
```

which will return results that look like this

source_id|target_id|common_neighbors|is_connected
A “Rose Is a Rose Is a Rose Is a Rose,” but Exactly What Is a Gastric Adenocarcinoma? | Carbon Monoxide: To Boldly Go Where NO Has Gone Before. | 10 | True
You Probably Think This Paper's About You: Narcissists' Perceptions of Their Personality and Reputation. | A Lucky Catch: Fishhook Injury of the Tongue. | 8 | False
Role of Childhood Aerobic Fitness in Successful Street Crossing. | An-arrgh-chy: The Law and Economics of Pirate Organization | 1 | True

Where we're assuming that the paper title is used as the ID and the results are ordered by descending order of common neighbors.
