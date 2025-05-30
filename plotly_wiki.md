# Plotly Guide

## Installation

```bash
pip install plotly
# For enhanced features and Jupyter support
pip install plotly[complete]
# For Dash web applications
pip install dash
```

## Overview

**Specialty:** Interactive web-based visualizations and dashboards

**Best Use Cases:**
- Interactive web dashboards and applications
- 3D visualizations and scientific plotting
- Financial data visualization with real-time updates
- Geographic and mapping visualizations
- Presentation-quality interactive charts
- Cross-platform visualization (web, mobile, desktop)

## ASCII Example

Below is an ASCII representation of what we'll create - an interactive financial dashboard:

```
┌─────────────────────────────────────────────────────────────┐
│                Financial Analytics Dashboard                │
├─────────────────────────────────────────────────────────────┤
│ [Dropdown: Stock ▼] [Date Range: ──●──●──] [Update Button] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Stock Price Chart              │  Volume Analysis          │
│                                 │                           │
│    ●●●●●                        │      ████                 │
│   ●     ●●                      │    ██████                 │
│  ●        ●●                    │  ████████                 │
│ ●           ●●                  │ ██████████                │
│                                 │                           │
├─────────────────────────────────┼───────────────────────────┤
│                                 │                           │
│  Portfolio Performance          │  Risk Metrics             │
│                                 │                           │
│  ████ AAPL (40%)                │  Volatility: 15.2%        │
│  ███  GOOGL (30%)               │  Sharpe Ratio: 1.45       │
│  ██   MSFT (30%)                │  Max Drawdown: -8.5%      │
│                                 │  Beta: 1.12               │
└─────────────────────────────────┴───────────────────────────┘
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
import plotly.graph_objects as go
import plotly.express as px
from plotly.subplots import make_subplots
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import plotly.offline as pyo

# Set default template
import plotly.io as pio
pio.templates.default = "plotly_white"
```

### 2. Prepare Your Data

```python
# Generate sample financial data
np.random.seed(42)

# Create date range
start_date = datetime(2023, 1, 1)
end_date = datetime(2023, 12, 31)
dates = pd.date_range(start_date, end_date, freq='D')

# Generate stock data for multiple companies
companies = ['AAPL', 'GOOGL', 'MSFT', 'AMZN', 'TSLA']
stock_data = []

for company in companies:
    # Generate realistic stock price data
    n_days = len(dates)
    initial_price = np.random.uniform(100, 300)
    
    # Random walk with trend
    returns = np.random.normal(0.001, 0.02, n_days)  # Daily returns
    prices = [initial_price]
    
    for i in range(1, n_days):
        price = prices[-1] * (1 + returns[i])
        prices.append(max(price, 1))  # Ensure positive prices
    
    # Generate volume data
    base_volume = np.random.uniform(1e6, 5e6)
    volumes = np.random.lognormal(np.log(base_volume), 0.3, n_days)
    
    # Create DataFrame for this company
    company_df = pd.DataFrame({
        'date': dates,
        'company': company,
        'price': prices,
        'volume': volumes
    })
    
    # Add technical indicators
    company_df['price_ma_7'] = company_df['price'].rolling(7).mean()
    company_df['price_ma_30'] = company_df['price'].rolling(30).mean()
    company_df['daily_return'] = company_df['price'].pct_change()
    company_df['volatility'] = company_df['daily_return'].rolling(30).std() * np.sqrt(252)
    
    stock_data.append(company_df)

# Combine all data
df = pd.concat(stock_data, ignore_index=True)

# Add market data
market_data = df.groupby('date').agg({
    'price': 'mean',
    'volume': 'sum',
    'daily_return': 'mean'
}).reset_index()
market_data['company'] = 'MARKET'
market_data['price_ma_7'] = market_data['price'].rolling(7).mean()
market_data['price_ma_30'] = market_data['price'].rolling(30).mean()
market_data['volatility'] = market_data['daily_return'].rolling(30).std() * np.sqrt(252)

print(f"Dataset shape: {df.shape}")
print(f"Companies: {companies}")
print(f"Date range: {df['date'].min()} to {df['date'].max()}")
```

### 3. Create the Basic Visualization

