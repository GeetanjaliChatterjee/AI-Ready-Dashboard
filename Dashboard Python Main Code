"""
Complete Enhanced Dash Application - Project Feasibility Dashboard
"""
import dash
from dash import dcc, html, Input, Output, State, callback, dash_table, no_update
import dash_bootstrap_components as dbc
import pandas as pd
import base64
import io

# Initialize Dash app
app = dash.Dash(
    __name__,
    external_stylesheets=[
        dbc.themes.BOOTSTRAP,
        "https://cdn.jsdelivr.net/npm/bootstrap-icons@1.7.2/font/bootstrap-icons.css"
    ],
    suppress_callback_exceptions=True,
    title="Project Feasibility Dashboard"
)

def create_data_preview_component(df, filename):
    """Create a data preview component"""
    return dbc.Card([
        dbc.CardHeader([
            html.H5([
                html.I(className="bi bi-table me-2"),
                "Data Preview"
            ], className="mb-0")
        ]),
        dbc.CardBody([
            dash_table.DataTable(
                data=df.head(5).to_dict('records'),
                columns=[{"name": i, "id": i} for i in df.columns],
                style_table={'overflowX': 'auto'},
                style_cell={'textAlign': 'left', 'padding': '8px'},
                style_header={'backgroundColor': '#e9ecef', 'fontWeight': 'bold'}
            ),
            html.Hr(),
            dbc.Button(
                [html.I(className="bi bi-arrow-right me-2"), "Proceed to Analysis"],
                id="proceed-to-analysis",
                color="success",
                className="w-100"
            )
        ])
    ])

def get_upload_tab(stored_data=None, data_info=None):
    """Create the data upload tab content"""
    base_content = [
        dbc.Card([
            dbc.CardHeader([
                html.H4("📊 Upload Project Data", className="mb-0")
            ]),
            dbc.CardBody([
                html.P(
                    "Upload your project data file (CSV format) to begin the feasibility analysis.",
                    className="text-muted mb-4"
                ),
                
                dcc.Upload(
                    id='upload-data',
                    children=html.Div([
                        html.I(className="bi bi-cloud-upload", style={'fontSize': '2rem', 'color': '#007bff'}),
                        html.Br(),
                        html.Br(),
                        html.H5("Click to Upload CSV File", className="text-primary"),
                        html.P("Drag and drop or click to select", className="text-muted")
                    ], className="text-center"),
                    style={
                        'width': '100%',
                        'height': '120px',
                        'lineHeight': '30px',
                        'borderWidth': '2px',
                        'borderStyle': 'dashed',
                        'borderRadius': '10px',
                        'borderColor': '#007bff',
                        'textAlign': 'center',
                        'backgroundColor': '#f8f9fa',
                        'cursor': 'pointer',
                        'padding': '20px'
                    },
                    multiple=False
                ),
                
                html.Div(id='upload-status', className="mt-3"),
                html.Div(id='data-preview', className="mt-3")
            ])
        ])
    ]
    
    # Add upload status and preview if data exists
    if stored_data and data_info:
        df = pd.DataFrame(stored_data)
        
        # Add success status
        status = dbc.Alert([
            html.I(className="bi bi-check-circle me-2"),
            f"✅ Successfully uploaded {data_info['filename']} ({len(df)} rows, {len(df.columns)} columns)"
        ], color="success")
        
        # Add data preview
        preview = create_data_preview_component(df, data_info['filename'])
        
        # Add them to the content
        base_content.extend([status, preview])
    
    return base_content

