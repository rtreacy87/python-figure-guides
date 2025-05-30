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

## Executive Summary

This code section defines the data structure for a software architecture diagram:

1. `components` dictionary defines each system component with display attributes (label and color)
2. `dependencies` list defines the connections between components (directional relationships)

This data preparation is essential because:
- It separates data from visualization logic
- It allows for reuse across multiple diagram variations
- It makes the architecture definition maintainable

The dictionary order is **not significant** for functionality since dictionaries are accessed by key. However, the current order follows a logical top-down flow of the architecture (frontend → API → services → infrastructure), which helps with readability and maintenance.


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

## Executive Summary

This code section creates a basic directed graph visualization using Graphviz:

1. Initializes a directed graph with top-to-bottom layout and specific dimensions
2. Configures node appearance with rounded boxes and fill colors
3. Adds all components as nodes with their labels and colors from the data structure
4. Creates directed edges between components based on the dependencies list
5. Renders the final diagram to a PNG file and cleans up temporary files

This visualization step is essential because it transforms the abstract data structure into a visual representation that clearly shows the relationships between system components.

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
Line-by-line explanation of the selected code:

```python
# Create a directed graph
dot = Digraph(comment='Software Architecture')
```
Creates a new directed graph object with a descriptive comment "Software Architecture".

The package provides two main graph types:

1. `Digraph` - Directed graph (arrows show direction between nodes)
2. `Graph` - Undirected graph (connections have no direction)

The code starts with `Digraph` because this architecture diagram needs to show directional relationships (like data flow from frontend to API to database). If you were creating a diagram where relationships don't have direction (like a network of peers), you would start with:

```python
dot = Graph(comment='Network Diagram')
```

The import statement at the beginning of the tutorial shows both options:
```python
from graphviz import Digraph, Graph
```

your starting choice depends on whether your relationships need direction or not.


```python
dot.attr(rankdir='TB', size='8,10')
```
Sets graph attributes: 'TB' (top-to-bottom) layout direction and dimensions of 8×10 inches.

Graphviz supports four main layout direction options through the `rankdir` attribute:

1. `TB` - Top to Bottom (default)
2. `LR` - Left to Right
3. `BT` - Bottom to Top
4. `RL` - Right to Left

You can see this in the "Create Multiple Layout Variations" section of the tutorial where three of these options are demonstrated:

```python
layouts = {
    'top_down': {'rankdir': 'TB', 'filename': 'architecture_vertical'},
    'left_right': {'rankdir': 'LR', 'filename': 'architecture_horizontal'},
    'bottom_up': {'rankdir': 'BT', 'filename': 'architecture_bottom_up'}
}
```

The fourth option (`RL` - Right to Left) isn't shown in the example but is also available.

These direction options control the primary flow of the graph layout, which is particularly important for hierarchical diagrams.


1. `rankdir` (Rank Direction):
   - This is a Graphviz attribute that controls the primary flow direction of the graph layout
   - It determines how nodes are arranged relative to each other
   - Options include:
     - `TB`: Top to Bottom (nodes flow downward)
     - `LR`: Left to Right (nodes flow rightward)
     - `BT`: Bottom to Top (nodes flow upward)
     - `RL`: Right to Left (nodes flow leftward)

2. `filename`:
   - This is a custom parameter used in this code to specify the output filename for each layout variation
   - It defines the base name of the file that will be created when the graph is rendered
   - The code later uses these filenames when calling `layout_dot.render(config['filename'], format='png', cleanup=True)`
   - For example, `'architecture_vertical'` will create files named `architecture_vertical.png` and potentially `architecture_vertical.dot`

These parameters are used together to create multiple variations of the same graph with different orientations, each saved to a distinctly named file.

**Mixed Layouts:**

You can only select one primary direction (`rankdir`) for the overall graph layout. This sets the main flow direction for the entire graph.

However, you can create more complex layouts by:

1. **Using subgraphs/clusters** with different internal structures
2. **Constraining specific node positions** using invisible edges or rank statements
3. **Creating multiple graphs** with different directions and combining them in a presentation

For example, to create a mixed layout with some parts flowing top-to-bottom and others left-to-right:

````python path=mixed_layout_example.py mode=EDIT
from graphviz import Digraph

# Main graph with top-to-bottom flow
dot = Digraph(comment='Mixed Layout Example')
dot.attr(rankdir='TB')

# Create a left-to-right subgraph
with dot.subgraph(name='cluster_lr') as c:
    c.attr(rankdir='LR')  # Note: This doesn't override parent rankdir
    c.attr(label='Left-to-Right Section')
    c.node('A', 'Node A')
    c.node('B', 'Node B')
    c.node('C', 'Node C')
    c.edge('A', 'B')
    c.edge('B', 'C')

# Add nodes to main graph (top-to-bottom)
dot.node('D', 'Node D')
dot.node('E', 'Node E')
dot.edge('D', 'E')
dot.edge('C', 'D')  # Connect subgraph to main graph

# Use rank to force specific arrangements
dot.body.append('{ rank=same; A; B; C; }')  # Force A, B, C to be on same level