```python
# Create basic interactive financial charts

# 1. Stock price chart with candlestick
def create_candlestick_chart(company_data, company_name):
    # Prepare OHLC data (simplified - using price as close)
    ohlc_data = company_data.copy()
    ohlc_data['open'] = ohlc_data['price'].shift(1)
    ohlc_data['high'] = ohlc_data[['price', 'open']].max(axis=1) * 1.02
    ohlc_data['low'] = ohlc_data[['price', 'open']].min(axis=1) * 0.98
    ohlc_data['close'] = ohlc_data['price']
    
    fig = go.Figure()
    
    # Add candlestick
    fig.add_trace(go.Candlestick(
        x=ohlc_data['date'],
        open=ohlc_data['open'],
        high=ohlc_data['high'],
        low=ohlc_data['low'],
        close=ohlc_data['close'],
        name=f'{company_name} Price'
    ))
    
    # Add moving averages
    fig.add_trace(go.Scatter(
        x=ohlc_data['date'],
        y=ohlc_data['price_ma_7'],
        mode='lines',
        name='7-day MA',
        line=dict(color='orange', width=1)
    ))
    
    fig.add_trace(go.Scatter(
        x=ohlc_data['date'],
        y=ohlc_data['price_ma_30'],
        mode='lines',
        name='30-day MA',
        line=dict(color='red', width=1)
    ))
    
    fig.update_layout(
        title=f'{company_name} Stock Price with Moving Averages',
        xaxis_title='Date',
        yaxis_title='Price ($)',
        height=400,
        showlegend=True
    )
    
    return fig

# Create chart for AAPL
aapl_data = df[df['company'] == 'AAPL'].copy()
candlestick_fig = create_candlestick_chart(aapl_data, 'AAPL')
candlestick_fig.show()

# 2. Volume analysis
volume_fig = px.bar(
    aapl_data, 
    x='date', 
    y='volume',
    title='AAPL Trading Volume',
    labels={'volume': 'Volume', 'date': 'Date'}
)
volume_fig.update_layout(height=300)
volume_fig.show()

# 3. Multi-company comparison
comparison_fig = px.line(
    df, 
    x='date', 
    y='price', 
    color='company',
    title='Stock Price Comparison',
    labels={'price': 'Price ($)', 'date': 'Date'}
)
comparison_fig.update_layout(height=400)
comparison_fig.show()
```

### 4. Customize the Appearance

```python
# Create an enhanced financial dashboard with custom styling

# Create subplot layout
fig = make_subplots(
    rows=3, cols=2,
    subplot_titles=('Stock Prices', 'Volume Analysis', 
                   'Daily Returns', 'Volatility Comparison',
                   'Correlation Heatmap', 'Portfolio Performance'),
    specs=[[{"secondary_y": False}, {"secondary_y": False}],
           [{"secondary_y": False}, {"secondary_y": False}],
           [{"secondary_y": False}, {"secondary_y": False}]],
    vertical_spacing=0.08,
    horizontal_spacing=0.1
)

# 1. Stock prices (normalized to 100 for comparison)
for i, company in enumerate(companies[:3]):  # Show top 3 companies
    company_data = df[df['company'] == company].copy()
    normalized_price = (company_data['price'] / company_data['price'].iloc[0]) * 100
    
    fig.add_trace(
        go.Scatter(
            x=company_data['date'],
            y=normalized_price,
            mode='lines',
            name=f'{company} (Normalized)',
            line=dict(width=2)
        ),
        row=1, col=1
    )

# 2. Volume analysis
total_volume = df.groupby('date')['volume'].sum().reset_index()
fig.add_trace(
    go.Bar(
        x=total_volume['date'],
        y=total_volume['volume'],
        name='Total Volume',
        marker_color='lightblue',
        opacity=0.7
    ),
    row=1, col=2
)

# 3. Daily returns distribution
for company in companies[:3]:
    company_data = df[df['company'] == company]
    fig.add_trace(
        go.Histogram(
            x=company_data['daily_return'].dropna(),
            name=f'{company} Returns',
            opacity=0.7,
            nbinsx=30
        ),
        row=2, col=1
    )

# 4. Volatility comparison
volatility_data = df.groupby('company')['volatility'].last().reset_index()
fig.add_trace(
    go.Bar(
        x=volatility_data['company'],
        y=volatility_data['volatility'],
        name='Volatility',
        marker_color='red',
        opacity=0.7
    ),
    row=2, col=2
)

# 5. Correlation heatmap
price_pivot = df.pivot(index='date', columns='company', values='price')
correlation_matrix = price_pivot.corr()

fig.add_trace(
    go.Heatmap(
        z=correlation_matrix.values,
        x=correlation_matrix.columns,
        y=correlation_matrix.index,
        colorscale='RdBu',
        zmid=0,
        text=correlation_matrix.round(2).values,
        texttemplate="%{text}",
        textfont={"size": 10},
        name='Correlation'
    ),
    row=3, col=1
)

# 6. Portfolio performance (pie chart)
portfolio_weights = [0.3, 0.25, 0.2, 0.15, 0.1]
fig.add_trace(
    go.Pie(
        labels=companies,
        values=portfolio_weights,
        name='Portfolio',
        hole=0.3
    ),
    row=3, col=2
)

# Update layout
fig.update_layout(
    title_text="Financial Analytics Dashboard",
    title_x=0.5,
    title_font_size=20,
    height=900,
    showlegend=False
)

# Update axes labels
fig.update_xaxes(title_text="Date", row=1, col=1)
fig.update_yaxes(title_text="Normalized Price", row=1, col=1)
fig.update_xaxes(title_text="Date", row=1, col=2)
fig.update_yaxes(title_text="Volume", row=1, col=2)
fig.update_xaxes(title_text="Daily Return", row=2, col=1)
fig.update_yaxes(title_text="Frequency", row=2, col=1)
fig.update_xaxes(title_text="Company", row=2, col=2)
fig.update_yaxes(title_text="Volatility (%)", row=2, col=2)

fig.show()
```

