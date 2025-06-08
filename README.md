# micro_IT2
import pandas as pd

df = pd.read_csv('internship.csv')
display(df.head())
display(df.info())
display(df.columns)
display(df.dtypes)
display(df['internship_title'].nunique())
display(df['location'].nunique())
display(df.head())
import numpy as np

# 1. Clean 'stipend' column
def clean_stipend(stipend_str):
    if pd.isna(stipend_str):
        return 0
    stipend_str = stipend_str.lower().replace('₹', '').replace(',', '').strip()
    if 'unpaid' in stipend_str or 'performance' in stipend_str:
        return 0
    if '-' in stipend_str:
        try:
            low, high = map(float, stipend_str.split('-')[0].split()[0].split('/')[0].split('to'))
            return (low + high) / 2
        except (ValueError, IndexError):
            return 0
    try:
        return float(stipend_str.split()[0].split('/')[0])
    except (ValueError, IndexError):
        return 0

df['cleaned_stipend'] = df['stipend'].apply(clean_stipend)

# 2. Clean 'duration' column
def clean_duration(duration_str):
    if pd.isna(duration_str):
        return np.nan
    duration_str = duration_str.lower().strip()
    if 'month' in duration_str:
        try:
            return float(duration_str.split()[0]) * 4 # Convert months to weeks (approx)
        except (ValueError, IndexError):
            return np.nan
    elif 'week' in duration_str:
        try:
            return float(duration_str.split()[0])
        except (ValueError, IndexError):
            return np.nan
    else:
        return np.nan

df['cleaned_duration_weeks'] = df['duration'].apply(clean_duration)
df['domain'] = df['internship_title']
domain_engagement = df['domain'].value_counts().reset_index()
domain_engagement.columns = ['domain', 'engagement_count']
df = pd.merge(df, domain_engagement, on='domain', how='left')

display(df[['stipend', 'cleaned_stipend', 'duration', 'cleaned_duration_weeks', 'domain', 'engagement_count']].head())
display(df.info())
domain_engagement_summary = df.groupby('domain')['engagement_count'].sum().reset_index()
domain_engagement_summary = domain_engagement_summary.sort_values(by='engagement_count', ascending=False)
display(domain_engagement_summary.head())