def get_analysis_config(df):
    """Create analysis configuration components"""
    return dbc.Card([
        dbc.CardHeader([
            html.H5("🔧 Data Analysis Configuration", className="mb-0")
        ]),
        dbc.CardBody([
            html.P("Configure your analysis parameters:", className="text-muted"),
            
            dbc.Row([
                dbc.Col([
                    dbc.Label("Target Column (what to predict):"),
                    dcc.Dropdown(
                        id="target-column",
                        options=[{"label": col, "value": col} for col in df.columns],
                        placeholder="Select target variable...",
                        className="mb-3"
                    )
                ], md=6),
                
                dbc.Col([
                    dbc.Label("Analysis Type:"),
                    dcc.Dropdown(
                        id="analysis-type",
                        options=[
                            {"label": "Feasibility Score", "value": "feasibility"},
                            {"label": "Classification Analysis", "value": "classification"},
                            {"label": "Regression Analysis", "value": "regression"}
                        ],
                        value="feasibility",
                        className="mb-3"
                    )
                ], md=6)
            ]),
            
            dbc.Button(
                [html.I(className="bi bi-play-circle me-2"), "Run Analysis"],
                id="run-analysis",
                color="primary",
                size="lg",
                className="w-100 mb-3"
            )
        ])
    ])

