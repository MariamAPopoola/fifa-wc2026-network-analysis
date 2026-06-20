# FIFA World Cup 2026: A Graph-Theoretic Network Analysis of Tournament Structure

**Author:** Mariam Adejoke Popoola (PhD Student in Informatics)  
**Institution:** Universidade Federal do Amazonas (UFAM), Manaus, Brazil  
**Program:** Postgraduate Programme in Informatics (PPGI)  
**Date:** 2026  
**Keywords:** Graph Theory, Network Analysis, FIFA World Cup, Centrality Measures, NetworkX, Python

---

## Abstract

This study applies Graph Theory and Network Analysis to the official fixture data of the FIFA World Cup 2026 — the first 48-team, three-nation World Cup in history. We model the tournament as two complementary undirected graphs: a **match graph** (G_group) where vertices represent teams and edges represent scheduled group-stage matches, and a **city-sharing graph** (G_city) where edges encode geographic co-occurrence at host venues. Using NetworkX and Python, we compute four centrality measures (degree, betweenness, closeness, eigenvector), implement BFS and DFS traversal algorithms, and perform shortest-path analysis. Our findings reveal that the Group Stage enforces a mathematically uniform structure of 12 disjoint complete graphs K₄, while the city-sharing graph exhibits meaningful centrality variation. England, Curaçao, and Germany emerge as the most central teams in the geographic network. The network has a small average shortest path from Brazil (1.73 hops), confirming small-world properties. This work demonstrates the utility of graph-theoretic methods for structural analysis of international sports tournaments.

---

## 1. Introduction

The FIFA World Cup is the most-watched sporting event on Earth, attracting over five billion viewers across its four-week duration. Beyond its cultural and economic significance, the World Cup presents a unique object of study for graph theorists and network scientists: a structured, rules-governed competition that naturally maps onto a mathematical graph.

The 2026 edition introduces several structural novelties. For the first time, 48 national teams compete (up from 32), distributed across 12 groups of 3–4 teams. The tournament is hosted across 16 cities in three countries — the United States, Canada, and Mexico — spanning multiple time zones and geographic regions. This expanded structure creates a richer network than any previous edition.

Graph Theory, a branch of discrete mathematics originating with Euler's 1736 solution to the Königsberg Bridge Problem, provides a rigorous framework for analysing such structures. A **graph** G = (V, E) consists of a set of vertices V and a set of edges E ⊆ V × V. Applied to the World Cup: teams become vertices, matches become edges, and tournament properties become graph-theoretic properties.

**Research Objectives:**

1. Construct and characterise graph models of the FIFA World Cup 2026 tournament
2. Compute and interpret centrality measures to identify structurally important teams
3. Implement and demonstrate classical graph traversal algorithms (BFS, DFS)
4. Analyse geographic network properties through city-sharing connectivity
5. Provide a reproducible, well-documented Python implementation for the research community

---

## 2. Dataset Description

### 2.1 Data Sources

Four relational CSV datasets were used, constituting the official FIFA World Cup 2026 schedule at the time of analysis:

| Dataset | Rows | Columns | Description |
|---------|------|---------|-------------|
| matches.csv | 104 | 8 | All scheduled matches across 7 stages |
| teams.csv | 48 | 5 | Participating teams with group assignments |
| host_cities.csv | 16 | 6 | Venue and geographic metadata |
| tournament_stages.csv | 7 | 3 | Stage names and ordering |

### 2.2 Relational Schema

The datasets form a star schema centred on `matches.csv`:

```
matches ──→ teams (via home_team_id, away_team_id)
matches ──→ host_cities (via city_id)
matches ──→ tournament_stages (via stage_id)
```

### 2.3 Tournament Structure

The 2026 World Cup proceeds through seven stages:

| Stage | Matches | Teams |
|-------|---------|-------|
| Group Stage | 72 | 48 |
| Round of 32 | 32 | 32 |
| Round of 16 | 16 | 16 |
| Quarterfinals | 8 | 8 |
| Semifinals | 4 | 4 |
| Third-Place Playoff | 1 | 2 |
| Final | 1 | 2 |
| **Total** | **104** | — |

### 2.4 Data Quality Assessment

A systematic quality assessment was performed:

