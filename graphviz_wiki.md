# Graphviz (python-graphviz) Guide

## Installation

```bash
# Install the Python package
pip install graphviz

# Install Graphviz system software (required)
# On Ubuntu/Debian:
sudo apt-get install graphviz
# On macOS:
brew install graphviz
# On Windows: Download from https://graphviz.org/download/
```

## Overview

**Specialty:** Automatic graph layout for hierarchical and directed graphs

**Best Use Cases:**
- Organizational charts and hierarchical structures
- Software dependency diagrams
- Decision trees and flowcharts
- Database schema visualization
- Abstract syntax trees
- Documentation diagrams with automatic layout

## ASCII Example

Below is an ASCII representation of what we'll create - a software architecture dependency graph:

```
    ┌─────────────┐
    │   Frontend  │
    └──────┬──────┘
           │
    ┌──────▼──────┐
    │     API     │
    └──────┬──────┘
           │
    ┌──────▼──────┐     ┌─────────────┐
    │  Business   │────▶│   Cache     │
    │    Logic    │     │  (Redis)    │
    └──────┬──────┘     └─────────────┘
           │
    ┌──────▼──────┐
    │  Database   │
    │ (PostgreSQL)│
    └─────────────┘
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
from graphviz import Digraph, Graph
import os
```

### 2. Prepare Your Data

```python
# Define a software architecture with components and dependencies
components = {
    'frontend': {'label': 'Frontend\n(React)', 'color': 'lightblue'},
    'api': {'label': 'API Gateway\n(FastAPI)', 'color': 'lightgreen'},
    'auth': {'label': 'Authentication\nService', 'color': 'lightyellow'},
    'business': {'label': 'Business Logic\nService', 'color': 'lightcoral'},
    'cache': {'label': 'Cache\n(Redis)', 'color': 'lightgray'},
    'database': {'label': 'Database\n(PostgreSQL)', 'color': 'lightpink'},
    'queue': {'label': 'Message Queue\n(RabbitMQ)', 'color': 'lightsteelblue'}
}

dependencies = [
    ('frontend', 'api'),
    ('api', 'auth'),
    ('api', 'business'),
    ('business', 'cache'),
    ('business', 'database'),
    ('business', 'queue'),
    ('auth', 'database')
]
```

### 3. Create the Basic Visualization

```python
# Create a directed graph
dot = Digraph(comment='Software Architecture')
dot.attr(rankdir='TB', size='8,10')
dot.attr('node', shape='box', style='rounded,filled')

# Add nodes
for node_id, attrs in components.items():
    dot.node(node_id, attrs['label'], fillcolor=attrs['color'])

# Add edges
for source, target in dependencies:
    dot.edge(source, target)

# Render the basic graph
dot.render('basic_architecture', format='png', cleanup=True)
```

### 4. Customize the Appearance

```python
# Create an enhanced version with better styling
enhanced_dot = Digraph(comment='Enhanced Software Architecture')

# Set graph attributes
enhanced_dot.attr(rankdir='TB', 
                  bgcolor='white',
                  fontname='Arial',
                  fontsize='12',
                  label='Software Architecture Diagram',
                  labelloc='t')

# Set default node attributes
enhanced_dot.attr('node', 
                  shape='box',
                  style='rounded,filled',
                  fontname='Arial',
                  fontsize='10',
                  margin='0.2')

# Set default edge attributes
enhanced_dot.attr('edge',
                  fontname='Arial',
                  fontsize='8',
                  color='gray40',
                  arrowsize='0.8')

# Add nodes with enhanced styling
layer_colors = {
    'frontend': '#E3F2FD',
    'api': '#E8F5E8', 
    'auth': '#FFF3E0',
    'business': '#FCE4EC',
    'cache': '#F3E5F5',
    'database': '#E0F2F1',
    'queue': '#E1F5FE'
}

for node_id, attrs in components.items():
    enhanced_dot.node(node_id, 
                      attrs['label'], 
                      fillcolor=layer_colors[node_id],
                      color='gray60')

# Add edges with labels
edge_labels = {
    ('frontend', 'api'): 'HTTP/REST',
    ('api', 'auth'): 'JWT validation',
    ('api', 'business'): 'Service calls',
    ('business', 'cache'): 'Cache ops',
    ('business', 'database'): 'CRUD ops',
    ('business', 'queue'): 'Async tasks',
    ('auth', 'database'): 'User data'
}

for source, target in dependencies:
    label = edge_labels.get((source, target), '')
    enhanced_dot.edge(source, target, label=label)
```

### 5. Create Multiple Layout Variations

```python
# Create different layout orientations
layouts = {
    'top_down': {'rankdir': 'TB', 'filename': 'architecture_vertical'},
    'left_right': {'rankdir': 'LR', 'filename': 'architecture_horizontal'},
    'bottom_up': {'rankdir': 'BT', 'filename': 'architecture_bottom_up'}
}

for layout_name, config in layouts.items():
    layout_dot = Digraph(comment=f'Architecture - {layout_name}')
    layout_dot.attr(rankdir=config['rankdir'])
    layout_dot.attr('node', shape='box', style='rounded,filled')
    
    # Add all nodes and edges
    for node_id, attrs in components.items():
        layout_dot.node(node_id, attrs['label'], fillcolor=attrs['color'])
    
    for source, target in dependencies:
        layout_dot.edge(source, target)
    
    layout_dot.render(config['filename'], format='png', cleanup=True)
```

### 6. Export or Display

```python
# Render in multiple formats
formats = ['png', 'svg', 'pdf']

for fmt in formats:
    enhanced_dot.render(f'software_architecture.{fmt}', 
                       format=fmt, 
                       cleanup=True)

# Save the source code
with open('architecture.dot', 'w') as f:
    f.write(enhanced_dot.source)

# Display the source code
print("Generated Graphviz DOT source:")
print(enhanced_dot.source)

print(f"Architecture diagrams saved in formats: {', '.join(formats)}")
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Graphviz software not installed or not in PATH
  - Solution: Install system Graphviz package and ensure it's in PATH
- **Pitfall 2**: Overlapping nodes in complex graphs
  - Solution: Use subgraphs, adjust node spacing, or use different layout engines (dot, neato, fdp)
- **Pitfall 3**: Poor text rendering in nodes
  - Solution: Use proper escaping for special characters and consider HTML-like labels

## Advanced Techniques

- Using subgraphs and clusters for logical grouping
- Creating custom node shapes with HTML-like labels
- Implementing conditional styling based on data attributes
- Generating graphs from code analysis tools
- Creating animated graph evolution using multiple renders
- Integrating with documentation generation pipelines

## Resources

- [Graphviz Documentation](https://graphviz.org/documentation/)
- [Python Graphviz Package Docs](https://graphviz.readthedocs.io/en/stable/)
- [DOT Language Reference](https://graphviz.org/doc/info/lang.html)
- [Graphviz Gallery](https://graphviz.org/gallery/)
