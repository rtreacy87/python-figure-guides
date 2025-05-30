# Seaborn Guide

## Installation

```bash
pip install seaborn
# Seaborn automatically installs matplotlib, numpy, pandas, and scipy
pip install jupyter  # Optional, for notebook usage
```

## Overview

**Specialty:** Statistical visualizations and data exploration

**Best Use Cases:**
- Exploratory data analysis (EDA)
- Statistical relationship visualization
- Correlation matrices and heatmaps
- Distribution analysis and comparison
- Regression analysis visualization
- Publication-ready statistical plots

## ASCII Example

Below is an ASCII representation of what we'll create - a statistical analysis dashboard:

```
+---------------------------+---------------------------+
|                           |                           |
|    Distribution Plot      |    Correlation Matrix     |
|                           |                           |
|    ████                   |    [1.0][0.8][0.3][0.1]   |
|   ██████                  |    [0.8][1.0][0.5][0.2]   |
|  ████████                 |    [0.3][0.5][1.0][0.7]   |
| ██████████                |    [0.1][0.2][0.7][1.0]   |
+---------------------------+---------------------------+
|                           |                           |
|    Scatter Plot with      |    Box Plot Comparison    |
|    Regression Line        |                           |
|                           |      ┌─┬─┐                |
|        ●●●                |      │ │ │                |
|      ●●●●●                |    ┌─┼─┼─┼─┐              |
|    ●●●●●●●                |    │ │ │ │ │              |
|  ●●●●●●●●●                |    └─┴─┴─┴─┘              |
+---------------------------+---------------------------+
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
from scipy import stats

# Set the aesthetic style
sns.set_style("whitegrid")
sns.set_palette("husl")
```

### 2. Prepare Your Data

```python
# Generate sample dataset for analysis
np.random.seed(42)

# Create a dataset simulating customer behavior
n_customers = 1000

data = {
    'age': np.random.normal(35, 12, n_customers),
    'income': np.random.normal(50000, 15000, n_customers),
    'spending_score': np.random.normal(50, 25, n_customers),
    'years_customer': np.random.exponential(3, n_customers),
    'satisfaction': np.random.normal(7, 2, n_customers),
    'category': np.random.choice(['Premium', 'Standard', 'Basic'], n_customers, p=[0.2, 0.5, 0.3])
}

# Add some correlations to make it more realistic
data['spending_score'] = data['spending_score'] + 0.3 * (data['income'] - 50000) / 1000
data['satisfaction'] = data['satisfaction'] + 0.2 * data['spending_score'] / 10

# Create DataFrame
df = pd.DataFrame(data)

# Clean the data
df['age'] = np.clip(df['age'], 18, 80)
df['income'] = np.clip(df['income'], 20000, 150000)
df['spending_score'] = np.clip(df['spending_score'], 0, 100)
df['satisfaction'] = np.clip(df['satisfaction'], 1, 10)
df['years_customer'] = np.clip(df['years_customer'], 0, 20)

print(df.head())
print(f"Dataset shape: {df.shape}")
```

### 3. Create the Basic Visualization

```python
# Create a comprehensive statistical analysis dashboard
fig, axes = plt.subplots(2, 3, figsize=(18, 12))

# 1. Distribution plot
sns.histplot(data=df, x='age', hue='category', kde=True, ax=axes[0, 0])
axes[0, 0].set_title('Age Distribution by Customer Category')

# 2. Correlation heatmap
numeric_cols = ['age', 'income', 'spending_score', 'years_customer', 'satisfaction']
correlation_matrix = df[numeric_cols].corr()
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', center=0, ax=axes[0, 1])
axes[0, 1].set_title('Correlation Matrix')

# 3. Scatter plot with regression
sns.scatterplot(data=df, x='income', y='spending_score', hue='category', ax=axes[0, 2])
sns.regplot(data=df, x='income', y='spending_score', scatter=False, ax=axes[0, 2])
axes[0, 2].set_title('Income vs Spending Score')

# 4. Box plot comparison
sns.boxplot(data=df, x='category', y='satisfaction', ax=axes[1, 0])
axes[1, 0].set_title('Satisfaction by Customer Category')

# 5. Violin plot for distribution comparison
sns.violinplot(data=df, x='category', y='spending_score', ax=axes[1, 1])
axes[1, 1].set_title('Spending Score Distribution by Category')

# 6. Pair plot subset (using a separate figure due to complexity)
axes[1, 2].text(0.5, 0.5, 'See separate\nPair Plot', 
                ha='center', va='center', fontsize=14, 
                transform=axes[1, 2].transAxes)
axes[1, 2].set_title('Pair Plot Reference')
axes[1, 2].axis('off')

plt.tight_layout()
plt.show()

# Create the pair plot separately
plt.figure(figsize=(12, 10))
pair_plot = sns.pairplot(df[numeric_cols + ['category']], hue='category', diag_kind='kde')
pair_plot.fig.suptitle('Pairwise Relationships', y=1.02)
plt.show()
```

### 4. Customize the Appearance

