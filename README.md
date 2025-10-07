# Data Warehouse Lab 1

Hey there! 👋 This is Data Warehouse Lab 1 where we're building a cool stock price prediction system. 

Think of it as a smart assistant that watches Google and Microsoft stocks, learns from their past behavior, and tries to predict what might happen next week. Pretty neat, right?

## What We're Building

We're creating an automated system that:
- Grabs stock data from the internet (Google & Microsoft)
- Stores it safely in Snowflake (our data warehouse)
- Uses machine learning to predict future prices
- Runs everything automatically with Airflow

## System Architecture

```mermaid
graph TB
    %% External Data Sources
    YF[Yahoo Finance API<br/>yfinance]
    
    %% Apache Airflow Orchestration
    AF[Apache Airflow<br/>Scheduler & Orchestrator]
    
    %% ETL Pipeline
    ET[ETL Pipeline<br/>Daily Schedule]
    FETCH[Fetch Stock Data<br/>GOOGL, MSFT<br/>180 days history]
    LOAD[Load to Snowflake<br/>Raw Data Storage]
    
    %% Snowflake Data Warehouse
    SF[Snowflake Data Warehouse]
    RAW[Raw Data Table<br/>dev.raw.stock_data<br/>Symbol, Date, OHLCV]
    ADHOC[Ad-hoc Views<br/>dev.adhoc.stock_data_view]
    FORECAST_TBL[Forecast Table<br/>dev.adhoc.stock_data_forecast]
    FINAL[Final Results<br/>dev.analytics.predicted_stock_data]
    
    %% Forecasting Pipeline
    FC[Forecasting Pipeline<br/>Daily Schedule]
    TRAIN[Train ML Model<br/>Snowflake ML Forecast]
    PREDICT[Generate Predictions<br/>7-day forecast<br/>95% confidence interval]
    
    %% Data Flow
    YF -->|yfinance API| FETCH
    AF -->|Orchestrates| ET
    AF -->|Orchestrates| FC
    
    ET --> FETCH
    FETCH --> LOAD
    LOAD --> RAW
    
    FC --> TRAIN
    RAW --> ADHOC
    ADHOC --> TRAIN
    TRAIN --> PREDICT
    PREDICT --> FORECAST_TBL
    FORECAST_TBL --> FINAL
    RAW --> FINAL
    
    %% Styling
    classDef external fill:#e1f5fe
    classDef airflow fill:#fff3e0
    classDef snowflake fill:#f3e5f5
    classDef pipeline fill:#e8f5e8
    classDef data fill:#fff8e1
    
    class YF external
    class AF airflow
    class ET,FC pipeline
    class SF,RAW,ADHOC,FORECAST_TBL,FINAL snowflake
    class FETCH,LOAD,TRAIN,PREDICT data
```

## Pipeline Components

### ETL Pipeline
- **Schedule**: Daily execution via Apache Airflow
- **Data Source**: Yahoo Finance API (yfinance)
- **Stocks**: Google (GOOGL) and Microsoft (MSFT)
- **Data Period**: 180 days of historical data
- **Target**: Snowflake raw data table with OHLCV data

### Forecasting Pipeline  
- **Schedule**: Daily execution via Apache Airflow
- **ML Framework**: Snowflake ML Forecast
- **Prediction Horizon**: 7 days ahead
- **Confidence Interval**: 95%
- **Output**: Combined actual and predicted data with confidence bounds

### Data Warehouse Schema
- **Raw Layer**: `dev.raw.stock_data` - Historical stock data
- **Ad-hoc Layer**: Views and temporary forecast tables
- **Analytics Layer**: `dev.analytics.predicted_stock_data` - Final results with actuals and forecasts

## Files You'll Need

```
Stock_Price_Prediction/
├── etl_pipeline.py          # Gets the stock data and puts it in Snowflake
├── forecast_pipeline.py     # Makes predictions using ML
├── credentials_setup.md     # How to set up your Snowflake account
└── requirements.txt         # Python packages needed
```

## Getting Started

1. Set up your Snowflake credentials (check `credentials_setup.md`)
2. Install the Python packages: `pip install -r requirements.txt`
3. Copy the pipeline files to your Airflow setup
4. Let it run and watch the magic happen! ✨

That's it! The system will automatically fetch data daily and make predictions for the next 7 days. Pretty straightforward once you get your Snowflake account connected.

Happy coding! 🚀
# Stock_Price_Prediction