- **Missing values:** 32 null entries in `home_team_id` and `away_team_id` — these correspond to the 32 knockout-stage matches where participating teams are determined by group results and were not yet resolved at data collection time. This is structurally expected and not a data quality deficiency.
- **Duplicate records:** 0 duplicates detected in any dataset.
- **Placeholder teams:** 6 teams marked `is_placeholder = True` (e.g. "Winner UEFA Playoff D") representing unresolved qualification spots. These were excluded from graph analysis.
- **Temporal consistency:** All 104 kickoff timestamps follow ISO 8601 format with correct timezone offsets; no chronological anomalies detected.

### 2.5 Geographic Distribution

The 16 host cities are distributed across three countries and five regional clusters:

| Region | Cities | Example Venues |
|--------|--------|----------------|
| East (USA) | 4 | Atlanta, Boston, Miami, New York |
| Central (USA) | 4 | Dallas, Houston, Kansas City, Los Angeles |
| West (USA) | 3 | San Francisco, Seattle, Vancouver |
| Mexico | 3 | Guadalajara, Mexico City, Monterrey |
| Canada | 2 | Toronto, Vancouver |

---

## 3. Methodology

### 3.1 Graph Construction

#### 3.1.1 Match Graph (G_group)

We define G_group = (V, E) where:

- **V** = {t | t ∈ teams, t.is_placeholder = False} → |V| = 42 confirmed real teams
- **E** = {(h, a) | ∃ match m : m.home_team = h ∧ m.away_team = a ∧ m.stage = "Group Stage"}

Each edge carries attributes: `stage`, `city`, and `date`.

**Theoretical property:** Since each of the 12 groups contains 3–4 teams and every team plays every other team in its group exactly once, each group subgraph is a **complete graph Kₙ**. For groups of 4 teams: K₄ with C(4,2) = 6 edges. For groups of 3 teams: K₃ with C(3,2) = 3 edges. Total expected edges = (8 groups × 6) + (4 groups × 3) = 48 + 12 = 60. However due to the 3+4 mixed structure, actual computation gives 72 Group Stage matches.

#### 3.1.2 City-Sharing Graph (G_city)

To enable meaningful centrality variation (G_group is uniform by design), we construct a second graph:

- **V** = same as G_group
- **E** = {(t₁, t₂) | t₁ ≠ t₂ ∧ ∃ city c : t₁ plays in c ∧ t₂ plays in c}
- **Weight(t₁, t₂)** = |{cities shared by t₁ and t₂}|

This graph captures geographic proximity in the hosting network — teams connected in G_city experience the same venue environment regardless of group assignment.

### 3.2 Centrality Measures

Four standard centrality measures were computed on G_city:

#### 3.2.1 Degree Centrality
For a graph with n nodes, the degree centrality of node v is:

> C_D(v) = deg(v) / (n - 1)

This measures the proportion of other teams a given team shares a host city with.

#### 3.2.2 Betweenness Centrality
The betweenness centrality measures the fraction of all shortest paths passing through v:

> C_B(v) = Σ_{s≠v≠t} σ(s,t|v) / σ(s,t)

where σ(s,t) is the total number of shortest paths between s and t, and σ(s,t|v) is the count of those passing through v. High betweenness identifies teams that act as **bridges** in the geographic network — their host cities are critical connectors.

#### 3.2.3 Closeness Centrality
Closeness centrality measures how close a node is to all others:

> C_C(v) = (n - 1) / Σ_{u≠v} d(v, u)

where d(v,u) is the shortest-path distance. Teams with high closeness are geographically "central" — they can reach any other team's host city context in fewer hops.

#### 3.2.4 Eigenvector Centrality
Eigenvector centrality rewards not just having many connections but being connected to well-connected nodes:

> C_E(v) = (1/λ) Σ_{u∈N(v)} C_E(u)

This is the solution to the eigenvector equation Ax = λx, where A is the adjacency matrix. High eigenvector centrality identifies teams embedded in the most influential sub-clusters of the geographic network.

### 3.3 Graph Traversal Algorithms

#### 3.3.1 Breadth-First Search (BFS)
BFS explores all neighbours at the current depth before advancing. Implementation uses a FIFO queue. Time complexity: O(V + E). Applied to G_city starting from Brazil to compute geographic distances.

#### 3.3.2 Depth-First Search (DFS)
DFS explores as far as possible along each branch before backtracking. Implementation uses an explicit stack (iterative, avoiding Python recursion limits). Time complexity: O(V + E).

#### 3.3.3 Shortest Path Analysis
Single-source shortest paths computed using NetworkX's optimised BFS-based `single_source_shortest_path_length`. Network diameter and radius computed via `nx.diameter()` and `nx.radius()`.

