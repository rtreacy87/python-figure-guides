# Altair Guide

## Installation

```bash
pip install altair
# For enhanced functionality
pip install altair[all]
# Required for rendering in Jupyter
pip install vega_datasets
```

## Overview

**Specialty:** Statistical visualizations following grammar of graphics

**Best Use Cases:**
- Exploratory data analysis with clean syntax
- Statistical data visualization
- Multi-dimensional data exploration
- Interactive statistical charts
- Declarative visualization specification
- Quick prototyping of data visualizations

## ASCII Example

Below is an ASCII representation of what we'll create - a multi-faceted statistical analysis:

```
┌─────────────────────────────────────────────────────────────┐
│                Statistical Analysis Dashboard               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Scatter Plot Matrix        │  Distribution by Category     │
│                             │                               │
│    ●●●●●                    │      ████                     │
│   ●     ●●                  │    ██████                     │
│  ●        ●●                │  ████████                     │
│ ●           ●●              │ ██████████                    │
│                             │                               │
├─────────────────────────────┼───────────────────────────────┤
│                             │                               │
│  Time Series Trend          │  Correlation Heatmap          │
│                             │                               │
│    ●●●●●                    │  [1.0][0.8][0.3]             │
│   ●     ●●                  │  [0.8][1.0][0.5]             │
│  ●        ●●                │  [0.3][0.5][1.0]             │
│ ●           ●●              │                               │
│                             │                               │
└─────────────────────────────┴───────────────────────────────┘
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
import altair as alt
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
from vega_datasets import data

# Enable Altair to render in Jupyter notebooks
alt.data_transformers.enable('json')

# Set theme
alt.themes.enable('opaque')
```

### 2. Prepare Your Data

```python
# Generate sample dataset for comprehensive analysis
np.random.seed(42)

# Create a dataset simulating customer behavior over time
n_records = 1000
start_date = datetime(2023, 1, 1)

# Generate synthetic customer data
customer_data = {
    'customer_id': range(1, n_records + 1),
    'age': np.random.normal(35, 12, n_records),
    'income': np.random.normal(50000, 15000, n_records),
    'spending_score': np.random.normal(50, 25, n_records),
    'satisfaction': np.random.normal(7, 2, n_records),
    'region': np.random.choice(['North', 'South', 'East', 'West'], n_records),
    'customer_type': np.random.choice(['Premium', 'Standard', 'Basic'], n_records, p=[0.2, 0.5, 0.3]),
    'signup_date': [start_date + timedelta(days=np.random.randint(0, 365)) for _ in range(n_records)]
}

# Add correlations to make data more realistic
customer_data['spending_score'] = (customer_data['spending_score'] + 
                                  0.3 * (customer_data['income'] - 50000) / 1000)
customer_data['satisfaction'] = (customer_data['satisfaction'] + 
                               0.2 * customer_data['spending_score'] / 10)

# Create DataFrame
df = pd.DataFrame(customer_data)

# Clean and constrain data
df['age'] = np.clip(df['age'], 18, 80)
df['income'] = np.clip(df['income'], 20000, 150000)
df['spending_score'] = np.clip(df['spending_score'], 0, 100)
df['satisfaction'] = np.clip(df['satisfaction'], 1, 10)

# Add time-based features
df['signup_month'] = df['signup_date'].dt.month
df['signup_quarter'] = df['signup_date'].dt.quarter

print(f"Dataset shape: {df.shape}")
print(df.head())
```

### 3. Create the Basic Visualization

```python
# Create basic statistical visualizations using Altair's grammar of graphics

# 1. Scatter plot with regression line
scatter_plot = alt.Chart(df).mark_circle(size=60, opacity=0.7).encode(
    x=alt.X('income:Q', title='Annual Income ($)'),
    y=alt.Y('spending_score:Q', title='Spending Score'),
    color=alt.Color('customer_type:N', title='Customer Type'),
    tooltip=['customer_id:O', 'income:Q', 'spending_score:Q', 'customer_type:N']
).properties(
    title='Income vs Spending Score by Customer Type',
    width=400,
    height=300
)

# Add regression line
regression_line = alt.Chart(df).mark_line(color='red', strokeDash=[5, 5]).transform_regression(
    'income', 'spending_score'
).encode(
    x='income:Q',
    y='spending_score:Q'
)

scatter_with_regression = scatter_plot + regression_line

# 2. Distribution histogram
histogram = alt.Chart(df).mark_bar(opacity=0.7).encode(
    x=alt.X('age:Q', bin=alt.Bin(maxbins=20), title='Age'),
    y=alt.Y('count():Q', title='Count'),
    color=alt.Color('customer_type:N', title='Customer Type')
).properties(
    title='Age Distribution by Customer Type',
    width=400,
    height=300
)

# 3. Box plot for satisfaction by region
box_plot = alt.Chart(df).mark_boxplot(size=50).encode(
    x=alt.X('region:N', title='Region'),
    y=alt.Y('satisfaction:Q', title='Satisfaction Score'),
    color=alt.Color('region:N', title='Region')
).properties(
    title='Satisfaction Distribution by Region',
    width=400,
    height=300
)

# Combine plots
basic_dashboard = alt.vconcat(
    alt.hconcat(scatter_with_regression, histogram),
    box_plot
).resolve_scale(color='independent')

basic_dashboard.show()
```