```python
# Create a publication-ready statistical analysis
plt.style.use('seaborn-v0_8-paper')
sns.set_palette("Set2")

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# Enhanced correlation heatmap
mask = np.triu(np.ones_like(correlation_matrix, dtype=bool))
sns.heatmap(correlation_matrix, 
            mask=mask,
            annot=True, 
            cmap='RdBu_r', 
            center=0,
            square=True,
            fmt='.2f',
            cbar_kws={"shrink": .8},
            ax=axes[0, 0])
axes[0, 0].set_title('Customer Metrics Correlation Matrix', fontsize=14, fontweight='bold')

# Enhanced regression plot with confidence intervals
sns.regplot(data=df, x='years_customer', y='satisfaction', 
            scatter_kws={'alpha': 0.6}, ax=axes[0, 1])
axes[0, 1].set_title('Customer Loyalty vs Satisfaction', fontsize=14, fontweight='bold')
axes[0, 1].set_xlabel('Years as Customer')
axes[0, 1].set_ylabel('Satisfaction Score')

# Enhanced distribution comparison
sns.kdeplot(data=df, x='income', hue='category', fill=True, alpha=0.6, ax=axes[1, 0])
axes[1, 0].set_title('Income Distribution by Customer Category', fontsize=14, fontweight='bold')
axes[1, 0].set_xlabel('Annual Income ($)')

# Statistical summary plot
sns.barplot(data=df, x='category', y='spending_score', 
            capsize=0.1, errwidth=2, ax=axes[1, 1])
axes[1, 1].set_title('Average Spending Score by Category', fontsize=14, fontweight='bold')
axes[1, 1].set_ylabel('Spending Score')

plt.tight_layout()
plt.show()
```

### 5. Add Statistical Analysis

```python
# Perform statistical tests and create summary visualizations
from scipy.stats import f_oneway, pearsonr

# ANOVA test for spending score differences between categories
categories = df['category'].unique()
spending_by_category = [df[df['category'] == cat]['spending_score'] for cat in categories]
f_stat, p_value = f_oneway(*spending_by_category)

print(f"ANOVA Test Results:")
print(f"F-statistic: {f_stat:.4f}")
print(f"P-value: {p_value:.4f}")

# Correlation analysis
correlations = {}
for col in numeric_cols:
    if col != 'satisfaction':
        corr, p_val = pearsonr(df[col], df['satisfaction'])
        correlations[col] = {'correlation': corr, 'p_value': p_val}

# Create correlation significance plot
corr_df = pd.DataFrame(correlations).T
corr_df['significant'] = corr_df['p_value'] < 0.05

plt.figure(figsize=(10, 6))
colors = ['red' if sig else 'gray' for sig in corr_df['significant']]
bars = plt.bar(corr_df.index, corr_df['correlation'], color=colors, alpha=0.7)
plt.axhline(y=0, color='black', linestyle='-', alpha=0.3)
plt.title('Correlation with Customer Satisfaction\n(Red = Significant, Gray = Not Significant)')
plt.ylabel('Correlation Coefficient')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

# Print correlation results
print("\nCorrelation with Satisfaction:")
for var, stats in correlations.items():
    significance = "***" if stats['p_value'] < 0.001 else "**" if stats['p_value'] < 0.01 else "*" if stats['p_value'] < 0.05 else ""
    print(f"{var}: r = {stats['correlation']:.3f} {significance}")
```

### 6. Export or Display

```python
# Save high-quality plots for publication
plt.style.use('seaborn-v0_8-paper')

# Create final summary dashboard
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Key insights visualization
sns.scatterplot(data=df, x='income', y='spending_score', 
                hue='satisfaction', size='years_customer',
                sizes=(20, 200), alpha=0.7, ax=axes[0, 0])
axes[0, 0].set_title('Customer Segmentation Analysis')

sns.boxplot(data=df, x='category', y='satisfaction', ax=axes[0, 1])
axes[0, 1].set_title('Satisfaction by Customer Tier')

sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', 
            center=0, ax=axes[1, 0])
axes[1, 0].set_title('Feature Correlations')

# Summary statistics
summary_stats = df.groupby('category')[numeric_cols].mean()
sns.heatmap(summary_stats.T, annot=True, cmap='viridis', ax=axes[1, 1])
axes[1, 1].set_title('Average Metrics by Category')

plt.tight_layout()
plt.savefig('customer_analysis_dashboard.png', dpi=300, bbox_inches='tight')
plt.savefig('customer_analysis_dashboard.pdf', bbox_inches='tight')
plt.show()

print("Analysis complete! Files saved:")
print("- customer_analysis_dashboard.png")
print("- customer_analysis_dashboard.pdf")
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Overlapping labels in categorical plots
  - Solution: Rotate labels with `plt.xticks(rotation=45)` or use `sns.despine()`
- **Pitfall 2**: Poor color choices for colorblind accessibility
  - Solution: Use colorblind-friendly palettes like `sns.set_palette("colorblind")`
- **Pitfall 3**: Overplotting in scatter plots with large datasets
  - Solution: Use `alpha` parameter, sampling, or `sns.scatterplot()` with `s` parameter

## Advanced Techniques

- Creating custom color palettes for brand consistency
- Using FacetGrid for complex multi-panel visualizations
- Implementing statistical annotations with `statannotations` package
- Creating interactive plots by combining with Plotly
- Building custom plot functions for repeated analysis patterns
- Integrating with statistical modeling libraries (statsmodels, scikit-learn)

## Resources

- [Seaborn Documentation](https://seaborn.pydata.org/)
- [Seaborn Tutorial](https://seaborn.pydata.org/tutorial.html)
- [Statistical Visualization Gallery](https://seaborn.pydata.org/examples/index.html)
- [Python for Data Analysis Book](https://wesmckinney.com/book/)
