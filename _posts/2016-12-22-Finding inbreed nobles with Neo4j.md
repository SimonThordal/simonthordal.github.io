---
layout: post
categories : [neo4j]
tags : [neo4j, link-prediction, common-neighbors]
---


There are some things that most people think they know about noble families: That they were rich, rare and horribly inbred. The first two are outside of my expertise, but the genealogy data I implemented in neo4j I am able to look at inbreeding. Particularly we will be able to quantify the degree of inbreeding using the inbreeding coefficient which I will write more about in a later post, but for now it is enough to know that the more common ancestors a couple has, the more inbred their children will be.

## First off a bit of history

Marrying children to close relatives has been done in many cultures through history, sometimes because you wanted wealth and titles to stay in the immediate family and not be lost through inheritance, sometimes because there simply were noone else around and sometimes just because they were royals so [they could do what they wanted to do.](http://ngm.nationalgeographic.com/2010/09/tut-dna/dobbs-text)
On the other end of the spectrum there were plenty of laws against marriages between family or cosanguineous marriages as well. The laws adopted by the early Catholic church forbade marriages where there were [less than four ancestors between husband and wife](http://www.rootsweb.ancestry.com/~medieval/consang.htm) which meant no first cousin marriages allowed, in ancient China marriages were only allowed between people [of different surnames](http://teacup.media/the-china-history-podcast/) and while first cousin marriages were allowed in Islam per example of the Prophet Muhammad [everything
closer than that is not.](https://wikiislam.net/wiki/Cousin_Marriage_in_Islam)

## Measuring inbreeding

As mentioned, the inbreeding coefficient of a child is based on the number of common ancestors between its parents and the distance to the common ancestors, or equivalently the length and number of paths between the parents when the family tree is seen as an undirected graph. Summed over each path, $$p$$, with length $$n$$ the formulation is

$$
F(x) = \sum_p^P \frac{1}{2}^n
$$

The first thing to notice is that there is no limit to the length of the paths that are considered, which can lead to very long execution times and for long path lengths the contribution to $$F(x)$$ quickly becomes insignificant. For these reasons we can set the max path length to 10 and still be relatively on target.


## Results or Who's the derpiest of them all

Now that we have a way to measure inbreeding we can start applying it. I decided to look at every king and emperor and the top ten list of inbred monarchs can found in the table below.
The top guy with an inbreeding coefficient really is a good indicator that out methods are working. Charles the II was famous for suffering from several disabilities, probably due to the massive inbreeding in the Habsburg dynasty. He did not speak before he was four, could not walk before he was eight and when he died the physician performing the autopsy said that his body, "did not contain a single drop of blood; his heart was the size of a peppercorn; his lungs corroded; his intestines rotten and gangrenous; he had a single testicle, black as coal, and his head was full of water ([source](https://www.google.es/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiQ18_5yofRAhVJnRoKHRkDAdcQFggcMAA&url=http%3A%2F%2Fwww.esferalibros.com%2Flibro%2Fenfermedades-de-los-reyes-de-espana-los-austrias%2F&usg=AFQjCNHInbh5j0uS_Hulu6U7zB4eJM5kcg&sig2=XzknlM4CR5YUaDFt7ygplw))."


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Date of Birth</th>
      <th>Dynasty</th>
      <th>Inbreeding coef.</th>
      <th>Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1661-11-06</td>
      <td>Habsburg</td>
      <td>0.176758</td>
      <td>Charles II of Spain</td>
    </tr>
    <tr>
      <td>1857-01-01</td>
      <td>de Bourbon</td>
      <td>0.174805</td>
      <td>Alfonso XII of Spain</td>
    </tr>
    <tr>
      <td>1822-05-13</td>
      <td>de Asis</td>
      <td>0.166016</td>
      <td>Francisco</td>
    </tr>
    <tr>
      <td>1578-04-14</td>
      <td>Habsburg</td>
      <td>0.166016</td>
      <td>Philip III of Spain</td>
    </tr>
    <tr>
      <td>1554-01-20</td>
      <td>Avíz</td>
      <td>0.146484</td>
      <td>Sebastian of Portugal</td>
    </tr>
    <tr>
      <td>1334-08-30</td>
      <td>Ivrea Burgundy</td>
      <td>0.135742</td>
      <td>Peter the Cruel of Castile</td>
    </tr>
    <tr>
      <td>1767-05-13</td>
      <td>de Bragança</td>
      <td>0.132812</td>
      <td>John VI of Portugal</td>
    </tr>
    <tr>
      <td>1069-01-01</td>
      <td>Sanchez</td>
      <td>0.125000</td>
      <td>Pedro I of Aragón</td>
    </tr>
    <tr>
      <td>1073-01-01</td>
      <td>Sanchez</td>
      <td>0.125000</td>
      <td>Alfonso I the battler of Aragón</td>
    </tr>
    <tr>
      <td>1075-01-01</td>
      <td>Sanchez</td>
      <td>0.125000</td>
      <td>Ramiro II the Monk of Aragón</td>
    </tr>
  </tbody>
</table>

So Charles the II really was the poster child of why a homogenous gene pool is a bad thing and that he pops up at the top of the list gives us some confidence that our calculations are correct.

Among the other luminaries on the list we see a lot of Spanish and Portuguese monarchs in particular and as far as I understand from a bit of googling, this was the result of marriages often being arranged to keep the peace between different Iberian families while at the same time attempting to keep the power in their family. Very few modern monarchs show up on this list, probably a result of marrying relatives going out of fashion and the only currently living person in the top-20 is King Olav V of Norway who pops in at a 16th place with an inbreeding coefficient of 0.08, not that that seems to have affected him in the slightest.

It should be noted that there are several problems with the list above. First of all there might be people for whom data is missing, for instance if two people marry and only the parents of one of them are known, then we have no idea of their degree of relatedness. At some point I will probably do some analysis of the completeness of the data, but for now we can assume that the values are lower bounds. Secondly inbreeding is an inherited trait meaning that if your common ancestors are inbred then that leads to a higher degree of inbreeding in your children. This can actually be computed, but the contributions should be relatively small and implementing it would lead to the algorithm becoming recursive which again leads to longer computational times. All of these are issues that I plan on tackling later on, both by adding data to the network and possibly calculating the corrected inbreeding coefficient for a few people to see the magnitude of the differences.


## Neo4j implementation

Now to the Cypher query itself. Given that we want to find the inbreeding coefficient for children of persons `a` and `b`, we find the length of all paths between them up to and including lengths of 10.

```
MATCH (a:Person), MATCH (b:Person)
WHERE a.name = {:name_a}
AND b.name = {:name_b}
OPTIONAL MATCH (a)-[p1:child_of *..5]->(common_ancestor)<-[p2:child_of *..5]-(b)
WITH size(p1) + size(p2) AS path_length, common_ancestor
RETURN DISTINCT path_length, c
```

The first part of this query is pretty self-explanatory

```
MATCH (a:Person), MATCH (b:Person)
WHERE a.name = {:name_a}
AND b.name = {:name_b}
```
simply matches the people by their names which are set by the standard Neo4j parameter syntax.


```
OPTIONAL MATCH (a)-[p1:child_of *..5]->(common_ancestor)<-[p2:child_of *..5]-(b)
```


matches all common ancestors at a maximum distance of 5 to either `a` or `b` and returns the paths. Now, there is an issue with this statement. We wanted to find every path between two people that had a path length smaller or equal to 10, however if one person is removed 4 steps from the common ancestor and the other is 6 steps removed we will not find them using this since p1 and p2 are broken into two. The reason for not just using an undirected relationship like


```
OPTIONAL MATCH (a)-[p2:child_of *..10]-(b)
```


is that by not using directedness we are including descendants in our query, i.e. if two people have a child together the path through the child would be returned. There are ways to fix this, but all of them add significantly to the calculation time and the contribution from the excluded paths will be very small due to path length, so I choose to go with this conservative estimate.
The final part of the statement


```
WITH size(p1) + size(p2) AS path_length, common_ancestor
RETURN DISTINCT path_length, c
```


simply finds the path length as the sum of both paths and returns distinct paths along with their path lengths. If we did not return distinct paths then every path would be counted twice; once from `a` to `b` and once from `b` to `a`.

The results from this query is now plugged into the equation for inbreeding coefficient and voíla, we can now figure out just how inbred every person in our geneaology graph is.