dot.render('mixed_layout', format='png', cleanup=True)
````

Note that while you can create complex layouts, Graphviz's automatic layout engine will still try to optimize the overall graph according to the main `rankdir`, so perfect mixed layouts can be challenging to achieve.

It's generally recommended to choose one orientation and stick to it for several reasons:

1. **Consistency**: A single orientation creates a more coherent and easier-to-understand diagram
2. **Predictability**: Graphviz's layout engine works best when given clear, consistent constraints
3. **Readability**: Mixed orientations can create confusing or counterintuitive layouts
4. **Maintainability**: Simpler layout rules make future modifications easier

While it's technically possible to create mixed layouts using subgraphs and rank constraints, these approaches often fight against Graphviz's automatic layout algorithms and can produce unpredictable results.

For most use cases, choosing the most appropriate single orientation (TB, LR, BT, or RL) based on your diagram's logical structure will yield the best results with minimal effort.


```python
dot.attr('node', shape='box', style='rounded,filled')
```
Sets default attributes for all nodes: box shape with rounded corners and filled background.

Graphviz offers many options for node shapes and styles. Here are the key options:

## Common Shape Options
- `box` - Rectangle (default)
- `ellipse` - Oval shape
- `circle` - Perfect circle
- `diamond` - Diamond shape
- `polygon` - Custom polygon (requires "sides" attribute)
- `triangle` - Triangle shape
- `plaintext` - No shape, just text
- `point` - Small point/dot
- `hexagon` - Six-sided shape
- `octagon` - Eight-sided shape
- `doublecircle` - Circle with double border
- `doubleoctagon` - Octagon with double border
- `tripleoctagon` - Octagon with triple border
- `Mdiamond` - Diamond with rounded corners
- `Msquare` - Square with rounded corners
- `Mcircle` - Circle with double border
- `house` - House shape (pentagon with bottom edge horizontal)
- `invhouse` - Inverted house shape
- `parallelogram` - Parallelogram shape
- `cylinder` - 3D cylinder shape
- `note` - Note/document shape
- `tab` - Tab/folder shape
- `folder` - Folder shape
- `box3d` - 3D box
- `component` - UML component shape

## Common Style Options
- `filled` - Fill with background color
- `rounded` - Round the corners (for shapes that support it)
- `dashed` - Dashed border line
- `dotted` - Dotted border line
- `solid` - Solid border line (default)
- `bold` - Bold border line
- `invis` - Make the node invisible
- `diagonals` - Add diagonal lines from corners

You can combine multiple styles with commas:
```python
dot.attr('node', shape='box', style='rounded,filled,bold')
```

Additional attributes like `color`, `fillcolor`, `penwidth`, and `fontname` can further customize the appearance.

**Distinguishing Graph, Node, and Edge**:

**Graph**:
- The overall container/diagram that holds all elements
- Represents the entire visualization structure
- Set with `dot.attr()` without specifying a type
- Controls global properties like layout direction, background color, and overall dimensions
- Example: `enhanced_dot.attr(rankdir='TB', bgcolor='white')`

**Node**:
- An individual element/component within the graph
- Represents a single entity in your architecture
- Set with `dot.attr('node', ...)` to define default properties for all nodes
- Individual nodes are added with `dot.node(id, label, **attrs)`
- Controls appearance of boxes/shapes in the diagram
- Example: `enhanced_dot.attr('node', shape='box', style='rounded,filled')`

In the architecture diagram:
- The graph is the entire software architecture visualization
- Nodes are the individual components (frontend, API, database, etc.)
- Edges (another element type) are the connections between nodes

Think of the graph as the canvas and nodes as the shapes placed on that canvas.

```python
# Add nodes
for node_id, attrs in components.items():
    dot.node(node_id, attrs['label'], fillcolor=attrs['color'])
```
Iterates through each component in the dictionary, adding a node with:
- ID from the dictionary key
- Label from the component's 'label' attribute
- Fill color from the component's 'color' attribute

You can add many additional attributes to your components dictionary. Here are the most useful options:

```python
components = {
    'node_id': {
        # Basic attributes
        'label': 'Display Label',
        'color': 'border_color',
        'fillcolor': 'background_color',
        
        # Visual styling
        'shape': 'box',                   # Node shape (box, circle, diamond, etc.)
        'style': 'rounded,filled',        # Combined styles
        'fontname': 'Arial',              # Font family
        'fontsize': '12',                 # Font size
        'fontcolor': 'black',             # Font color
        'penwidth': '2.0',                # Border thickness
        'width': '1.5',                   # Width in inches
        'height': '0.8',                  # Height in inches
        
        # Advanced styling
        'margin': '0.2',                  # Margin around label
        'URL': 'https://example.com',     # Clickable link (in SVG output)
        'tooltip': 'Hover text',          # Text shown on hover
        'peripheries': '2',               # Number of borders
        'gradientangle': '90',            # For gradient fills
        
        # Positioning hints
        'group': 'cluster1',              # Group for layout
        'rank': 'same',                   # Rank constraint
        'weight': '2.0',                  # Edge weight for layout
        
        # Custom data (not used by Graphviz but useful for your code)
        'category': 'database',           # Custom categorization
        'importance': 'high',             # Custom priority
        'version': '1.2.3'                # Custom versioning
    }
}
```