def generate_feasibility_assessment(df, target_column, analysis_type):
    """Generate an analysis-type specific feasibility assessment"""
    target_values = df[target_column]
    unique_values = len(target_values.unique())
    total_rows = len(df)
    missing_values = target_values.isnull().sum()
    data_quality = (total_rows - missing_values) / total_rows * 100
    feature_count = len(df.columns) - 1  # Exclude target
    
    assessments = []
    
    # Data Quality Score (same for all types but affects score differently)
    if data_quality >= 95:
        assessments.append("✅ Excellent data quality (95%+ complete)")
        quality_score = 1.0
    elif data_quality >= 80:
        assessments.append("⚠️ Good data quality (80-95% complete)")
        quality_score = 0.7
    else:
        assessments.append("❌ Poor data quality (<80% complete)")
        quality_score = 0.3
    
    # ANALYSIS-TYPE SPECIFIC SCORING
    if analysis_type == "classification":
        # Classification-specific assessments
        target_score = 0
        sample_score = 0
        feature_score = 0
        
        # Target Variable Assessment for Classification
        if 2 <= unique_values <= 20:
            assessments.append("✅ Optimal number of classes for classification")
            target_score = 1.0
        elif unique_values > 50:
            assessments.append("⚠️ Too many classes - consider grouping or regression")
            target_score = 0.4
        elif unique_values == 1:
            assessments.append("❌ Only one class - cannot perform classification")
            target_score = 0.0
        else:
            assessments.append("⚠️ Very few classes - check data distribution")
            target_score = 0.6
        
        # Check class balance for classification
        value_counts = target_values.value_counts()
        class_balance = value_counts.min() / value_counts.max()
        if class_balance >= 0.1:  # Not severely imbalanced
            assessments.append("✅ Reasonably balanced classes")
            target_score = min(target_score + 0.2, 1.0)
        else:
            assessments.append("⚠️ Imbalanced classes - may need special handling")
            target_score = max(target_score - 0.2, 0.0)
        
        # Sample Size for Classification (needs more data per class)
        min_per_class = total_rows / unique_values
        if total_rows >= 1000 and min_per_class >= 50:
            assessments.append("✅ Excellent sample size for classification")
            sample_score = 1.0
        elif total_rows >= 200 and min_per_class >= 20:
            assessments.append("⚠️ Adequate sample size for classification")
            sample_score = 0.6
        else:
            assessments.append("❌ Small sample size for classification")
            sample_score = 0.2
        
        # Feature Assessment for Classification
        if feature_count >= 3:
            assessments.append("✅ Good number of features for classification")
            feature_score = 1.0
        elif feature_count >= 1:
            assessments.append("⚠️ Limited features - may affect classification accuracy")
            feature_score = 0.5
        else:
            assessments.append("❌ No features available for classification")
            feature_score = 0.0
            
    elif analysis_type == "regression":
        # Regression-specific assessments
        target_score = 0
        sample_score = 0
        feature_score = 0
        
        # Target Variable Assessment for Regression
        if target_values.dtype in ['int64', 'float64']:
            if unique_values >= 10:
                assessments.append("✅ Good target variable distribution for regression")
                target_score = 1.0
            elif unique_values >= 5:
                assessments.append("⚠️ Limited target variation - check if regression is appropriate")
                target_score = 0.6
            else:
                assessments.append("❌ Very few unique values - consider classification instead")
                target_score = 0.2
        else:
            assessments.append("❌ Non-numeric target - not suitable for regression")
            target_score = 0.0
            
        # Check target distribution (avoid extreme skewness)
        if target_values.dtype in ['int64', 'float64'] and len(target_values.dropna()) > 0:
            try:
                skewness = abs(target_values.skew())
                if skewness < 2:
                    assessments.append("✅ Well-distributed target variable")
                    target_score = min(target_score + 0.1, 1.0)
                else:
                    assessments.append("⚠️ Highly skewed target - may need transformation")
                    target_score = max(target_score - 0.1, 0.0)
            except:
                pass
        
        # Sample Size for Regression
        if total_rows >= 500:
            assessments.append("✅ Excellent sample size for regression")
            sample_score = 1.0
        elif total_rows >= 100:
            assessments.append("⚠️ Adequate sample size for regression")
            sample_score = 0.7
        else:
            assessments.append("❌ Small sample size for regression")
            sample_score = 0.3
        
        # Feature Assessment for Regression (needs more features typically)
        if feature_count >= 5:
            assessments.append("✅ Good number of features for regression")
            feature_score = 1.0
        elif feature_count >= 2:
            assessments.append("⚠️ Limited features - may need feature engineering")
            feature_score = 0.6
        else:
            assessments.append("❌ Very few features for regression")
            feature_score = 0.2
            
    else:  # feasibility analysis - general scoring
        target_score = 0
        sample_score = 0
        feature_score = 0
        
        # General Target Assessment
        if unique_values > 1:
            assessments.append("✅ Target variable has variation for analysis")
            target_score = 0.8
        else:
            assessments.append("❌ Target variable has no variation")
            target_score = 0.0
        
        # General Sample Size Assessment
        if total_rows >= 1000:
            assessments.append("✅ Excellent sample size for analysis")
            sample_score = 1.0
        elif total_rows >= 100:
            assessments.append("⚠️ Adequate sample size for analysis")
            sample_score = 0.6
        else:
            assessments.append("❌ Small sample size for analysis")
            sample_score = 0.3
        
        # General Feature Assessment
        if feature_count >= 5:
            assessments.append("✅ Good number of features for analysis")
            feature_score = 1.0
        elif feature_count >= 2:
            assessments.append("⚠️ Limited features available")
            feature_score = 0.5
        else:
            assessments.append("❌ Very few features available")
            feature_score = 0.2
    
    # Calculate weighted feasibility score based on analysis type
    if analysis_type == "classification":
        # For classification: target variable characteristics are most important
        feasibility_score = (quality_score * 0.2 + target_score * 0.4 + sample_score * 0.3 + feature_score * 0.1) * 100
    elif analysis_type == "regression":
        # For regression: features and sample size are more important
        feasibility_score = (quality_score * 0.2 + target_score * 0.3 + sample_score * 0.3 + feature_score * 0.2) * 100
    else:  # feasibility
        # Balanced scoring for general feasibility
        feasibility_score = (quality_score * 0.25 + target_score * 0.25 + sample_score * 0.25 + feature_score * 0.25) * 100
    
    # Overall Feasibility based on analysis-specific thresholds
    if analysis_type == "classification":
        if feasibility_score >= 70:
            overall = "🟢 HIGH FEASIBILITY - Classification project looks very promising!"
        elif feasibility_score >= 45:
            overall = "🟡 MEDIUM FEASIBILITY - Classification possible with improvements"
        else:
            overall = "🔴 LOW FEASIBILITY - Significant challenges for classification"
    elif analysis_type == "regression":
        if feasibility_score >= 65:
            overall = "🟢 HIGH FEASIBILITY - Regression analysis looks very promising!"
        elif feasibility_score >= 40:
            overall = "🟡 MEDIUM FEASIBILITY - Regression possible with improvements"
        else:
            overall = "🔴 LOW FEASIBILITY - Significant challenges for regression"
    else:
        if feasibility_score >= 75:
            overall = "🟢 HIGH FEASIBILITY - Project looks very promising!"
        elif feasibility_score >= 50:
            overall = "🟡 MEDIUM FEASIBILITY - Project has potential with improvements"
        else:
            overall = "🔴 LOW FEASIBILITY - Significant challenges identified"
    
    return assessments, overall, feasibility_score

