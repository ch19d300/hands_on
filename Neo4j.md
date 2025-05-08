# Neo4j Hands-On Workshop

## Introduction to Graph Databases with Neo4j

This workshop will guide you through practical exercises using Neo4j, a leading graph database that excels at managing highly connected data and complex relationships.

## Workshop Objectives
- Understand graph data models and their advantages
- Learn Cypher query language syntax and patterns
- Design and implement graph data models
- Perform complex graph traversals and pattern matching
- Visualize and analyze connected data

## Connecting to Neo4j

### Using Neo4j Browser (Web Interface)
1. Open your web browser and navigate to: [http://localhost:7474/](http://localhost:7474/)
2. Connect with:
   - Username: neo4j
   - Password: workshop

### Using Cypher Shell (Command Line)
```bash
# From Docker
docker exec -it workshop-neo4j cypher-shell -u neo4j -p workshop

# Direct connection (if Neo4j is installed)
cypher-shell -u neo4j -p workshop -a localhost:7687
```

## Part 1: Graph Database Concepts

### Key Terminology
- **Node**: Entity in the graph (person, place, thing)
- **Label**: Categorizes nodes (like a table in RDBMS)
- **Relationship**: Connection between nodes (directed, with a type)
- **Property**: Key-value pair stored on nodes or relationships
- **Traversal**: Process of following relationships to find connected data

### Visualizing Graph Data
Unlike tables with rows and columns, a graph database represents data as:

```
(NodeA:Label {property1: "value"})--[RELATIONSHIP_TYPE {property: "value"}]-->(NodeB:Label)
```

### Graph Data Model Thinking
When modeling with a graph database:
- Nodes represent entities (nouns)
- Relationships represent connections (verbs)
- Properties store attributes
- Labels group similar nodes

## Part 2: Basic Cypher Query Language

### Creating Nodes and Relationships

```cypher
// Create a single node
CREATE (p:Person {name: "John Doe", age: 35})

// Create multiple nodes
CREATE (a:Product {name: "Smartphone XL", price: 699.99})
CREATE (b:Product {name: "Laptop Pro", price: 1299.99})
CREATE (c:Product {name: "Cotton T-Shirt", price: 19.99})

// Create a node with multiple labels
CREATE (c:Person:Customer {name: "Jane Smith", email: "jane@example.com"})

// Create relationships
MATCH (a:Person {name: "John Doe"}), (b:Product {name: "Smartphone XL"})
CREATE (a)-[:PURCHASED {date: date("2023-05-15")}]->(b)

// Create multiple relationships at once
MATCH (a:Person {name: "Jane Smith"})
MATCH (b:Product {name: "Laptop Pro"})
MATCH (c:Product {name: "Cotton T-Shirt"})
CREATE (a)-[:PURCHASED {date: date("2023-06-10")}]->(b),
       (a)-[:PURCHASED {date: date("2023-06-10")}]->(c),
       (a)-[:REVIEWED {rating: 5, comment: "Great laptop!"}]->(b)
```

### Reading Data with Cypher

```cypher
// Match all nodes with a specific label
MATCH (p:Person)
RETURN p

// Match with property filters
MATCH (p:Product)
WHERE p.price > 500
RETURN p.name, p.price
ORDER BY p.price DESC

// Match with relationship patterns
MATCH (person:Person)-[:PURCHASED]->(product:Product)
RETURN person.name, collect(product.name) AS purchased_products

// Match with relationship properties
MATCH (person:Person)-[purchase:PURCHASED]->(product:Product)
WHERE purchase.date > date("2023-06-01")
RETURN person.name, product.name, purchase.date

// Optional pattern matching
MATCH (product:Product)
OPTIONAL MATCH (product)<-[review:REVIEWED]-()
RETURN product.name, count(review) AS review_count

// Collect and aggregate
MATCH (person:Person)-[:PURCHASED]->(product:Product)
RETURN person.name, 
       count(product) AS product_count,
       collect(product.name) AS products,
       sum(product.price) AS total_spent
```

### Updating Data with Cypher

```cypher
// Update node properties
MATCH (p:Person {name: "John Doe"})
SET p.age = 36, p.last_updated = datetime()

// Add new labels
MATCH (p:Person {name: "John Doe"})
SET p:PremiumCustomer

// Remove labels
MATCH (p:Person:PremiumCustomer {name: "John Doe"})
REMOVE p:PremiumCustomer

// Update relationship properties
MATCH (a:Person)-[r:PURCHASED]->(b:Product)
WHERE a.name = "Jane Smith" AND b.name = "Laptop Pro"
SET r.delivery_status = "Delivered"

// Add properties dynamically
MATCH (p:Product {name: "Smartphone XL"})
SET p += {color: "Black", storage: "128GB"}
```

### Deleting Data with Cypher

```cypher
// Delete a specific node (if it has no relationships)
MATCH (p:Person {name: "Temporary User"})
DELETE p

// Delete a node and all its relationships
MATCH (p:Person {name: "Temporary User"})
DETACH DELETE p

// Delete only relationships
MATCH (a:Person)-[r:VIEWED]->(b:Product)
DELETE r

// Delete all data (be careful!)
MATCH (n)
DETACH DELETE n
```

### Exercise 1: Building a Simple Social Network
Create a small social network graph with:

1. Create at least 5 Person nodes with properties (name, age)
2. Create FRIENDS_WITH relationships between some people
3. Add LIVES_IN relationships to City nodes
4. Query to find friends of friends
5. Find people who live in the same city

## Part 3: Advanced Cypher Queries

### Path Finding and Graph Algorithms

```cypher
// Find the shortest path
MATCH path = shortestPath((a:Person {name: "John"})-[*]-(b:Person {name: "Alice"}))
RETURN path

// Variable length paths
MATCH (a:Person {name: "John"})-[r:FRIENDS_WITH*1..3]-(b:Person)
WHERE a <> b
RETURN b.name, length(r) AS distance

// Path filtering
MATCH path = (a:Person {name: "John"})-[:FRIENDS_WITH*]-(b:Person)
WHERE none(r IN relationships(path) WHERE r.strength < 5)
RETURN b.name
```

### Graph Projections and Aggregations

```cypher
// Count relationships by type
MATCH (a:Person)-[r]->(b)
RETURN type(r) AS relationship_type, count(r) AS count
ORDER BY count DESC

// Find the most connected nodes
MATCH (a:Person)-[r]-()
RETURN a.name, count(r) AS connection_count
ORDER BY connection_count DESC
LIMIT 5

// Group products by category with stats
MATCH (p:Product)
RETURN p.category, 
       count(p) AS product_count, 
       avg(p.price) AS avg_price,
       collect(p.name) AS product_names
ORDER BY product_count DESC
```

### Advanced Pattern Matching

```cypher
// Find common purchases
MATCH (a:Person)-[:PURCHASED]->(p:Product)<-[:PURCHASED]-(b:Person)
WHERE a.name = "John" AND a <> b
RETURN b.name, count(p) AS common_products, collect(p.name) AS products
ORDER BY common_products DESC

// Recommendation pattern
MATCH (a:Person)-[:PURCHASED]->(p1:Product)
MATCH (p1)<-[:PURCHASED]-(other:Person)-[:PURCHASED]->(p2:Product)
WHERE a <> other AND NOT (a)-[:PURCHASED]->(p2)
RETURN p2.name, count(distinct other) AS recommendation_strength
ORDER BY recommendation_strength DESC

// Finding triangles (cliques)
MATCH (a:Person)-[:FRIENDS_WITH]->(b:Person)-[:FRIENDS_WITH]->(c:Person)-[:FRIENDS_WITH]->(a)
RETURN a.name, b.name, c.name
```

### Exercise 2: Advanced Querying
Using the social network you created in Exercise 1:

1. Find all possible paths between two specific people with max length 3
2. Identify "friend clusters" (groups of mutually connected people)
3. Find the most "central" person in the network
4. Write a friend recommendation query based on common friends

## Part 4: Graph Data Modeling

### Domain Modeling with Graphs
A good graph model should:
- Be intuitive and reflect the domain naturally
- Support the most common queries efficiently
- Balance query performance with storage needs
- Consider relationship direction carefully

### Common Graph Patterns

```cypher
// Hierarchical (tree) structure
CREATE (ceo:Employee {name: "CEO"})
CREATE (cto:Employee {name: "CTO"})
CREATE (cfo:Employee {name: "CFO"})
CREATE (dev1:Employee {name: "Developer 1"})
CREATE (dev2:Employee {name: "Developer 2"})
CREATE (ceo)-[:MANAGES]->(cto)
CREATE (ceo)-[:MANAGES]->(cfo)
CREATE (cto)-[:MANAGES]->(dev1)
CREATE (cto)-[:MANAGES]->(dev2)

// Find all employees under a manager
MATCH (m:Employee {name: "CTO"})-[:MANAGES*]->(e:Employee)
RETURN e.name

// Temporal patterns (time tree)
CREATE (y2023:Year {year: 2023})
CREATE (m01:Month {month: 1}), (m02:Month {month: 2})
CREATE (d01:Day {day: 1}), (d02:Day {day: 2}), (d03:Day {day: 3})
CREATE (y2023)-[:HAS_MONTH]->(m01), (y2023)-[:HAS_MONTH]->(m02)
CREATE (m01)-[:HAS_DAY]->(d01), (m01)-[:HAS_DAY]->(d02), (m01)-[:HAS_DAY]->(d03)

// Connect events to time tree
MATCH (d:Day {day: 1})<-[:HAS_DAY]-(m:Month {month: 1})<-[:HAS_MONTH]-(y:Year {year: 2023})
MATCH (e:Event {name: "Conference"})
CREATE (e)-[:OCCURRED_ON]->(d)

// Graph within a graph (meta-graph)
CREATE (g1:Group {name: "Sales Team"})
CREATE (g2:Group {name: "Engineering Team"})
CREATE (p1:Person {name: "Alice"})
CREATE (p2:Person {name: "Bob"})
CREATE (p3:Person {name: "Charlie"})
CREATE (g1)-[:MEMBER {role: "Manager"}]->(p1)
CREATE (g1)-[:MEMBER {role: "Associate"}]->(p2)
CREATE (g2)-[:MEMBER {role: "Engineer"}]->(p2)
CREATE (g2)-[:MEMBER {role: "Lead"}]->(p3)
CREATE (g1)-[:COLLABORATES_WITH {since: date("2022-01-01")}]->(g2)
```

### Exercise 3: Domain Modeling
Choose one of the following domains and create a graph model with sample data:

1. Movie database (actors, directors, movies, genres)
2. E-commerce system (products, customers, orders, reviews)
3. Transportation network (locations, routes, vehicles)

Create at least 10 nodes and appropriate relationships, then write 3 useful queries for your domain.

## Part 5: Graph Algorithms and Analytics

Neo4j has built-in graph algorithms for analyzing connected data.

### Centrality Algorithms

```cypher
// PageRank
CALL gds.pageRank.stream('myGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY score DESC

// Betweenness Centrality
CALL gds.betweenness.stream('myGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY score DESC
```

### Path Finding Algorithms

```cypher
// Shortest path with cost
CALL gds.shortestPath.stream('myGraph', {
  sourceNode: sourceNodeId,
  targetNode: targetNodeId,
  relationshipWeightProperty: 'distance'
})
YIELD nodeIds, costs
RETURN [nodeId IN nodeIds | gds.util.asNode(nodeId).name] AS path, costs

// A* shortest path
CALL gds.shortestPath.astar.stream('myGraph', {
  sourceNode: sourceNodeId,
  targetNode: targetNodeId,
  relationshipWeightProperty: 'distance',
  latitudeProperty: 'lat',
  longitudeProperty: 'lng'
})
YIELD path
RETURN path
```

### Community Detection

```cypher
// Louvain modularity
CALL gds.louvain.stream('myGraph')
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).name AS name, communityId
ORDER BY communityId, name

// Connected components
CALL gds.wcc.stream('myGraph')
YIELD nodeId, componentId
RETURN componentId, collect(gds.util.asNode(nodeId).name) AS nodes
ORDER BY size(nodes) DESC
```

> Note: To use these algorithms with your Docker setup, you'll need Neo4j with the GDS (Graph Data Science) library installed. The examples are shown for reference.

### Exercise 4: Graph Analytics
Using your domain model from Exercise 3:

1. Identify the most important entities using a centrality algorithm
2. Find communities or clusters in your data
3. Calculate the shortest paths between key entities
4. Visualize the results using Neo4j Browser's visualization capabilities

## Part 6: Importing Data into Neo4j

### CSV Import with Cypher

```cypher
// Load CSV with header
LOAD CSV WITH HEADERS FROM 'file:///people.csv' AS row
CREATE (p:Person {
  id: toInteger(row.id),
  name: row.name,
  age: toInteger(row.age),
  email: row.email
})

// Load CSV with relationships
LOAD CSV WITH HEADERS FROM 'file:///friendships.csv' AS row
MATCH (p1:Person {id: toInteger(row.person1_id)})
MATCH (p2:Person {id: toInteger(row.person2_id)})
CREATE (p1)-[:FRIENDS_WITH {since: date(row.since_date)}]->(p2)
```

### Batch Import for Large Datasets

For very large datasets, Neo4j provides command-line tools:

```bash
# Example using neo4j-admin import
docker exec -it workshop-neo4j neo4j-admin database import full \
  --nodes=Person=people.csv \
  --relationships=FRIENDS_WITH=friendships.csv \
  --delimiter="," \
  --array-delimiter=";" \
  --id-type=INTEGER \
  --database=neo4j
```

### Exercise 5: Data Import
1. Create a small CSV dataset (people and relationships)
2. Import the data using Cypher LOAD CSV
3. Verify the import with appropriate queries
4. Create an index on frequently queried properties

## Sample Dataset

For more extensive practice, we'll use the Movies dataset:

```cypher
// Create constraints and indexes
CREATE CONSTRAINT movie_id IF NOT EXISTS FOR (m:Movie) REQUIRE m.id IS UNIQUE;
CREATE CONSTRAINT person_id IF NOT EXISTS FOR (p:Person) REQUIRE p.id IS UNIQUE;
CREATE INDEX movie_title IF NOT EXISTS FOR (m:Movie) ON (m.title);
CREATE INDEX person_name IF NOT EXISTS FOR (p:Person) ON (p.name);

// Load the Movies dataset
:play movies

// Sample queries for the Movies dataset
// Find actors who worked with a director
MATCH (actor:Person)-[:ACTED_IN]->(movie:Movie)<-[:DIRECTED]-(director:Person {name: "Steven Spielberg"})
RETURN actor.name, collect(movie.title) AS movies
ORDER BY size(movies) DESC

// Find the shortest path between two actors (Bacon Path)
MATCH path = shortestPath(
  (bacon:Person {name: "Kevin Bacon"})-[*]-(actor:Person {name: "Tom Hanks"})
)
RETURN [n IN nodes(path) | n.name] AS names, 
       [r IN relationships(path) | type(r)] AS types,
       length(path) AS pathLength

// Movie recommendations based on common actors
MATCH (p:Person {name: "Tom Hanks"})-[:ACTED_IN]->(m:Movie)<-[:ACTED_IN]-(coActor:Person)-[:ACTED_IN]->(recommendation:Movie)
WHERE NOT (p)-[:ACTED_IN]->(recommendation)
RETURN recommendation.title, count(coActor) AS score
ORDER BY score DESC
LIMIT 10
```

## Additional Exercises

1. **Knowledge Graph**: Build a small knowledge graph on a topic of your choice
2. **Fraud Detection**: Implement a simple fraud detection pattern in a transaction network
3. **Social Network Analysis**: Calculate influence scores in your social network
4. **Recommendation Engine**: Create a product recommendation engine based on purchase patterns

## Graph Database vs. RDBMS Comparison Exercise

Create equivalent solutions using both Neo4j and PostgreSQL for the following scenarios:

1. Find friends of friends up to 3 levels deep
2. Find the shortest path between two entities
3. Find all connections between two entities
4. Detect communities or clusters

Compare:
- Query complexity and readability
- Performance characteristics
- Modeling approach
- Scalability considerations

## Resources
- [Neo4j Documentation](https://neo4j.com/docs/)
- [Neo4j Cypher Manual](https://neo4j.com/docs/cypher-manual/current/)
- [Neo4j Graph Academy (Free Courses)](https://graphacademy.neo4j.com/)
- [Neo4j Graph Data Science Library](https://neo4j.com/docs/graph-data-science/current/)
- [Neo4j Example Datasets](https://neo4j.com/developer/example-data/)