### 4. Customize the Appearance

```python
# Create an enhanced dashboard with custom styling and themes

# Define custom color scheme
custom_colors = ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728', '#9467bd']

# Enhanced scatter plot with selection interaction
brush = alt.selection_interval()

enhanced_scatter = alt.Chart(df).mark_circle(size=80).encode(
    x=alt.X('income:Q', 
            title='Annual Income ($)',
            scale=alt.Scale(zero=False)),
    y=alt.Y('spending_score:Q', 
            title='Spending Score',
            scale=alt.Scale(zero=False)),
    color=alt.condition(brush, 
                       alt.Color('customer_type:N', 
                                scale=alt.Scale(range=custom_colors),
                                title='Customer Type'),
                       alt.value('lightgray')),
    size=alt.Size('satisfaction:Q', 
                  scale=alt.Scale(range=[50, 200]),
                  title='Satisfaction'),
    tooltip=['customer_id:O', 'income:Q', 'spending_score:Q', 
             'satisfaction:Q', 'customer_type:N', 'region:N']
).add_selection(
    brush
).properties(
    title=alt.TitleParams(
        text='Customer Segmentation Analysis',
        fontSize=16,
        fontWeight='bold'
    ),
    width=500,
    height=350
)

# Linked histogram that responds to selection
linked_histogram = alt.Chart(df).mark_bar().encode(
    x=alt.X('satisfaction:Q', 
            bin=alt.Bin(maxbins=15),
            title='Satisfaction Score'),
    y=alt.Y('count():Q', title='Number of Customers'),
    color=alt.Color('customer_type:N', 
                   scale=alt.Scale(range=custom_colors),
                   title='Customer Type'),
    opacity=alt.condition(brush, alt.value(0.8), alt.value(0.3))
).transform_filter(
    brush
).properties(
    title='Satisfaction Distribution (Filtered)',
    width=500,
    height=200
)

# Time series analysis
monthly_signups = df.groupby(['signup_month', 'customer_type']).size().reset_index(name='count')

time_series = alt.Chart(monthly_signups).mark_line(point=True, strokeWidth=3).encode(
    x=alt.X('signup_month:O', title='Signup Month'),
    y=alt.Y('count:Q', title='Number of Signups'),
    color=alt.Color('customer_type:N', 
                   scale=alt.Scale(range=custom_colors),
                   title='Customer Type'),
    tooltip=['signup_month:O', 'customer_type:N', 'count:Q']
).properties(
    title='Monthly Customer Signups by Type',
    width=500,
    height=250
)

# Enhanced dashboard layout
enhanced_dashboard = alt.vconcat(
    enhanced_scatter,
    alt.hconcat(linked_histogram, time_series)
).resolve_scale(color='shared')

enhanced_dashboard.show()
```

### 5. Add Advanced Statistical Analysis