# Create the main layout
app.layout = dbc.Container([
    # Hidden data storage components
    dcc.Store(id='stored-data'),
    dcc.Store(id='data-info'),
    dcc.Store(id='analysis-results-store'),
    
    # Header
    dbc.Row([
        dbc.Col([
            html.H1(
                "🚀 Project Feasibility Dashboard",
                className="text-center mb-4 text-primary"
            ),
            html.P(
                "AI-driven project automation feasibility analysis",
                className="text-center text-muted mb-4"
            ),
            html.Hr()
        ])
    ]),
    
    # Navigation tabs
    dbc.Row([
        dbc.Col([
            dbc.Tabs(
                id="main-tabs",
                active_tab="upload",
                children=[
                    dbc.Tab(label="📊 Data Upload", tab_id="upload"),
                    dbc.Tab(label="🔧 Analysis", tab_id="analysis"),
                    dbc.Tab(label="📈 Results", tab_id="results"),
                    dbc.Tab(label="💡 Recommendations", tab_id="recommendations"),
                ],
                className="mb-4"
            )
        ])
    ]),
    
    # Main content area
    dbc.Row([
        dbc.Col([
            # Upload tab content
            html.Div(
                id="upload-tab-content",
                style={"display": "block"}
            ),
            
            # Analysis tab content - PERSISTENT
            html.Div([
                # Data status
                html.Div(id="analysis-data-status"),
                
                # Analysis configuration
                html.Div(id="analysis-config"),
                
                # PERSISTENT analysis areas
                html.Div(id="analysis-status-persistent", style={"margin": "20px 0"}),
                html.Div(id="analysis-results-persistent", style={"margin": "20px 0"}),
            ], id="analysis-tab-content", style={"display": "none"}),
            
            # Results tab content
            html.Div([
                html.H4("Results will appear here after analysis")
            ], id="results-tab-content", style={"display": "none"}),
            
            # Recommendations tab content
            html.Div([
                html.H4("Recommendations will appear here after analysis")
            ], id="recommendations-tab-content", style={"display": "none"})
        ])
    ])
    
], fluid=True, className="px-4")

# CALLBACKS

# Tab visibility callback
@app.callback(
    [Output("upload-tab-content", "style"),
     Output("analysis-tab-content", "style"),
     Output("results-tab-content", "style"),
     Output("recommendations-tab-content", "style")],
    [Input("main-tabs", "active_tab")]
)
def show_tab_content(active_tab):
    """Show/hide tab content based on active tab"""
    upload_style = {"display": "block" if active_tab == "upload" else "none"}
    analysis_style = {"display": "block" if active_tab == "analysis" else "none"}
    results_style = {"display": "block" if active_tab == "results" else "none"}
    recommendations_style = {"display": "block" if active_tab == "recommendations" else "none"}
    
    return upload_style, analysis_style, results_style, recommendations_style

# Data upload and processing
@app.callback(
    [Output('stored-data', 'data'),
     Output('data-info', 'data')],
    Input('upload-data', 'contents'),
    State('upload-data', 'filename'),
    prevent_initial_call=True
)
def process_uploaded_file(contents, filename):
    """Process uploaded file and store data"""
    if contents is None:
        return None, None
    
    try:
        content_type, content_string = contents.split(',')
        decoded = base64.b64decode(content_string)
        
        if filename and filename.endswith('.csv'):
            df = pd.read_csv(io.StringIO(decoded.decode('utf-8')))
            
            data_info = {
                'filename': filename,
                'rows': len(df),
                'columns': len(df.columns),
                'column_names': list(df.columns)
            }
            
            stored_data = df.to_dict('records')
            return stored_data, data_info
        else:
            return None, None
            
    except Exception as e:
        print(f"Error processing file: {e}")
        return None, None

