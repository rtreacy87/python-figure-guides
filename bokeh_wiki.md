# Bokeh Guide

## Installation

```bash
pip install bokeh
# Optional dependencies for enhanced features
pip install pandas numpy scipy
# For Jupyter notebook support
pip install jupyter
```

## Overview

**Specialty:** Interactive web applications and dashboards

**Best Use Cases:**
- Interactive data dashboards
- Real-time data visualization
- Web-based analytics applications
- Large dataset visualization with good performance
- Custom interactive tools and widgets
- Streaming data visualization

## ASCII Example

Below is an ASCII representation of what we'll create - an interactive dashboard:

```
┌─────────────────────────────────────────────────────────────┐
│                    Sales Dashboard                         │
├─────────────────────────────────────────────────────────────┤
│ [Dropdown: Region ▼] [Slider: Year ────●────] [Button]     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Sales Trend                    │  Regional Performance     │
│                                 │                           │
│    ●●●●●                        │      ████                 │
│   ●     ●●                      │    ██████                 │
│  ●        ●●                    │  ████████                 │
│ ●           ●●                  │ ██████████                │
│                                 │                           │
├─────────────────────────────────┼───────────────────────────┤
│                                 │                           │
│  Product Categories             │  Performance Metrics      │
│                                 │                           │
│  ████ Electronics (45%)         │  Revenue: $2.5M           │
│  ███  Clothing (30%)            │  Growth:  +15%            │
│  ██   Books (25%)               │  Customers: 1,250         │
│                                 │  Avg Order: $125          │
└─────────────────────────────────┴───────────────────────────┘
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
from bokeh.plotting import figure, show, save, output_file
from bokeh.layouts import column, row, gridplot
from bokeh.models import ColumnDataSource, HoverTool, Select, Slider, Button
from bokeh.models import DatetimeTickFormatter, NumeralTickFormatter
from bokeh.io import curdoc
from bokeh.palettes import Category20, Viridis256
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
```

### 2. Prepare Your Data

```python
# Generate sample sales data
np.random.seed(42)

# Create date range
start_date = datetime(2020, 1, 1)
end_date = datetime(2023, 12, 31)
date_range = pd.date_range(start_date, end_date, freq='D')

# Generate sales data
n_days = len(date_range)
base_sales = 1000
trend = np.linspace(0, 500, n_days)  # Growing trend
seasonal = 200 * np.sin(2 * np.pi * np.arange(n_days) / 365.25)  # Yearly seasonality
noise = np.random.normal(0, 100, n_days)

sales_data = base_sales + trend + seasonal + noise
sales_data = np.maximum(sales_data, 0)  # Ensure non-negative

# Create DataFrame
df = pd.DataFrame({
    'date': date_range,
    'sales': sales_data,
    'region': np.random.choice(['North', 'South', 'East', 'West'], n_days),
    'product_category': np.random.choice(['Electronics', 'Clothing', 'Books'], n_days),
    'customers': np.random.poisson(50, n_days)
})

# Add derived metrics
df['revenue'] = df['sales'] * np.random.uniform(80, 150, n_days)
df['avg_order_value'] = df['revenue'] / df['customers']

print(f"Dataset shape: {df.shape}")
print(df.head())
```

### 3. Create the Basic Visualization

