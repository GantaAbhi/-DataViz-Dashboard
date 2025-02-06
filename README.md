# -DataViz-Dashboard
An interactive dashboard to visualize and analyze trends in large datasets, such as sales, stock prices, or demographic data
import os
import pandas as pd
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output
import plotly.express as px
from flask import Flask

# Initialize Flask app
server = Flask(__name__)

# Initialize Dash app
app = dash.Dash(__name__, server=server, routes_pathname_prefix='/dashboard/')

# Sample dataset
DATA_PATH = "data/sample_data.csv"
if not os.path.exists(DATA_PATH):
    df = pd.DataFrame({
        'Date': pd.date_range(start='1/1/2023', periods=100),
        'Sales': [x*100 for x in range(100)],
        'Category': ['A', 'B', 'C', 'D'] * 25
    })
    df.to_csv(DATA_PATH, index=False)
else:
    df = pd.read_csv(DATA_PATH)

# Layout of dashboard
app.layout = html.Div([
    html.H1("DataViz Dashboard"),
    dcc.Dropdown(
        id='category-dropdown',
        options=[{'label': cat, 'value': cat} for cat in df['Category'].unique()],
        value='A'
    ),
    dcc.Graph(id='sales-trend')
])

# Callback to update graph
@app.callback(
    Output('sales-trend', 'figure'),
    Input('category-dropdown', 'value')
)
def update_graph(selected_category):
    filtered_df = df[df['Category'] == selected_category]
    fig = px.line(filtered_df, x='Date', y='Sales', title=f'Sales Trend for Category {selected_category}')
    return fig

if __name__ == '__main__':
    app.run_server(debug=True)