# Upload tab content updates
@app.callback(
    Output('upload-tab-content', 'children'),
    [Input('stored-data', 'data'),
     Input('data-info', 'data')],
    prevent_initial_call=False
)
def update_upload_tab(stored_data, data_info):
    """Update upload tab content"""
    return get_upload_tab(stored_data, data_info)

# Analysis tab status and config
@app.callback(
    [Output('analysis-data-status', 'children'),
     Output('analysis-config', 'children')],
    [Input('stored-data', 'data'),
     Input('data-info', 'data')],
    prevent_initial_call=False
)
def update_analysis_tab(stored_data, data_info):
    """Update analysis tab with data status and configuration"""
    if not stored_data or not data_info:
        status = dbc.Alert([
            html.I(className="bi bi-info-circle me-2"),
            "Upload data in the Data Upload tab to enable analysis."
        ], color="info")
        return status, ""
    
    df = pd.DataFrame(stored_data)
    
    # Data status
    status = dbc.Alert([
        html.I(className="bi bi-check-circle me-2"),
        f"Data loaded successfully! Ready to analyze {len(df)} records with {len(df.columns)} columns."
    ], color="success")
    
    # Configuration
    config = get_analysis_config(df)
    
    return status, config

# Navigate to analysis tab
@app.callback(
    Output("main-tabs", "active_tab"),
    Input("proceed-to-analysis", "n_clicks"),
    prevent_initial_call=True
)
def proceed_to_analysis(n_clicks):
    """Navigate to analysis tab when button is clicked"""
    if n_clicks and n_clicks > 0:
        return "analysis"
    return no_update