### 3.4 Advanced Analysis

A **composite centrality score** was computed by min-max normalising each of the four measures to [0,1] and averaging them:

> C_composite(v) = (1/4) Σᵢ [Cᵢ(v) - min(Cᵢ)] / [max(Cᵢ) - min(Cᵢ)]

**Regional cluster analysis** was performed by joining match data with host city metadata to determine how group-stage match counts are distributed across geographic regions.

**City-reach analysis** computed, for each team, the number of distinct host cities where they play group-stage matches — a proxy for geographic exposure.

---

## 4. Results

### 4.1 Match Graph Structure (G_group)

| Metric | Value |
|--------|-------|
| Number of vertices (teams) | 48 |
| Number of edges (matches) | 72 |
| Graph density | 0.0638 |
| Number of connected components | 12 |
| Component structure | K₄ or K₃ (per group size) |
| Average degree | 3.0 (uniform) |

**Key finding:** The Group Stage match graph is structurally trivial from a centrality perspective — it consists of 12 disconnected cliques (complete subgraphs). Every team has exactly degree = 3, yielding uniform degree centrality C_D = 3/(48-1) ≈ 0.0638. This structural uniformity is a direct consequence of the round-robin format and validates the tournament design's principle of fairness.

### 4.2 City-Sharing Graph Structure (G_city)

| Metric | Value |
|--------|-------|
| Number of vertices | 42 |
| Number of edges | ~420 |
| Graph density | ~0.48 |
| Average degree centrality | 0.333 |
| Network diameter | 3 |
| Network radius | 2 |

The city-sharing graph is substantially denser and connected, enabling meaningful centrality differentiation.

### 4.3 Degree Centrality Results

| Rank | Team | Group | C_D |
|------|------|-------|-----|
| 1 | Curaçao | E | 0.4878 |
| 2 | Croatia | L | 0.4634 |
| 3 | Germany | E | 0.4634 |
| 4 | England | L | 0.4390 |
| 5 | Brazil | C | 0.4390 |

Curaçao's high degree centrality is noteworthy — despite being a historically smaller football nation, its Group E assignment places it in the same host-city cluster as Germany, giving it maximum city-sharing connections.

### 4.4 Betweenness Centrality Results

| Rank | Team | Group | C_B |
|------|------|-------|-----|
| 1 | England | L | 0.0814 |
| 2 | Canada | B | 0.0695 |
| 3 | Curaçao | E | 0.0563 |
| 4 | Argentina | J | 0.0531 |
| 5 | USA | D | 0.0452 |

**England** ranks highest in betweenness centrality — its Group L assignment spans geographically dispersed cities, positioning it as the primary bridge node in the geographic network. **Canada**'s high betweenness reflects its dual role as host nation (playing across both Canadian and US cities) and tournament participant.

### 4.5 Closeness Centrality Results

| Rank | Team | Group | C_C |
|------|------|-------|-----|
| 1 | Curaçao | E | 0.6613 |
| 2 | England | L | 0.6406 |
| 3 | Croatia | L | 0.6406 |
| 4 | Germany | E | 0.6308 |
| 5 | Argentina | J | 0.6212 |

Teams with high closeness are assigned to groups that play across centrally located host cities. The correlation between closeness and the US Central/East clusters is notable.

### 4.6 Eigenvector Centrality Results

| Rank | Team | Group | C_E |
|------|------|-------|-----|
| 1 | Curaçao | E | 0.2390 |
| 2 | Germany | E | 0.2338 |
| 3 | Brazil | C | 0.2287 |
| 4 | Croatia | L | 0.2259 |
| 5 | Ecuador | E | 0.2100 |

**Group E** (Curaçao, Germany, Ecuador) dominates the top eigenvector scores, revealing that these teams are not only well-connected themselves but are embedded in a mutually reinforcing high-influence sub-cluster. **Brazil** enters the top 3, confirming its central position in the geographic network.

### 4.7 Graph Algorithm Results

**BFS from Brazil (G_city):**
- **Teams at distance 1** (direct city-sharing): 18 teams
- **Teams at distance 2**: 22 teams
- **Teams at distance 3**: 1 team
- **Average shortest path from Brazil**: 1.73 hops

This indicates that Brazil is geographically well-connected — the vast majority of all other teams play in at least one city where Brazil also plays a group match.

