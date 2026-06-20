# ⚽ # ⚽ FIFA World Cup 2026 – Network Analysis Using Graph Theory

> **Author:** Mariam Adejoke Popoola — PhD Student in Informatics, Universidade Federal do Amazonas (UFAM), Manaus, Brazil
> **LinkedIn:** [linkedin.com/in/mariam-popoola](https://linkedin.com/in/mariam-popoola)
> **GitHub:** [github.com/MariamAPopoola](https://github.com/MariamAPopoola)
> **Course:** Graph Theory & Network Analysis
> **Language:** Python 3.10+
> **Status:** Complete ✅
---

## 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [Dataset Description](#-dataset-description)
3. [Methodology](#-methodology)
4. [Project Structure](#-project-structure)
5. [Results & Findings](#-results--findings)
6. [Visualizations](#-visualizations)
7. [Technologies Used](#-technologies-used)
8. [How to Run](#-how-to-run)
9. [Future Work](#-future-work)
10. [References](#-references)

---

## 🌐 Project Overview

This project applies **Graph Theory** and **Network Analysis** techniques to the official FIFA World Cup 2026 fixture data. The 2026 edition is historically significant — it is the first World Cup to feature **48 teams**, hosted across **16 cities** in the **USA, Canada, and Mexico**, making it the most geographically expansive tournament ever held.

### Core Research Questions

| # | Question |
|---|----------|
| Q1 | Which teams are most **topologically central** in the tournament network? |
| Q2 | Which teams serve as **geographic bridges** between different tournament regions? |
| Q3 | What structural properties emerge from treating teams as graph vertices and matches as edges? |
| Q4 | How do classical graph algorithms (BFS, DFS, Shortest Paths) traverse the tournament network? |

### Graph Formulation

The tournament is modelled as two complementary graphs:

| Graph | Vertices | Edges | Purpose |
|-------|----------|-------|---------|
| **G_group** | 48 teams | 72 group-stage matches | Tournament match structure |
| **G_city** | 42 real teams | City-sharing connections | Geographic proximity analysis |

---

## 📁 Dataset Description

Four official datasets were used, totalling **219 rows** across **23 columns**.

| File | Rows | Columns | Role in Graph |
|------|------|---------|---------------|
| `matches.csv` | 104 | 8 | **Edge table** — each match is an edge |
| `teams.csv` | 48 | 5 | **Vertex attribute table** |
| `host_cities.csv` | 16 | 6 | **Geographic context** |
| `tournament_stages.csv` | 7 | 3 | **Edge classification** |

### Column Details

**matches.csv**
- `id` — Unique match identifier
- `match_number` — Sequential match number (1–104)
- `home_team_id` / `away_team_id` — FK to teams table (null for unresolved knockout fixtures)
- `city_id` — FK to host_cities
- `stage_id` — FK to tournament_stages
- `kickoff_at` — ISO 8601 timestamp with timezone offset
- `match_label` — Human-readable label (e.g. "Group A", "Final")

**teams.csv**
- `id` — Unique team identifier
- `team_name` — Full country name
- `fifa_code` — 3-letter FIFA code (e.g. BRA, ARG)
- `group_letter` — Group assignment (A–L)
- `is_placeholder` — True for unqualified playoff teams

**host_cities.csv**
- `city_name`, `country` — Location
- `venue_name` — Stadium name
- `region_cluster` — East / Central / West / Mexico / Canada

**tournament_stages.csv**
- 7 stages: Group Stage → Round of 32 → R16 → Quarterfinals → Semifinals → 3rd Place Playoff → Final

### Data Quality Notes
- **32 missing team IDs** in `matches.csv` — expected, corresponding to knockout-stage matches where participants are determined by group results
- **0 duplicate records** across all four datasets
- **6 placeholder teams** (unresolved UEFA / CONCACAF playoff positions) excluded from graph analysis

---

## 🔬 Methodology

The analysis follows a nine-phase pipeline:

```
RAW DATA → EDA → PREPROCESSING → GRAPH CONSTRUCTION → VISUALIZATION
    → CENTRALITY ANALYSIS → GRAPH ALGORITHMS → ADVANCED ANALYSIS → REPORTING
```

### Phase 1 — Data Understanding
Pandas-based EDA covering dimensions, dtypes, null counts, duplicates, and descriptive statistics.

### Phase 2 — Data Preprocessing
- Merged `matches.csv` ↔ `teams.csv` ↔ `stages` ↔ `cities`
- Replaced numeric IDs with country names
- Parsed timestamps to UTC-aware `datetime64`
- Isolated 72 confirmed Group Stage matches (known teams on both sides)

### Phase 3 — Graph Construction
Two graphs were built using **NetworkX**:

- **G_group (Match Graph):** Undirected graph G = (V, E) where V = teams, E = scheduled matches. The Group Stage produces 12 disjoint complete graphs K₄ (one per group), each with C(4,2) = 6 edges.
- **G_city (City-Sharing Graph):** Two teams are connected if they play at least one match in the **same host city**. This cross-group graph enables meaningful centrality variation.

### Phase 4 — Visualization
Seven professional figures generated with Matplotlib/NetworkX using a consistent 12-colour palette (one per group):
- Full tournament network
- Per-group subgraph grid (3×4)
- Degree distribution histogram
- Centrality comparison bar charts
- Centrality heatmap
- BFS spanning tree
- City-sharing network

### Phase 5 — Centrality Analysis
Four standard centrality measures computed on **G_city**:

| Measure | Formula | Interpretation |
|---------|---------|----------------|
| **Degree** | kᵥ / (n-1) | Number of city-sharing connections |
| **Betweenness** | Σ σ(s,t\|v) / σ(s,t) | Bridge role in geographic network |
| **Closeness** | (n-1) / Σd(v,u) | Proximity to all other teams |
| **Eigenvector** | Ax = λx | Influence through connected neighbours |

### Phase 6 — Graph Algorithms
Three classical algorithms implemented from scratch:

- **BFS** — Level-by-level traversal (FIFO queue), starting from Brazil
- **DFS** — Depth-first traversal (explicit stack), starting from Brazil  
- **Shortest Path** — `nx.single_source_shortest_path_length` + diameter/radius

### Phase 7 — Advanced Analysis
- Group density comparison (K₄ theory verification)
- Composite centrality score (min-max normalised average of four measures)
- Regional cluster match distribution
- City-reach analysis (teams visiting most distinct host cities)

---

## 📂 Project Structure

```
fifa_wc2026_network/
├── analysis.py                  ← Complete analysis (Phases 1–9)
├── requirements.txt
├── README.md
├── data/
│   ├── matches.csv
│   ├── teams.csv
│   ├── host_cities.csv
│   └── tournament_stages.csv
└── outputs/
    ├── figures/
    │   ├── fig1_full_network.png         ← Full Group Stage network
    │   ├── fig2_group_networks.png       ← 12 group subgraphs (3×4 grid)
    │   ├── fig3_degree_distribution.png  ← Degree histogram
    │   ├── fig4_centrality_comparison.png← 4-panel centrality bar charts
    │   ├── fig5_centrality_heatmap.png   ← All-teams centrality heatmap
    │   ├── fig6_bfs_tree.png             ← BFS tree rooted at Brazil
    │   └── fig7_city_sharing_graph.png   ← City-sharing network
    └── reports/
        └── centrality_summary.csv        ← Per-team metrics table
```

---

## 📊 Results & Findings

### Graph Structure (G_group — Match Graph)

| Metric | Value |
|--------|-------|
| Nodes (teams) | 48 |
| Edges (matches) | 72 |
| Graph Density | 0.0638 |
| Connected Components | 12 (one per group) |
| Component structure | K₄ (complete graph on 4 nodes) |

> **Key insight:** The Group Stage enforces a mathematically perfect structure — 12 disjoint K₄ graphs, where every pair of teams in the same group plays exactly once, producing uniform degree centrality (d = 3 for all teams).

---

### Centrality Rankings (G_city — City-Sharing Graph)

#### Degree Centrality — Most Connected Teams
| Rank | Team | Group | Score |
|------|------|-------|-------|
| 1 | Curaçao | E | 0.4878 |
| 2 | Croatia | L | 0.4634 |
| 3 | Germany | E | 0.4634 |
| 4 | England | L | 0.4390 |
| 5 | Brazil | C | 0.4390 |

#### Betweenness Centrality — Bridge Teams
| Rank | Team | Group | Score |
|------|------|-------|-------|
| 1 | England | L | 0.0814 |
| 2 | Canada | B | 0.0695 |
| 3 | Curaçao | E | 0.0563 |
| 4 | Argentina | J | 0.0531 |
| 5 | USA | D | 0.0452 |

> **England** ranks #1 in betweenness — its group plays across geographically diverse cities, making it the key bridge node in the city-sharing graph.  
> **Canada** ranks #2, reflecting its host-nation status (games played across both Canadian and US cities).

#### Closeness Centrality — Geographically Central Teams
| Rank | Team | Group | Score |
|------|------|-------|-------|
| 1 | Curaçao | E | 0.6613 |
| 2 | England | L | 0.6406 |
| 3 | Croatia | L | 0.6406 |
| 4 | Germany | E | 0.6308 |
| 5 | Argentina | J | 0.6212 |

#### Eigenvector Centrality — Most Influential Teams
| Rank | Team | Group | Score |
|------|------|-------|-------|
| 1 | Curaçao | E | 0.2390 |
| 2 | Germany | E | 0.2338 |
| 3 | Brazil | C | 0.2287 |
| 4 | Croatia | L | 0.2259 |
| 5 | Ecuador | E | 0.2100 |

> **Group E** (Curaçao, Germany, Ecuador) dominates eigenvector centrality — these teams are not only well-connected but connected to other well-connected teams.

---

### Graph Algorithm Results

**BFS from Brazil:**
- Average shortest path to all other teams: **1.73 hops**
- Brazil reaches **41 teams** in the city-sharing graph
- Nearest teams (distance = 1): England, Curaçao, Croatia, Germany, and 17 others sharing the same host cities

**Network Diameter:** The city-sharing graph has a small diameter, confirming the **small-world** property of shared geographic infrastructure.

---

## 🖼️ Visualizations

| Figure | Description |
|--------|-------------|
| `fig1_full_network.png` | Full 48-team Group Stage network coloured by group |
| `fig2_group_networks.png` | 12 group subgraphs in a 3×4 grid (circular layout) |
| `fig3_degree_distribution.png` | Uniform degree-3 histogram confirming K₄ structure |
| `fig4_centrality_comparison.png` | 4-panel horizontal bar charts (top 20 per metric) |
| `fig5_centrality_heatmap.png` | All-team × 4-metric normalised heatmap |
| `fig6_bfs_tree.png` | BFS spanning tree rooted at Brazil |
| `fig7_city_sharing_graph.png` | City-sharing graph (edge weight = #shared cities) |

---

## 🛠️ Technologies Used

| Library | Version | Purpose |
|---------|---------|---------|
| Python | 3.10+ | Core language |
| pandas | ≥2.0 | Data loading, merging, EDA |
| numpy | ≥1.24 | Numerical operations |
| networkx | ≥3.1 | Graph construction, centrality, algorithms |
| matplotlib | ≥3.7 | All visualizations |
| seaborn | ≥0.12 | Statistical plot styling |
| scikit-learn | ≥1.2 | Min-max normalisation for composite scores |
| scipy | ≥1.10 | Supporting numerical routines |

---

## ▶️ How to Run

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/fifa-wc2026-network-analysis.git
cd fifa-wc2026-network-analysis

# 2. Create a virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate          # Linux / macOS
venv\Scripts\activate             # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the full analysis
python3 analysis.py

# 5. View outputs
ls outputs/figures/               # Seven PNG visualizations
ls outputs/reports/               # CSV centrality summary
```

> **Note:** All figures are saved automatically to `outputs/figures/`. No display window is required (Matplotlib uses the `Agg` non-interactive backend).

---

## 🔭 Future Work

1. **Dynamic / Temporal Graph Analysis** — Model the tournament as a time-evolving graph where edges are added as matches are played, tracking how centrality scores shift after each match day.

2. **Knockout Stage Integration** — Once the bracket is resolved, extend G_group with knockout edges and re-compute centralities on the full 104-match graph.

3. **Weighted Graph Analysis** — Assign edge weights based on goal difference, match importance (group vs. final), or FIFA ranking delta.

4. **Community Detection** — Apply Louvain or Girvan-Newman community detection to G_city and compare discovered communities against official geographic regions.

5. **Predictive Modelling** — Use centrality scores as input features to a machine learning model predicting tournament advancement.

6. **3D Interactive Visualization** — Use Plotly or Pyvis to generate interactive HTML network graphs for web embedding.

7. **Historical Comparison** — Build the same graph for WC2022 (Qatar), WC2018 (Russia), and WC2014 (Brazil) and compare structural properties across editions.

8. **Travel Network Analysis** — Model team travel routes as a directed weighted graph using city-to-city distances, minimising total travel burden.

---

## 📚 References

- Bonacich, P. (1987). Power and Centrality: A Family of Measures. *American Journal of Sociology*, 92(5), 1170–1182.
- Brandes, U. (2001). A faster algorithm for betweenness centrality. *Journal of Mathematical Sociology*, 25(2), 163–177.
- Freeman, L. C. (1977). A Set of Measures of Centrality Based on Betweenness. *Sociometry*, 40(1), 35–41.
- Hagberg, A., Swart, P., & Chult, D. (2008). Exploring network structure, dynamics, and function using NetworkX. *Proceedings of SciPy 2008*.
- Newman, M. E. J. (2010). *Networks: An Introduction*. Oxford University Press.
- West, D. B. (2001). *Introduction to Graph Theory* (2nd ed.). Prentice Hall.
- FIFA (2026). Official FIFA World Cup 2026 Match Schedule. https://www.fifa.com/fifaplus/en/tournaments/mens/worldcup/canadamexicousa2026

---

<div align="center">

**Made with ❤️ and Graph Theory in Manaus, Amazonas, Brazil 🇧🇷**

*Universidade Federal do Amazonas — Instituto de Computação*

</div>