# ENHANCED ANALYSIS CALLBACK - Now updates Results tab too
@app.callback(
    [Output("analysis-status-persistent", "children"),
     Output("analysis-results-persistent", "children"),
     Output("results-tab-content", "children"),
     Output("analysis-results-store", "data")],
    [Input("run-analysis", "n_clicks")],
    [State("stored-data", "data"),
     State("target-column", "value"),
     State("analysis-type", "value")],
    prevent_initial_call=True
)
def run_analysis_enhanced(n_clicks, stored_data, target_column, analysis_type):
    """Enhanced analysis callback with detailed insights"""
    print(f"ENHANCED: run_analysis called with n_clicks={n_clicks}")
    print(f"ENHANCED: target_column={target_column}, analysis_type={analysis_type}")
    
    if not n_clicks or n_clicks == 0:
        print("ENHANCED: No button clicks")
        return "", "", html.H4("Results will appear here after analysis"), None
        
    if not stored_data:
        print("ENHANCED: No data")
        error = dbc.Alert("No data available!", color="danger")
        return error, "", error, None
        
    if not target_column:
        print("ENHANCED: No target column")
        error = dbc.Alert("Please select a target column!", color="warning")
        return error, "", error, None
    
    print("ENHANCED: Creating detailed results...")
    
    df = pd.DataFrame(stored_data)
    
    # Analyze the target column
    target_values = df[target_column]
    unique_values = target_values.unique()
    value_counts = target_values.value_counts()
    
    # Generate feasibility assessment
    assessments, overall_feasibility, feasibility_score = generate_feasibility_assessment(df, target_column, analysis_type)
    
    # Create enhanced status
    status = dbc.Alert([
        html.I(className="bi bi-check-circle me-2"),
        f"✅ Analysis completed! Feasibility Score: {feasibility_score:.0f}%"
    ], color="success")
    
    # Create detailed results based on target column
    results_card = dbc.Card([
        dbc.CardHeader([
            html.H4("📊 Enhanced Analysis Results", className="mb-0 text-primary")
        ]),
        dbc.CardBody([
            # Feasibility Score Header
            dbc.Alert([
                html.H5(overall_feasibility, className="mb-0")
            ], color="info" if feasibility_score >= 50 else "warning"),
            
            # Basic info
            dbc.Row([
                dbc.Col([
                    html.H5(f"🎯 Target: {target_column}", className="text-primary"),
                    html.H5(f"📈 Type: {analysis_type}", className="text-info"),
                    html.H5(f"📋 Dataset: {df.shape[0]} rows × {df.shape[1]} columns", className="text-success"),
                ], md=6),
                dbc.Col([
                    html.H6("Target Column Analysis:", className="text-dark"),
                    html.P(f"• Unique values: {len(unique_values)}"),
                    html.P(f"• Data type: {target_values.dtype}"),
                    html.P(f"• Missing values: {target_values.isnull().sum()}"),
                    html.P(f"• Complete data: {((len(df) - target_values.isnull().sum()) / len(df) * 100):.1f}%"),
                ], md=6)
            ]),
            
            html.Hr(),
            
            # Feasibility Assessment
            dbc.Row([
                dbc.Col([
                    html.H6("🤖 Feasibility Assessment:", className="text-success"),
                    html.Ul([
                        html.Li(assessment) for assessment in assessments
                    ]),
                    dbc.Progress(
                        value=feasibility_score,
                        label=f"{feasibility_score:.0f}%",
                        color="success" if feasibility_score >= 75 else "warning" if feasibility_score >= 50 else "danger",
                        className="mb-3"
                    )
                ], md=12)
            ]),
            
            html.Hr(),
            
            # Target column distribution and sample data
            dbc.Row([
                dbc.Col([
                    html.H6(f"Distribution of '{target_column}':", className="text-primary"),
                    dash_table.DataTable(
                        data=[
                            {"Value": str(val), "Count": count, "Percentage": f"{(count/len(df)*100):.1f}%"}
                            for val, count in value_counts.head(10).items()
                        ],
                        columns=[
                            {"name": "Value", "id": "Value"},
                            {"name": "Count", "id": "Count"},
                            {"name": "Percentage", "id": "Percentage"}
                        ],
                        style_table={'overflowY': 'auto', 'maxHeight': '200px'},
                        style_cell={'textAlign': 'left', 'padding': '8px', 'fontSize': '12px'},
                        style_header={'backgroundColor': '#e9ecef', 'fontWeight': 'bold'}
                    )
                ], md=6),
                dbc.Col([
                    html.H6("Sample Data (Target Highlighted):", className="text-primary"),
                    dash_table.DataTable(
                        data=df.sample(min(5, len(df))).to_dict('records'),
                        columns=[{"name": i, "id": i} for i in df.columns],
                        style_table={'overflowX': 'auto'},
                        style_cell={'textAlign': 'left', 'padding': '6px', 'fontSize': '11px'},
                        style_header={'backgroundColor': '#e9ecef', 'fontWeight': 'bold'},
                        # Highlight the target column
                        style_data_conditional=[
                            {
                                'if': {'column_id': target_column},
                                'backgroundColor': '#fff3cd',
                                'color': 'black',
                                'fontWeight': 'bold'
                            }
                        ]
                    )
                ], md=6)
            ])
        ])
    ])
    
    # Create Results tab content (cleaner version for Results tab)
    results_tab_content = [
        dbc.Row([
            dbc.Col([
                html.H3("📈 Analysis Results", className="text-primary mb-4"),
                
                # Feasibility Score Summary
                dbc.Card([
                    dbc.CardHeader([
                        html.H5("🎯 Feasibility Summary", className="mb-0")
                    ]),
                    dbc.CardBody([
                        dbc.Alert([
                            html.H4(overall_feasibility, className="mb-2"),
                            html.H5(f"Overall Score: {feasibility_score:.0f}%", className="mb-0")
                        ], color="success" if feasibility_score >= 75 else "warning" if feasibility_score >= 50 else "danger"),
                        
                        dbc.Progress(
                            value=feasibility_score,
                            label=f"Feasibility: {feasibility_score:.0f}%",
                            color="success" if feasibility_score >= 75 else "warning" if feasibility_score >= 50 else "danger",
                            className="mb-3",
                            style={"height": "30px", "fontSize": "16px"}
                        ),
                        
                        html.H6("Key Findings:", className="text-primary"),
                        html.Ul([
                            html.Li(assessment) for assessment in assessments
                        ])
                    ])
                ], className="mb-4"),
                
                # Data Summary
                dbc.Card([
                    dbc.CardHeader([
                        html.H5("📊 Data Summary", className="mb-0")
                    ]),
                    dbc.CardBody([
                        dbc.Row([
                            dbc.Col([
                                html.H6("Dataset Information:", className="text-info"),
                                html.P(f"• Total Records: {df.shape[0]:,}"),
                                html.P(f"• Total Features: {df.shape[1]}"),
                                html.P(f"• Target Variable: {target_column}"),
                                html.P(f"• Analysis Type: {analysis_type}"),
                            ], md=6),
                            dbc.Col([
                                html.H6("Target Variable Stats:", className="text-info"),
                                html.P(f"• Unique Values: {len(unique_values)}"),
                                html.P(f"• Data Type: {target_values.dtype}"),
                                html.P(f"• Missing Values: {target_values.isnull().sum()}"),
                                html.P(f"• Completeness: {((len(df) - target_values.isnull().sum()) / len(df) * 100):.1f}%"),
                            ], md=6)
                        ])
                    ])
                ])
            ])
        ])
    ]
    
    # Store results for other tabs to use
    analysis_results = {
        'feasibility_score': feasibility_score,
        'overall_assessment': overall_feasibility,
        'assessments': assessments,
        'target_column': target_column,
        'analysis_type': analysis_type,
        'dataset_info': {
            'rows': len(df),
            'columns': len(df.columns),
            'target_unique_values': len(unique_values),
            'target_missing': target_values.isnull().sum()
        }
    }
    
    print("ENHANCED: Returning enhanced results...")
    return status, results_card, results_tab_content, analysis_results

