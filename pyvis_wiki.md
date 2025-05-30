# Pyvis Guide

## Installation

```bash
pip install pyvis
# Optional dependencies for data manipulation
pip install networkx pandas numpy
```

## Overview

**Specialty:** Interactive network visualizations for web browsers

**Best Use Cases:**
- Interactive exploration of network structures
- Web-based network dashboards
- Educational network demonstrations
- Small to medium-sized network visualizations (< 1000 nodes)
- Presentations requiring interactive network exploration

## ASCII Example

Below is an ASCII representation of what we'll create - an interactive network with physics simulation:

```
     [A]----[B]----[C]
      |      |      |
      |      |      |
     [D]----[E]----[F]
             |
             |
     [G]----[H]----[I]

Interactive Features:
- Drag nodes to rearrange
- Zoom and pan
- Hover for node information
- Physics-based layout
- Click to highlight connections
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
from pyvis.network import Network
import networkx as nx
import pandas as pd
import random
```

### 2. Prepare Your Data

```python
# Create sample data for a technology stack network
nodes_data = [
    {'id': 'Frontend', 'label': 'Frontend', 'group': 'presentation', 'size': 30},
    {'id': 'React', 'label': 'React', 'group': 'framework', 'size': 25},
    {'id': 'Vue', 'label': 'Vue.js', 'group': 'framework', 'size': 20},
    {'id': 'Angular', 'label': 'Angular', 'group': 'framework', 'size': 22},
    {'id': 'Backend', 'label': 'Backend', 'group': 'logic', 'size': 30},
    {'id': 'Django', 'label': 'Django', 'group': 'framework', 'size': 25},
    {'id': 'Flask', 'label': 'Flask', 'group': 'framework', 'size': 20},
    {'id': 'FastAPI', 'label': 'FastAPI', 'group': 'framework', 'size': 18},
    {'id': 'Database', 'label': 'Database', 'group': 'storage', 'size': 30},
    {'id': 'PostgreSQL', 'label': 'PostgreSQL', 'group': 'database', 'size': 25},
    {'id': 'MongoDB', 'label': 'MongoDB', 'group': 'database', 'size': 22},
    {'id': 'Redis', 'label': 'Redis', 'group': 'cache', 'size': 15}
]

edges_data = [
    {'from': 'Frontend', 'to': 'React', 'weight': 3},
    {'from': 'Frontend', 'to': 'Vue', 'weight': 2},
    {'from': 'Frontend', 'to': 'Angular', 'weight': 2},
    {'from': 'Backend', 'to': 'Django', 'weight': 3},
    {'from': 'Backend', 'to': 'Flask', 'weight': 2},
    {'from': 'Backend', 'to': 'FastAPI', 'weight': 2},
    {'from': 'Database', 'to': 'PostgreSQL', 'weight': 3},
    {'from': 'Database', 'to': 'MongoDB', 'weight': 2},
    {'from': 'Database', 'to': 'Redis', 'weight': 1},
    {'from': 'React', 'to': 'Backend', 'weight': 2},
    {'from': 'Vue', 'to': 'Backend', 'weight': 2},
    {'from': 'Angular', 'to': 'Backend', 'weight': 2},
    {'from': 'Django', 'to': 'Database', 'weight': 3},
    {'from': 'Flask', 'to': 'Database', 'weight': 2},
    {'from': 'FastAPI', 'to': 'Database', 'weight': 2}
]
```

### 3. Create the Basic Visualization

```python
# Initialize the network
net = Network(height="600px", width="100%", bgcolor="#222222", font_color="white")

# Add nodes
for node in nodes_data:
    net.add_node(node['id'], 
                 label=node['label'],
                 group=node['group'],
                 size=node['size'],
                 title=f"Technology: {node['label']}")

# Add edges
for edge in edges_data:
    net.add_edge(edge['from'], 
                 edge['to'], 
                 width=edge['weight'],
                 title=f"Connection strength: {edge['weight']}")
```

### 4. Customize the Appearance

```python
# Configure physics simulation
net.set_options("""
var options = {
  "physics": {
    "enabled": true,
    "stabilization": {"iterations": 100},
    "barnesHut": {
      "gravitationalConstant": -8000,
      "centralGravity": 0.3,
      "springLength": 95,
      "springConstant": 0.04,
      "damping": 0.09
    }
  },
  "nodes": {
    "borderWidth": 2,
    "borderWidthSelected": 4,
    "font": {"size": 16, "color": "white"},
    "shadow": true
  },
  "edges": {
    "color": {"inherit": true},
    "smooth": false,
    "shadow": true
  },
  "groups": {
    "presentation": {"color": {"background": "#FF6B6B", "border": "#FF5252"}},
    "framework": {"color": {"background": "#4ECDC4", "border": "#26A69A"}},
    "logic": {"color": {"background": "#45B7D1", "border": "#2196F3"}},
    "storage": {"color": {"background": "#96CEB4", "border": "#4CAF50"}},
    "database": {"color": {"background": "#FFEAA7", "border": "#FFC107"}},
    "cache": {"color": {"background": "#DDA0DD", "border": "#9C27B0"}}
  }
}
""")
```

### 5. Add Interactivity

```python
# Add neighbor highlighting
net.set_options("""
var options = {
  "interaction": {
    "hover": true,
    "hoverConnectedEdges": true,
    "selectConnectedEdges": true
  },
  "manipulation": {
    "enabled": false
  }
}
""")

# Add custom JavaScript for enhanced interactivity
net.set_options("""
var options = {
  "configure": {
    "enabled": false
  }
}
""")
```

### 6. Export or Display

```python
# Generate and save the HTML file
net.show("tech_stack_network.html")

# Alternative: Save with custom name and open in browser
net.write_html("interactive_tech_network.html")

# For Jupyter notebooks, display inline
# net.show("network.html", notebook=True)

print("Interactive network saved as 'tech_stack_network.html'")
print("Open the file in a web browser to explore the interactive network!")
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Performance degradation with large networks (>1000 nodes)
  - Solution: Filter data, use clustering, or switch to specialized tools for large networks
- **Pitfall 2**: Overlapping nodes in dense networks
  - Solution: Adjust physics parameters, increase repulsion, or use hierarchical layout
- **Pitfall 3**: Poor mobile responsiveness
  - Solution: Set responsive width/height and test on mobile devices

## Advanced Techniques

- Creating hierarchical networks with custom positioning
- Implementing custom node shapes and images
- Adding time-based network evolution animations
- Integrating with web frameworks (Flask, Django) for dynamic networks
- Using clustering algorithms to group nodes visually
- Adding custom control panels for network filtering

## Resources

- [Pyvis Documentation](https://pyvis.readthedocs.io/en/latest/)
- [Pyvis Examples](https://pyvis.readthedocs.io/en/latest/tutorial.html)
- [Vis.js Network Documentation](https://visjs.github.io/vis-network/docs/network/)
- [Interactive Network Visualization Tutorial](https://towardsdatascience.com/python-interactive-network-visualization-using-networkx-plotly-and-dash-e44749161ed7)