```python
# Create advanced statistical visualizations

# 1. Correlation matrix heatmap
correlation_data = df[['age', 'income', 'spending_score', 'satisfaction']].corr().reset_index()
correlation_melted = correlation_data.melt('index', var_name='variable', value_name='correlation')

correlation_heatmap = alt.Chart(correlation_melted).mark_rect().encode(
    x=alt.X('index:N', title=''),
    y=alt.Y('variable:N', title=''),
    color=alt.Color('correlation:Q', 
                   scale=alt.Scale(scheme='redblue', domain=[-1, 1]),
                   title='Correlation'),
    tooltip=['index:N', 'variable:N', 'correlation:Q']
).properties(
    title='Feature Correlation Matrix',
    width=300,
    height=300
)

# Add correlation values as text
correlation_text = alt.Chart(correlation_melted).mark_text(
    fontSize=12,
    fontWeight='bold'
).encode(
    x=alt.X('index:N'),
    y=alt.Y('variable:N'),
    text=alt.Text('correlation:Q', format='.2f'),
    color=alt.condition(alt.datum.correlation > 0.5, alt.value('white'), alt.value('black'))
)

correlation_chart = correlation_heatmap + correlation_text

# 2. Multi-dimensional analysis with faceting
faceted_analysis = alt.Chart(df).mark_circle(size=60, opacity=0.7).encode(
    x=alt.X('income:Q', title='Income'),
    y=alt.Y('spending_score:Q', title='Spending Score'),
    color=alt.Color('satisfaction:Q', 
                   scale=alt.Scale(scheme='viridis'),
                   title='Satisfaction'),
    facet=alt.Facet('region:N', columns=2, title='Region'),
    tooltip=['customer_id:O', 'income:Q', 'spending_score:Q', 'satisfaction:Q']
).properties(
    title='Regional Customer Analysis',
    width=200,
    height=150
)

# 3. Statistical summary with error bars
summary_stats = df.groupby('customer_type').agg({
    'satisfaction': ['mean', 'std'],
    'spending_score': ['mean', 'std']
}).round(2)

summary_stats.columns = ['satisfaction_mean', 'satisfaction_std', 
                        'spending_mean', 'spending_std']
summary_stats = summary_stats.reset_index()

error_bars = alt.Chart(summary_stats).mark_errorbar(extent='stdev').encode(
    x=alt.X('customer_type:N', title='Customer Type'),
    y=alt.Y('satisfaction_mean:Q', title='Average Satisfaction'),
    yError='satisfaction_std:Q'
)

points = alt.Chart(summary_stats).mark_point(size=100, filled=True).encode(
    x=alt.X('customer_type:N'),
    y=alt.Y('satisfaction_mean:Q'),
    color=alt.Color('customer_type:N', scale=alt.Scale(range=custom_colors))
)

error_chart = (error_bars + points).properties(
    title='Average Satisfaction by Customer Type (with Standard Deviation)',
    width=300,
    height=250
)

# Combine advanced visualizations
advanced_dashboard = alt.vconcat(
    alt.hconcat(correlation_chart, error_chart),
    faceted_analysis
)

advanced_dashboard.show()
```

### 6. Export or Display

```python
# Create final comprehensive dashboard and export

# Combine all visualizations into a comprehensive dashboard
final_dashboard = alt.vconcat(
    alt.hconcat(enhanced_scatter),
    alt.hconcat(linked_histogram, time_series),
    alt.hconcat(correlation_chart, error_chart),
    faceted_analysis
).resolve_scale(
    color='independent'
).properties(
    title=alt.TitleParams(
        text='Comprehensive Customer Analytics Dashboard',
        fontSize=20,
        fontWeight='bold',
        anchor='start'
    )
)

# Save the dashboard
final_dashboard.save('customer_analytics_dashboard.html')

# Save individual charts
enhanced_scatter.save('customer_segmentation.html')
correlation_chart.save('correlation_analysis.html')
faceted_analysis.save('regional_analysis.html')

# Export as JSON specification (for sharing/version control)
final_dashboard.save('dashboard_spec.json')

# Display summary statistics
print("Dashboard Export Complete!")
print("\nDataset Summary:")
print(f"Total Customers: {len(df):,}")
print(f"Average Satisfaction: {df['satisfaction'].mean():.2f}")
print(f"Average Spending Score: {df['spending_score'].mean():.1f}")
print(f"Income Range: ${df['income'].min():,.0f} - ${df['income'].max():,.0f}")

print("\nFiles Generated:")
print("- customer_analytics_dashboard.html")
print("- customer_segmentation.html")
print("- correlation_analysis.html")
print("- regional_analysis.html")
print("- dashboard_spec.json")

# Show the final dashboard
final_dashboard.show()
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Data size limitations (5000 rows default)
  - Solution: Use `alt.data_transformers.enable('json')` or server-side data processing
- **Pitfall 2**: Complex interactions not working as expected
  - Solution: Use simpler selection types and test incrementally
- **Pitfall 3**: Charts not displaying in certain environments
  - Solution: Ensure proper Altair and Vega-Lite versions, use `alt.renderers.enable('default')`

## Advanced Techniques

- Creating custom transformations and calculations within charts
- Building interactive dashboards with linked selections
- Using Altair with streaming data sources
- Implementing custom color schemes and themes
- Creating animated visualizations with temporal data
- Integrating Altair charts into web applications

## Resources

- [Altair Documentation](https://altair-viz.github.io/)
- [Altair Gallery](https://altair-viz.github.io/gallery/index.html)
- [Vega-Lite Documentation](https://vega.github.io/vega-lite/)
- [Grammar of Graphics Principles](https://www.springer.com/gp/book/9780387245447)