```python
# Create basic interactive plots
output_file("sales_dashboard.html")

# Prepare data source
source = ColumnDataSource(df)

# 1. Time series plot
p1 = figure(title="Sales Trend Over Time", 
           x_axis_type='datetime',
           width=600, height=300,
           tools="pan,wheel_zoom,box_zoom,reset,save")

p1.line('date', 'sales', source=source, line_width=2, color='navy', alpha=0.8)
p1.circle('date', 'sales', source=source, size=4, color='navy', alpha=0.6)

# Add hover tool
hover1 = HoverTool(tooltips=[
    ('Date', '@date{%F}'),
    ('Sales', '@sales{0,0}'),
    ('Region', '@region')
], formatters={'@date': 'datetime'})
p1.add_tools(hover1)

# 2. Regional performance bar chart
regions_summary = df.groupby('region')['sales'].sum().reset_index()
regions_source = ColumnDataSource(regions_summary)

p2 = figure(x_range=regions_summary['region'], 
           title="Sales by Region",
           width=400, height=300,
           tools="pan,wheel_zoom,box_zoom,reset,save")

p2.vbar(x='region', top='sales', width=0.8, source=regions_source,
        color=Category20[4], alpha=0.8)

hover2 = HoverTool(tooltips=[('Region', '@region'), ('Total Sales', '@sales{0,0}')])
p2.add_tools(hover2)

# 3. Product category pie chart (using a donut chart)
categories_summary = df.groupby('product_category')['sales'].sum().reset_index()
categories_summary['angle'] = categories_summary['sales'] / categories_summary['sales'].sum() * 2 * np.pi
categories_summary['color'] = Category20[len(categories_summary)]

p3 = figure(title="Sales by Product Category", 
           width=400, height=300,
           tools="hover,reset,save")

p3.annular_wedge(x=0, y=1, inner_radius=0.2, outer_radius=0.4,
                start_angle='angle', end_angle='angle',
                color='color', alpha=0.8,
                source=ColumnDataSource(categories_summary))

# Layout the plots
layout = gridplot([[p1], [p2, p3]])
show(layout)
```

### 4. Customize the Appearance

```python
# Create an enhanced dashboard with custom styling
from bokeh.models import Div
from bokeh.plotting import figure

# Custom styling
output_file("enhanced_sales_dashboard.html")

# Create title
title_div = Div(text="""
<h1 style="text-align: center; color: #2F4F4F;">Sales Analytics Dashboard</h1>
<p style="text-align: center; color: #696969;">Interactive visualization of sales performance metrics</p>
""", width=800, height=80)

# Enhanced time series with multiple metrics
p1 = figure(title="Sales and Revenue Trends", 
           x_axis_type='datetime',
           width=800, height=350,
           tools="pan,wheel_zoom,box_zoom,reset,save",
           background_fill_color="#fafafa")

# Sales line
p1.line('date', 'sales', source=source, line_width=3, color='#1f77b4', 
        alpha=0.8, legend_label="Sales")

# Revenue line (secondary axis simulation)
revenue_normalized = df['revenue'] / df['revenue'].max() * df['sales'].max()
source.add(revenue_normalized, 'revenue_norm')

p1.line('date', 'revenue_norm', source=source, line_width=3, color='#ff7f0e', 
        alpha=0.8, legend_label="Revenue (normalized)")

# Styling
p1.legend.location = "top_left"
p1.legend.click_policy = "hide"
p1.xaxis.formatter = DatetimeTickFormatter(days="%m/%d", months="%m/%Y")
p1.yaxis.formatter = NumeralTickFormatter(format="0,0")

# Enhanced hover
hover_enhanced = HoverTool(tooltips=[
    ('Date', '@date{%F}'),
    ('Sales', '@sales{0,0}'),
    ('Revenue', '@revenue{$0,0.00}'),
    ('Region', '@region'),
    ('Category', '@product_category')
], formatters={'@date': 'datetime'})
p1.add_tools(hover_enhanced)

# Regional heatmap
from bokeh.transform import linear_cmap
from bokeh.models import ColorBar

# Create monthly regional data
df['month'] = df['date'].dt.to_period('M')
heatmap_data = df.groupby(['region', 'month'])['sales'].sum().reset_index()
heatmap_data['month_str'] = heatmap_data['month'].astype(str)

heatmap_source = ColumnDataSource(heatmap_data)

p2 = figure(title="Regional Sales Heatmap (Monthly)",
           x_range=sorted(heatmap_data['month_str'].unique()),
           y_range=sorted(heatmap_data['region'].unique()),
           width=800, height=300,
           tools="hover,reset,save",
           toolbar_location="above")

mapper = linear_cmap(field_name='sales', palette=Viridis256, 
                    low=heatmap_data['sales'].min(), high=heatmap_data['sales'].max())

p2.rect(x='month_str', y='region', width=1, height=1,
        source=heatmap_source, fill_color=mapper, line_color=None)

color_bar = ColorBar(color_mapper=mapper['transform'], width=8, location=(0,0))
p2.add_layout(color_bar, 'right')

p2.xaxis.major_label_orientation = 45

# Layout with title
enhanced_layout = column(title_div, p1, p2)
show(enhanced_layout)
```

