# Neo4j and Cypher

GraphGist:
* GraphGist website: http://portal.graphgist.org/about
* Way to run it privately with Docker: http://blog.armbruster-it.de/2013/11/running-neo4j-graphgists-locally-with-docker-io/

Cypher:
* Cypher Language Reference Card: https://neo4j.com/docs/cypher-refcard/current/
* Cypher Full Documentation: http://neo4j.com/docs/developer-manual/current/cypher/

Constraints & Indexes
* https://neo4j.com/docs/developer-manual/current/cypher/schema/constraints/
* https://neo4j.com/docs/developer-manual/current/cypher/schema/indexes/
* Constraints:
  * unique property constraints (nodes) <-- this will ALSO add an index on that property
  * property existence constraints (nodes or relationships) <-- ONLY available in enterprise

For data creation:
* MERGE is good for find_or_create-type behavior: https://neo4j.com/docs/developer-manual/current/cypher/clauses/merge/
* Can use things like ON CREATE and ON MATCH

```cypher
MERGE (keanu:Person { name: 'Keanu Reeves' })
ON CREATE SET keanu.created = timestamp()
RETURN keanu.name, keanu.created
```