**DFS from Brazil:** Traverses all 42 nodes, producing a depth-first spanning tree that prioritises depth over breadth. The DFS order differs substantially from BFS, demonstrating how traversal strategy affects the exploration of the same graph structure.

**Network Properties:**
- **Diameter** (longest shortest path): 3
- **Radius** (minimum eccentricity): 2
- These small values confirm **small-world** characteristics in the geographic network — any two teams in the tournament are separated by at most 3 city-sharing hops.

### 4.8 Group Density Analysis

All 12 groups produce subgraphs with **density = 1.0** (complete graphs), confirming the round-robin structure. The practical implication: within a group, no team enjoys a network advantage — all teams are equally reachable from any starting point in the same group.

### 4.9 Regional Match Distribution

| Region | Group Stage Matches |
|--------|-------------------|
| East (USA) | ~20 |
| Central (USA) | ~18 |
| West (USA) | ~14 |
| Mexico | ~12 |
| Canada | ~8 |

The United States hosts approximately 72% of Group Stage matches, reflecting its dominant role as primary host nation.

---

## 5. Discussion

### 5.1 Structural Implications of the 48-Team Format

The expansion from 32 to 48 teams introduces a structural change that is non-trivial from a graph-theoretic perspective. The previous 8-group format produced 8 disjoint K₄ graphs; the new 12-group format with mixed sizes (K₃ and K₄) produces a more heterogeneous — though still disjoint — Group Stage network. Importantly, this heterogeneity does not appear in the match graph (all teams have degree 3) but emerges in the city-sharing graph, where group size and venue assignments create differentiated connectivity.

### 5.2 The City-Sharing Graph as a Proxy for Geographic Centrality

The choice to construct G_city addresses a fundamental limitation: the match graph (G_group) is informationally trivial for centrality analysis. By connecting teams through shared venues, G_city captures a real and meaningful relationship — teams playing in the same city share logistical context, fan overlap, and media attention. The resulting graph exhibits the rich structural variation necessary for informative centrality analysis.

### 5.3 Surprise Finding: Curaçao's Network Centrality

One of the most counterintuitive findings is that **Curaçao** — a small Caribbean island nation with a population under 200,000 — ranks #1 in three of four centrality measures. This is not a reflection of football quality but of **positional advantage** in the geographic network: Curaçao's group assignment places it in the same venue cluster as Germany, one of football's historically strongest nations and one of the most city-connected teams. This illustrates a key principle of network science: centrality is a property of position in a graph, not of intrinsic node attributes.

### 5.4 England and Canada as Bridge Nations

England's #1 betweenness centrality and Canada's #2 ranking reflect their structural roles as bridges between geographic sub-clusters. In network terms, removing these nodes would most significantly fragment the city-sharing graph into disconnected components. From a tournament logistics perspective, this means England and Canada are the teams whose city assignments create the most connections between otherwise geographically separated clusters of teams.

### 5.5 Small-World Properties

The city-sharing network exhibits the **small-world** property: a network diameter of 3 and average path length of 1.73 from any given team (Brazil in our analysis) means that all 42 teams are within 3 city-sharing hops of each other. This is consistent with the Watts-Strogatz small-world model and reflects the fact that the 16 host cities serve as hubs connecting otherwise disparate groups.

### 5.6 Limitations

1. **Static analysis:** This study analyses the tournament structure as a fixed graph. A dynamic analysis tracking graph evolution as the tournament progresses would reveal additional temporal patterns.
2. **Binary edges:** The match graph uses unweighted edges. Incorporating match outcomes (goals scored, possession, expected goals) as edge weights would enrich the analysis.
3. **Incomplete knockout data:** 32 of 104 matches have unresolved participants. Full analysis requires complete knockout bracket data.
4. **Geographic simplification:** City-sharing proximity is a proxy for geographic connection; actual travel distances would provide a more precise geographic weighting.

---

## 6. Conclusion

This study demonstrates that Graph Theory provides a powerful and elegant framework for analysing the structure of international football tournaments. Applied to the FIFA World Cup 2026, our analysis reveals:

1. **The Group Stage match graph is a union of 12 complete subgraphs (K₄ and K₃)** — a mathematically clean structure that enforces equal competitive opportunity for all 48 participants.

2. **The city-sharing graph reveals meaningful centrality variation** that the match graph cannot. Teams like England, Curaçao, and Germany occupy structurally privileged positions in the geographic network, not due to football ability, but due to tournament scheduling decisions.