### 5. Add Advanced Interactivity

```python
# Create advanced interactive features

# Interactive stock selector with dropdown
from plotly.graph_objects import FigureWidget
import ipywidgets as widgets

# Create interactive chart with dropdown
def create_interactive_dashboard():
    # Initial data
    initial_company = 'AAPL'
    initial_data = df[df['company'] == initial_company]
    
    # Create figure widget
    fig_widget = go.FigureWidget()
    
    # Add initial traces
    fig_widget.add_scatter(
        x=initial_data['date'],
        y=initial_data['price'],
        mode='lines+markers',
        name='Price',
        line=dict(color='blue', width=2)
    )
    
    fig_widget.add_scatter(
        x=initial_data['date'],
        y=initial_data['price_ma_30'],
        mode='lines',
        name='30-day MA',
        line=dict(color='red', width=1, dash='dash')
    )
    
    fig_widget.update_layout(
        title='Interactive Stock Analysis',
        xaxis_title='Date',
        yaxis_title='Price ($)',
        height=400
    )
    
    # Create dropdown widget
    dropdown = widgets.Dropdown(
        options=companies,
        value=initial_company,
        description='Stock:'
    )
    
    # Update function
    def update_chart(change):
        selected_company = change['new']
        company_data = df[df['company'] == selected_company]
        
        with fig_widget.batch_update():
            fig_widget.data[0].x = company_data['date']
            fig_widget.data[0].y = company_data['price']
            fig_widget.data[1].x = company_data['date']
            fig_widget.data[1].y = company_data['price_ma_30']
            fig_widget.layout.title = f'Interactive Stock Analysis - {selected_company}'
    
    dropdown.observe(update_chart, names='value')
    
    return widgets.VBox([dropdown, fig_widget])

# Create 3D surface plot for advanced visualization
def create_3d_surface():
    # Create 3D data: Date vs Company vs Price
    pivot_data = df.pivot(index='date', columns='company', values='price')
    
    # Create meshgrid
    x = np.arange(len(pivot_data.columns))
    y = np.arange(len(pivot_data.index))
    X, Y = np.meshgrid(x, y)
    Z = pivot_data.values
    
    fig_3d = go.Figure(data=[go.Surface(
        z=Z,
        x=pivot_data.columns,
        y=pivot_data.index,
        colorscale='Viridis',
        name='Stock Prices'
    )])
    
    fig_3d.update_layout(
        title='3D Stock Price Surface',
        scene=dict(
            xaxis_title='Company',
            yaxis_title='Date',
            zaxis_title='Price ($)'
        ),
        height=600
    )
    
    return fig_3d

# Show 3D visualization
surface_fig = create_3d_surface()
surface_fig.show()

# Create animated time series
def create_animated_chart():
    # Prepare data for animation
    animation_data = []
    
    for date in df['date'].unique()[::7]:  # Weekly snapshots
        week_data = df[df['date'] <= date]
        latest_prices = week_data.groupby('company')['price'].last().reset_index()
        latest_prices['date'] = date
        animation_data.append(latest_prices)
    
    animation_df = pd.concat(animation_data, ignore_index=True)
    
    # Create animated bar chart
    animated_fig = px.bar(
        animation_df,
        x='company',
        y='price',
        animation_frame='date',
        title='Stock Price Evolution (Animated)',
        labels={'price': 'Price ($)', 'company': 'Company'},
        range_y=[0, animation_df['price'].max() * 1.1]
    )
    
    animated_fig.update_layout(height=400)
    
    return animated_fig

# Show animated chart
animated_fig = create_animated_chart()
animated_fig.show()
```

