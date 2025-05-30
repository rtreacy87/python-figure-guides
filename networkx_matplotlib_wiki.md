# NetworkX + Matplotlib Guide

## Installation

```bash
pip install networkx matplotlib
# Optional dependencies for additional features
pip install numpy pandas scipy
```

## Overview

**Specialty:** Graph theory and network analysis visualization

**Best Use Cases:**
- Social network analysis and visualization
- Dependency graphs and software architecture diagrams
- Network topology mapping
- Mathematical graph theory applications
- Large-scale network analysis with algorithmic processing

## ASCII Example

Below is an ASCII representation of what we'll create - a social network with community detection:

```
    A ---- B ---- C
    |      |      |
    |      |      |
    D ---- E ---- F
           |
           |
    G ---- H ---- I
    |      |      |
    |      |      |
    J ---- K ---- L

Community 1: {A, B, D, E}
Community 2: {C, F, I, L}
Community 3: {G, H, J, K}
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
from matplotlib.colors import ListedColormap
```

### 2. Prepare Your Data

```python
# Create a sample social network
G = nx.Graph()

# Add nodes with attributes
nodes = [
    ('Alice', {'age': 25, 'department': 'Engineering'}),
    ('Bob', {'age': 30, 'department': 'Engineering'}),
    ('Carol', {'age': 28, 'department': 'Marketing'}),
    ('David', {'age': 35, 'department': 'Engineering'}),
    ('Eve', {'age': 27, 'department': 'Marketing'}),
    ('Frank', {'age': 32, 'department': 'Sales'}),
    ('Grace', {'age': 29, 'department': 'Sales'}),
    ('Henry', {'age': 26, 'department': 'Marketing'})
]

G.add_nodes_from(nodes)

# Add edges (relationships)
edges = [
    ('Alice', 'Bob'), ('Alice', 'David'), ('Bob', 'Carol'),
    ('Bob', 'David'), ('Carol', 'Eve'), ('David', 'Eve'),
    ('Eve', 'Frank'), ('Frank', 'Grace'), ('Grace', 'Henry'),
    ('Carol', 'Henry')
]

G.add_edges_from(edges)
```

### 3. Create the Basic Visualization

```python
# Create figure with subplots
fig, axes = plt.subplots(2, 2, figsize=(15, 12))

# Basic network layout
pos = nx.spring_layout(G, seed=42)

# Plot 1: Basic network
ax1 = axes[0, 0]
nx.draw(G, pos, ax=ax1, with_labels=True, node_color='lightblue', 
        node_size=1000, font_size=10, font_weight='bold')
ax1.set_title('Basic Network Visualization')

# Plot 2: Colored by department
ax2 = axes[0, 1]
dept_colors = {'Engineering': 'red', 'Marketing': 'blue', 'Sales': 'green'}
node_colors = [dept_colors[G.nodes[node]['department']] for node in G.nodes()]
nx.draw(G, pos, ax=ax2, with_labels=True, node_color=node_colors,
        node_size=1000, font_size=10, font_weight='bold')
ax2.set_title('Network Colored by Department')

# Plot 3: Node size by degree centrality
ax3 = axes[1, 0]
centrality = nx.degree_centrality(G)
node_sizes = [centrality[node] * 3000 for node in G.nodes()]
nx.draw(G, pos, ax=ax3, with_labels=True, node_color='orange',
        node_size=node_sizes, font_size=10, font_weight='bold')
ax3.set_title('Node Size by Degree Centrality')

# Plot 4: Community detection
ax4 = axes[1, 1]
communities = nx.community.greedy_modularity_communities(G)
community_colors = ['red', 'blue', 'green', 'yellow', 'purple']
node_colors = []
for node in G.nodes():
    for i, community in enumerate(communities):
        if node in community:
            node_colors.append(community_colors[i % len(community_colors)])
            break

nx.draw(G, pos, ax=ax4, with_labels=True, node_color=node_colors,
        node_size=1000, font_size=10, font_weight='bold')
ax4.set_title('Community Detection')
```

### 4. Customize the Appearance

```python
# Apply consistent styling
plt.style.use('seaborn-v0_8-whitegrid')

# Adjust layout
plt.tight_layout()

# Add figure title
fig.suptitle('Social Network Analysis Dashboard', fontsize=16, y=0.98)

# Remove axes for cleaner look
for ax in axes.flat:
    ax.axis('off')
```

### 5. Add Network Analysis (Advanced Features)

```python
# Calculate network metrics
print("Network Analysis Metrics:")
print(f"Number of nodes: {G.number_of_nodes()}")
print(f"Number of edges: {G.number_of_edges()}")
print(f"Network density: {nx.density(G):.3f}")
print(f"Average clustering coefficient: {nx.average_clustering(G):.3f}")

# Find shortest paths
shortest_paths = dict(nx.all_pairs_shortest_path_length(G))
print(f"Average shortest path length: {nx.average_shortest_path_length(G):.3f}")

# Identify key nodes
betweenness = nx.betweenness_centrality(G)
most_central = max(betweenness, key=betweenness.get)
print(f"Most central node (betweenness): {most_central}")
```

### 6. Export or Display

```python
# Save the figure
plt.savefig('social_network_analysis.png', dpi=300, bbox_inches='tight')

# Save network data
nx.write_gexf(G, 'social_network.gexf')  # For Gephi
nx.write_graphml(G, 'social_network.graphml')  # For other tools

# Display the figure
plt.show()
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Overlapping node labels in dense networks
  - Solution: Use `nx.draw_networkx_labels()` with position adjustments or reduce font size
- **Pitfall 2**: Poor layout for specific network types
  - Solution: Choose appropriate layout algorithms (circular, hierarchical, spring, etc.)
- **Pitfall 3**: Slow performance with large networks
  - Solution: Use sampling, filtering, or specialized layouts like `nx.spring_layout(k=1/sqrt(n))`

## Advanced Techniques

- Implementing custom layout algorithms for domain-specific networks
- Using NetworkX with interactive tools like Plotly or Bokeh
- Applying machine learning algorithms for link prediction
- Creating animated network evolution over time
- Integrating with graph databases like Neo4j

## Resources

- [NetworkX Documentation](https://networkx.org/documentation/stable/)
- [NetworkX Tutorial](https://networkx.org/documentation/stable/tutorial.html)
- [Graph Analysis with Python](https://www.oreilly.com/library/view/complex-network-analysis/9781787123137/)
- [NetworkX Gallery](https://networkx.org/documentation/stable/auto_examples/index.html)