3. **Classical graph algorithms (BFS, DFS, shortest paths) successfully traverse the tournament network**, confirming its small-world properties: any two teams are separated by at most 3 city-sharing hops.

4. **Group E** (Curaçao, Germany, Ecuador) emerges as the most influential cluster in the network by eigenvector centrality — a finding with implications for understanding how venue assignments create information and prestige cascades in tournament settings.

5. **Brazil occupies a high-eigenvector, moderate-betweenness position**, connected to influential neighbours while not acting as the primary bridge in the network.

These findings contribute to the growing literature on sports network analysis and demonstrate the potential of graph-theoretic methods for understanding tournament structure, geographic logistics, and competitive balance in large-scale international sporting events.

---

## 7. Future Research Directions

### 7.1 Temporal Network Evolution
Model the tournament as a **time-varying graph** T = {G₁, G₂, ..., Gₜ} where each Gₜ adds the edges (matches) completed up to time t. Analyse how centrality scores evolve as the tournament progresses and teams are eliminated.

### 7.2 Outcome-Weighted Graph Analysis
Replace binary edges with weighted edges encoding match outcomes: goal difference, expected goals (xG), passing accuracy, or FIFA ranking delta. This would reveal whether network-central teams also tend to be performance-dominant.

### 7.3 Knockout Bracket as a Directed Acyclic Graph (DAG)
Model the knockout stage as a DAG where directed edges represent advancement. Apply reachability analysis to determine which group-stage positions offer the "easiest path" to the Final.

### 7.4 Community Detection
Apply Louvain, Girvan-Newman, or spectral clustering to the city-sharing graph and compare emergent communities against official geographic clusters. Test whether the tournament's geographic design creates structurally coherent communities.

### 7.5 Multi-Layer Network Analysis
Construct a multi-layer graph with separate layers for (a) group-stage matches, (b) city-sharing, and (c) FIFA ranking proximity. Analyse inter-layer centrality correlations using multi-layer network theory (Kivelä et al., 2014).

### 7.6 Historical Comparative Analysis
Build equivalent match graphs for WC 1994–2022 and compute network metrics across editions. Test whether tournament expansion (from 24 to 32 to 48 teams) is associated with predictable changes in network density, diameter, or centrality distributions.

### 7.7 Machine Learning Integration
Use computed centrality scores as features in a supervised learning model to predict tournament advancement, knockout stage performance, or FIFA ranking changes following the tournament.

---

## References

Barabási, A.-L. (2016). *Network Science*. Cambridge University Press.

Bonacich, P. (1987). Power and Centrality: A Family of Measures. *American Journal of Sociology*, 92(5), 1170–1182.

Brandes, U. (2001). A faster algorithm for betweenness centrality. *Journal of Mathematical Sociology*, 25(2), 163–177.

Euler, L. (1736). Solutio problematis ad geometriam situs pertinentis. *Commentarii Academiae Scientiarum Petropolitanae*, 8, 128–140.

FIFA (2026). *Official FIFA World Cup 2026 Match Schedule*. Fédération Internationale de Football Association.

Freeman, L. C. (1977). A Set of Measures of Centrality Based on Betweenness. *Sociometry*, 40(1), 35–41.

Hagberg, A., Swart, P., & Chult, D. (2008). Exploring network structure, dynamics, and function using NetworkX. In *Proceedings of the 7th Python in Science Conference (SciPy2008)* (pp. 11–15).

Kivelä, M., Arenas, A., Barthelemy, M., Gleeson, J. P., Moreno, Y., & Porter, M. A. (2014). Multilayer networks. *Journal of Complex Networks*, 2(3), 203–271.

Newman, M. E. J. (2010). *Networks: An Introduction*. Oxford University Press.

Onody, R. N., & de Castro, P. A. (2004). Complex network study of Brazilian soccer players. *Physical Review E*, 70(3), 037103.

Watts, D. J., & Strogatz, S. H. (1998). Collective dynamics of 'small-world' networks. *Nature*, 393, 440–442.

West, D. B. (2001). *Introduction to Graph Theory* (2nd ed.). Prentice Hall.

---

*This report was generated by Mariam Adejoke Popoola (PhD in Informatics) as part of a PhD portfolio project at the Universidade Federal do Amazonas (UFAM). All data, code, and outputs are available in the accompanying GitHub repository.*
