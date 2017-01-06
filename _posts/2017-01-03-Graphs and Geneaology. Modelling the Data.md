---
layout: post
categories : [neo4j, genealogy]
tags : [neo4j, genealogy, nobles, data modeling, data-modeling]
---

Lately I have been writing a little bit about a network I made of the relationships of European nobles without actually describing what the network looks like, so I figured I would amend that with a quick introduction to the database structure.

The source of my data is the genealogical database from [Hull University](http://www.hull.ac.uk/php/cssbct/genealogy/royal/) created by Brian Tompsett. It was scraped directly from the site and since I was mainly interested in Scandinavian royalty I started my search from Harald Bluetooth, the first king of Denmark, and then traversed the graph through children, parents and spouses. I expected this to get me the Scandinavians since the families are quite intermarried, but when looking through the network later on I found even the Caliphs of Baghdad, so I am guessing that I ended up downloading more or less the entire Hull database.

The data is modeled in a fairly simple way. There are 57975 people in the network represented by a `(:Person)` node which can be connected to other people either through `[:married_to]` or `[:child_of]` relationships. Additionally there are 9846 `(:Title)` nodes that people can be connected to through `[:holds]` relationships.
Using the titles as nodes makes sense since I often end up searching for specific titles such as "King of..." or "Queen of Britain" for instance. It would however make sense to split it into a `(:Title)` and `(:Demesne)` where "Demesne" means the actual territory that comes with the title such as Denmark or Northwich. In that way I can avoid using the Neo `STARTS WITH` query for text search and maybe even add GIS coordinates to the demesnes.

The nodes and relationships also have the following properties.

| `(:Person)` | `(:Title)` | `[:married_to]` | `[:holds]` |
| :---------- | :--------- | :-------------- | :--------- |
| name        | name       | date            | acceded_date |
| born        |            |                 |            |
| died ||||
| born_year ||||
| died_year ||||
| family    ||||

The notable part about the properties is that I both include actual dates as a string and the year as an integer. Hopefully the next version of Neo4j will include support for datetime types which will make this a bit less confusing. I may also at some point separate out the family property as a node and create a `[:part_of]` relationship. The most notable missing property is the gender of the people, which really decreases the fun analysis possibilities, but sadly it wasn't available in the source data.

That's basically it for the genealogy network, hopefully I will be able to use it to do some interesting analysis.