### 6. Export or Display

```python
# Create final comprehensive dashboard and export options

# Create master dashboard combining all visualizations
master_fig = make_subplots(
    rows=4, cols=2,
    subplot_titles=('Price Trends', 'Volume Analysis',
                   'Returns Distribution', 'Risk Metrics',
                   'Correlation Matrix', 'Portfolio Allocation',
                   'Technical Indicators', 'Performance Summary'),
    specs=[[{"secondary_y": True}, {"secondary_y": False}],
           [{"secondary_y": False}, {"secondary_y": False}],
           [{"secondary_y": False}, {"secondary_y": False}],
           [{"secondary_y": False}, {"secondary_y": False}]],
    vertical_spacing=0.06
)

# Add comprehensive visualizations
# (Previous subplot code would go here - abbreviated for space)

# Add final touches
master_fig.update_layout(
    title_text="Comprehensive Financial Analytics Dashboard",
    title_x=0.5,
    title_font_size=24,
    height=1200,
    showlegend=True
)

# Export options
# 1. Save as HTML (interactive)
master_fig.write_html("financial_dashboard.html")

# 2. Save as static image
master_fig.write_image("financial_dashboard.png", width=1200, height=1200)

# 3. Save as PDF
master_fig.write_image("financial_dashboard.pdf", width=1200, height=1200)

# 4. Save individual charts
candlestick_fig.write_html("stock_candlestick.html")
surface_fig.write_html("3d_surface.html")
animated_fig.write_html("animated_prices.html")

# Display summary
print("Financial Dashboard Export Complete!")
print("\nDataset Summary:")
print(f"Companies analyzed: {len(companies)}")
print(f"Date range: {df['date'].min().strftime('%Y-%m-%d')} to {df['date'].max().strftime('%Y-%m-%d')}")
print(f"Total data points: {len(df):,}")

# Calculate portfolio metrics
portfolio_returns = []
for company, weight in zip(companies, [0.3, 0.25, 0.2, 0.15, 0.1]):
    company_return = df[df['company'] == company]['daily_return'].mean() * weight
    portfolio_returns.append(company_return)

portfolio_daily_return = sum(portfolio_returns)
portfolio_annual_return = portfolio_daily_return * 252

print(f"\nPortfolio Metrics:")
print(f"Expected Annual Return: {portfolio_annual_return:.2%}")
print(f"Average Daily Volatility: {df.groupby('company')['volatility'].last().mean():.2%}")

print("\nFiles Generated:")
print("- financial_dashboard.html (interactive)")
print("- financial_dashboard.png")
print("- financial_dashboard.pdf")
print("- stock_candlestick.html")
print("- 3d_surface.html")
print("- animated_prices.html")

# Show the master dashboard
master_fig.show()
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Large datasets causing browser performance issues
  - Solution: Use data sampling, server-side processing, or Plotly Dash for large datasets
- **Pitfall 2**: Overlapping traces in complex charts
  - Solution: Use opacity, different line styles, or subplot organization
- **Pitfall 3**: Poor mobile responsiveness
  - Solution: Use responsive layout options and test on mobile devices

## Advanced Techniques

- Building real-time dashboards with Plotly Dash
- Creating custom JavaScript callbacks for complex interactions
- Implementing WebGL for high-performance rendering
- Using Plotly with streaming data sources
- Creating custom themes and templates
- Integrating with databases and APIs for dynamic data

## Resources

- [Plotly Documentation](https://plotly.com/python/)
- [Plotly Gallery](https://plotly.com/python/plotly-express/)
- [Dash Documentation](https://dash.plotly.com/)
- [Plotly Community Forum](https://community.plotly.com/)