When adding nodes, you can pass any of these attributes:

```python
for node_id, attrs in components.items():
    # Extract known attributes
    label = attrs.get('label', node_id)
    fillcolor = attrs.get('fillcolor', attrs.get('color', 'white'))
    
    # Create a dictionary of all other attributes to pass
    node_attrs = {k: v for k, v in attrs.items() 
                 if k not in ['label', 'fillcolor']}
    
    # Add the node with all attributes
    dot.node(node_id, label, fillcolor=fillcolor, **node_attrs)
```

This approach lets you add any Graphviz-supported attribute to your components dictionary and have it applied to the nodes.


```python
# Add edges
for source, target in dependencies:
    dot.edge(source, target)
```
Iterates through each tuple in the dependencies list, creating directed edges between nodes.

```python
# Render the basic graph
dot.render('basic_architecture', format='png', cleanup=True)
```
Renders the graph to a file named 'basic_architecture.png' and removes temporary DOT files (cleanup=True).



### 4. Customize the Appearance

## Executive Summary

Section 4 creates an enhanced software architecture diagram with professional styling:

1. Initializes a directed graph with a clear title and consistent typography
2. Establishes uniform styling for nodes (components) and edges (connections)
3. Implements a refined color palette with subtle pastel colors for different system layers
4. Adds descriptive labels to connections that explain the relationship types (HTTP/REST, JWT validation, etc.)

The enhancements transform a basic functional diagram into a presentation-quality visualization that clearly communicates both the structure and the nature of interactions between system components, making it suitable for technical documentation and stakeholder presentations.

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

Line-by-line explanation of the selected code:

```python
# Create an enhanced version with better styling
enhanced_dot = Digraph(comment='Enhanced Software Architecture')
```
Creates a new directed graph object with a descriptive comment for an enhanced version of the architecture diagram.
The selected line is just creating a new Digraph object with a different comment. At this point in the code, there's no difference in content from the diagram in section 3.

The actual differences in content and styling come in the subsequent lines (not selected) where this new object is configured with:
1. More detailed graph attributes
2. Custom node and edge styling
3. A refined color palette
4. Edge labels describing relationship types

This line alone is simply initializing a fresh graph object that will be enhanced throughout the rest of section 4.

```python
# Set graph attributes
enhanced_dot.attr(rankdir='TB', 
                  bgcolor='white',
                  fontname='Arial',
                  fontsize='12',
                  label='Software Architecture Diagram',
                  labelloc='t')
```
Sets multiple graph attributes at once:
- `rankdir='TB'`: Top-to-bottom layout direction
- `bgcolor='white'`: White background color
- `fontname='Arial'`: Arial font for graph text
- `fontsize='12'`: 12pt font size
- `label='Software Architecture Diagram'`: Title for the entire graph
- `labelloc='t'`: Places the title at the top of the diagram

```python
# Set default node attributes
enhanced_dot.attr('node', 
                  shape='box',
                  style='rounded,filled',
                  fontname='Arial',
                  fontsize='10',
                  margin='0.2')
```
Sets default attributes for all nodes:
- `shape='box'`: Rectangular node shape
- `style='rounded,filled'`: Rounded corners with filled background
- `fontname='Arial'`: Arial font for node text
- `fontsize='10'`: 10pt font size for node text
- `margin='0.2'`: 0.2 inch margin around text inside nodes

```python
# Set default edge attributes
enhanced_dot.attr('edge',
                  fontname='Arial',
                  fontsize='8',
                  color='gray40',
                  arrowsize='0.8')
```
Sets default attributes for all edges:
- `fontname='Arial'`: Arial font for edge labels
- `fontsize='8'`: 8pt font size for edge labels
- `color='gray40'`: Dark gray color for edge lines
- `arrowsize='0.8'`: Slightly smaller arrow size (80% of default)

```python
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
```
Creates a dictionary mapping component IDs to specific hex color codes, using a more refined color palette than the basic version.

```python
for node_id, attrs in components.items():
    enhanced_dot.node(node_id, 
                      attrs['label'], 
                      fillcolor=layer_colors[node_id],
                      color='gray60')
```
Iterates through each component, adding a node with:
- ID from the dictionary key
- Label from the component's 'label' attribute
- Fill color from the new layer_colors dictionary
- Border color set to medium gray ('gray60')

```python
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
```
Creates a dictionary mapping edge connections (as tuples of source and target) to descriptive labels that explain the relationship type.

```python
for source, target in dependencies:
    label = edge_labels.get((source, target), '')
    enhanced_dot.edge(source, target, label=label)
```
Iterates through each dependency, creating an edge with:
- Source and target nodes from the dependency tuple
- A label from the edge_labels dictionary if one exists, otherwise an empty string


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