### 5. Add Interactivity

```python
# Create a fully interactive dashboard with widgets
from bokeh.models import CustomJS

# Create interactive widgets
region_select = Select(title="Select Region:", value="All", 
                      options=["All"] + list(df['region'].unique()))

year_slider = Slider(start=2020, end=2023, value=2023, step=1, title="Year")

update_button = Button(label="Update Dashboard", button_type="success")

# Create filtered data source
filtered_source = ColumnDataSource(df)

# JavaScript callback for interactivity
callback = CustomJS(args=dict(source=source, filtered_source=filtered_source,
                             region_select=region_select, year_slider=year_slider), code="""
    const data = source.data;
    const filtered_data = filtered_source.data;
    const region = region_select.value;
    const year = year_slider.value;
    
    // Clear filtered data
    for (let key in filtered_data) {
        filtered_data[key] = [];
    }
    
    // Filter data based on selections
    for (let i = 0; i < data['date'].length; i++) {
        const date = new Date(data['date'][i]);
        const item_region = data['region'][i];
        
        if ((region === 'All' || item_region === region) && 
            date.getFullYear() === year) {
            for (let key in data) {
                filtered_data[key].push(data[key][i]);
            }
        }
    }
    
    filtered_source.change.emit();
""")

# Attach callbacks
region_select.js_on_change('value', callback)
year_slider.js_on_change('value', callback)
update_button.js_on_click(callback)

# Create plots using filtered source
interactive_p1 = figure(title="Filtered Sales Data", 
                       x_axis_type='datetime',
                       width=700, height=300,
                       tools="pan,wheel_zoom,reset,save")

interactive_p1.line('date', 'sales', source=filtered_source, 
                   line_width=2, color='blue')
interactive_p1.circle('date', 'sales', source=filtered_source, 
                     size=6, color='blue', alpha=0.6)

# Widget panel
widgets = column(region_select, year_slider, update_button)

# Complete interactive layout
interactive_layout = row(widgets, interactive_p1)
show(interactive_layout)
```

### 6. Export or Display

```python
# Create and save the complete dashboard
output_file("complete_sales_dashboard.html")

# Summary statistics
summary_div = Div(text=f"""
<div style="background-color: #f0f0f0; padding: 20px; border-radius: 5px;">
<h3>Dashboard Summary</h3>
<ul>
<li>Total Sales: {df['sales'].sum():,.0f}</li>
<li>Total Revenue: ${df['revenue'].sum():,.2f}</li>
<li>Average Daily Sales: {df['sales'].mean():.0f}</li>
<li>Date Range: {df['date'].min().strftime('%Y-%m-%d')} to {df['date'].max().strftime('%Y-%m-%d')}</li>
</ul>
</div>
""", width=300, height=200)

# Final comprehensive layout
final_layout = column(
    title_div,
    row(interactive_layout),
    row(p2),
    row(summary_div)
)

save(final_layout)
show(final_layout)

print("Interactive dashboard created successfully!")
print("Files generated:")
print("- sales_dashboard.html")
print("- enhanced_sales_dashboard.html")
print("- complete_sales_dashboard.html")
print("\nOpen any HTML file in a web browser to interact with the dashboard!")
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Poor performance with large datasets
  - Solution: Use data decimation, server-side filtering, or Bokeh's DataTable for large data
- **Pitfall 2**: Callback complexity in JavaScript
  - Solution: Use Python callbacks with Bokeh server for complex interactions
- **Pitfall 3**: Layout issues on different screen sizes
  - Solution: Use responsive sizing and test on multiple devices

## Advanced Techniques

- Building real-time dashboards with Bokeh server
- Creating custom JavaScript extensions and widgets
- Implementing streaming data visualization
- Using Bokeh with web frameworks (Flask, Django)
- Creating mobile-responsive dashboards
- Integrating with databases for dynamic data loading

## Resources

- [Bokeh Documentation](https://docs.bokeh.org/en/latest/)
- [Bokeh Gallery](https://docs.bokeh.org/en/latest/docs/gallery.html)
- [Bokeh Server Guide](https://docs.bokeh.org/en/latest/docs/user_guide/server.html)
- [Interactive Data Visualization with Bokeh](https://realpython.com/python-data-visualization-bokeh/)
