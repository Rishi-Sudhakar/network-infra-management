## Sorry for the spell error "comit" instead of "commit" 🥺
# network-infra-management
Network Infrastructure Management with Neo4j and Graph Algorithms ❤️

## Overview

This project aims to manage and analyze a network of infrastructure components using Neo4j, focusing on city nodes connected by road relationships with distances. It includes procedures for data import, graph algorithms for analysis, and periodic data updates.

## Table of Contents

1. [Setup](#setup)
2. [Data Import](#data-import)
3. [Graph Algorithms](#graph-algorithms)
   - [Betweenness Centrality](#betweenness-centrality)
   - [PageRank Algorithm](#pagerank-algorithm)
   - [Shortest Path](#shortest-path)
4. [Periodic Data Updates](#periodic-data-updates)

## Setup 

### Neo4j Installation and Configuration

1. **Install Neo4j Desktop**
   - Download Neo4j Desktop from [neo4j.com/download](https://neo4j.com/download/) and follow installation instructions.

2. **Install Plugins**
   - Open Neo4j Desktop, select your project, and install the following plugins:
     - Graph Data Science Library (GDS)
     - APOC Procedures

3. **Start Neo4j Database**
   - Launch Neo4j Desktop, start your database, and ensure it's running.

## Data Import 

### Importing City Data 😴

- Ensure your CSV file (`cities.csv`) is formatted with columns: `Origin`, `Destination`, `Distance`.

```cypher
// Load the CSV file
LOAD CSV WITH HEADERS FROM 'file:///cities.csv' AS row

// Create nodes for Origin cities
MERGE (origin:City {name: row.Origin})

// Create nodes for Destination cities
MERGE (destination:City {name: row.Destination})

// Create relationships with distance property
MERGE (origin)-[r:ROAD {distance: toInteger(row.Distance)}]->(destination)
```

## Graph Algorithms

### Betweenness Centrality

- Identifies critical nodes based on their influence over the flow of information.
  
```cypher
CALL gds.betweenness.stream('cityGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS city, score
ORDER BY score DESC;
```
<img width="965" alt="Screenshot 2024-06-27 at 10 17 28 PM" src="https://github.com/Rishi-Sudhakar/network-infra-management/assets/79398572/ad7eae4a-a12f-42ee-82cd-2898bc369ec6">



### PageRank Algorithm

- Measures the importance of nodes based on their connections.

```cypher
CALL gds.pageRank.stream('cityGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS city, score
ORDER BY score DESC;
```
<img width="965" alt="Screenshot 2024-06-27 at 10 17 41 PM" src="https://github.com/Rishi-Sudhakar/network-infra-management/assets/79398572/7b76690f-1929-4f2f-b80f-2c6c490cecfd">

### Shortest Path

- Finds the shortest path between two cities using Dijkstra's algorithm. (Feel free to choose any other algorithms, maybe add an algorithm as a cypher file and make a pull request😉)

```cypher
MATCH (start:City {name: 'Agra'}), (end:City {name: 'Jaipur'})
CALL gds.shortestPath.dijkstra.stream('cityGraph', {
  sourceNode: id(start),
  targetNode: id(end),
  relationshipWeightProperty: 'distance'
})
YIELD index, sourceNode, targetNode, totalCost, nodeIds, costs
UNWIND [nodeId IN nodeIds | gds.util.asNode(nodeId).name] AS city
RETURN city, totalCost;
```

## Periodic Data Updates

### Updating Data Automatically

- Use APOC to periodically update timestamps for nodes. DO NOT FORGET TO INSTALL APOC Plugin for this. uwu💕

```cypher
CALL apoc.periodic.commit(
  "MATCH (n:City) SET n.updatedAt = timestamp() WITH count(*) AS cnt RETURN cnt LIMIT 1",
  {limit: 1}
);
```