# Recommendations tab callback
@app.callback(
    Output("recommendations-tab-content", "children"),
    Input("analysis-results-store", "data"),
    prevent_initial_call=True
)
def update_recommendations_tab(analysis_results):
    """Update recommendations tab based on analysis results"""
    if not analysis_results:
        return html.H4("Recommendations will appear here after analysis")
    
    feasibility_score = analysis_results['feasibility_score']
    assessments = analysis_results['assessments']
    target_column = analysis_results['target_column']
    analysis_type = analysis_results['analysis_type']
    dataset_info = analysis_results['dataset_info']
    
    # Generate recommendations based on feasibility score and assessments
    recommendations = []
    
    if feasibility_score >= 75:
        recommendations.extend([
            "🚀 **Proceed with confidence** - Your project shows high feasibility!",
            "📈 Consider implementing a pilot program to validate assumptions",
            "🔧 Focus on model selection and hyperparameter tuning",
            "📊 Plan for model monitoring and maintenance in production"
        ])
    elif feasibility_score >= 50:
        recommendations.extend([
            "⚠️ **Proceed with caution** - Address key issues before implementation",
            "🔧 Focus on improving data quality and completeness",
            "📈 Consider collecting additional features or data points",
            "🧪 Run extensive testing before production deployment"
        ])
    else:
        recommendations.extend([
            "🛑 **Address critical issues** before proceeding",
            "📊 Improve data collection and quality processes",
            "🔍 Consider if this problem is suitable for ML/AI solutions",
            "💡 Explore alternative approaches or simpler rule-based systems"
        ])
    
    # Specific recommendations based on assessment findings
    for assessment in assessments:
        if "Poor data quality" in assessment:
            recommendations.append("🧹 **Data Cleaning Priority**: Focus on improving data completeness and handling missing values")
        elif "Small sample size" in assessment:
            recommendations.append("📈 **Data Collection**: Gather more data before training complex models")
        elif "Too many classes" in assessment:
            recommendations.append("🎯 **Class Consolidation**: Consider grouping similar classes together")
        elif "Very few features" in assessment:
            recommendations.append("🔍 **Feature Engineering**: Create additional relevant features from existing data")
    
    # Technical recommendations based on analysis type
    if analysis_type == "classification":
        recommendations.extend([
            "🤖 **Model Suggestions**: Try Random Forest, XGBoost, or Neural Networks",
            "⚖️ **Handle Class Imbalance**: Use techniques like SMOTE or class weighting if needed",
            "📏 **Evaluation Metrics**: Focus on precision, recall, and F1-score beyond just accuracy"
        ])
    elif analysis_type == "regression":
        recommendations.extend([
            "📈 **Model Suggestions**: Consider Linear Regression, Random Forest, or Gradient Boosting",
            "📊 **Feature Scaling**: Normalize or standardize features for better performance",
            "📏 **Evaluation Metrics**: Use MAE, RMSE, and R² to evaluate model performance"
        ])
    
    # Next steps recommendations
    next_steps = [
        "1. **Data Preparation**: Clean and preprocess your data",
        "2. **Feature Selection**: Identify the most relevant features",
        "3. **Model Training**: Start with simple models before trying complex ones",
        "4. **Validation**: Use cross-validation to ensure robust results",
        "5. **Testing**: Test thoroughly with unseen data",
        "6. **Deployment**: Plan for gradual rollout and monitoring"
    ]
    
    return [
        dbc.Row([
            dbc.Col([
                html.H3("💡 Recommendations & Next Steps", className="text-primary mb-4"),
                
                # Executive Summary
                dbc.Card([
                    dbc.CardHeader([
                        html.H5("📋 Executive Summary", className="mb-0")
                    ]),
                    dbc.CardBody([
                        dbc.Alert([
                            html.H5(f"Feasibility Score: {feasibility_score:.0f}%", className="mb-2"),
                            html.P(analysis_results['overall_assessment'], className="mb-0")
                        ], color="success" if feasibility_score >= 75 else "warning" if feasibility_score >= 50 else "danger")
                    ])
                ], className="mb-4"),
                
                # Specific Recommendations
                dbc.Card([
                    dbc.CardHeader([
                        html.H5("🎯 Specific Recommendations", className="mb-0")
                    ]),
                    dbc.CardBody([
                        html.Div([
                            dcc.Markdown(rec) for rec in recommendations
                        ])
                    ])
                ], className="mb-4"),
                
                # Next Steps
                dbc.Card([
                    dbc.CardHeader([
                        html.H5("🚀 Next Steps", className="mb-0")
                    ]),
                    dbc.CardBody([
                        html.Ol([
                            dcc.Markdown(step.split('. ', 1)[1]) for step in next_steps
                        ])
                    ])
                ], className="mb-4"),
                
                # Risk Assessment
                dbc.Card([
                    dbc.CardHeader([
                        html.H5("⚠️ Risk Assessment", className="mb-0")
                    ]),
                    dbc.CardBody([
                        html.H6("Potential Risks:", className="text-warning"),
                        html.Ul([
                            html.Li("Model may not generalize well to new data"),
                            html.Li("Data quality issues could affect performance"),
                            html.Li("Business requirements may change during development"),
                            html.Li("Technical complexity may exceed initial estimates")
                        ]),
                        html.H6("Mitigation Strategies:", className="text-success"),
                        html.Ul([
                            html.Li("Implement robust validation and testing procedures"),
                            html.Li("Maintain high data quality standards"),
                            html.Li("Use agile development with regular stakeholder feedback"),
                            html.Li("Start with simpler solutions and iterate")
                        ])
                    ])
                ])
            ])
        ])
    ]

if __name__ == "__main__":
    print("Starting Enhanced Project Feasibility Dashboard at http://localhost:8050")
  import os
port = int(os.environ.get("PORT", 8050))
app.run(debug=False, host='0.0.0.0', port=port)
