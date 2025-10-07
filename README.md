# Data Warehouse Lab 1

Hey there! 👋 This is Data Warehouse Lab 1, where we're building a cool stock price prediction system.

Think of it as a smart assistant that watches Google and Microsoft stocks, learns from their past behavior, and tries to predict what might happen next week. Pretty neat, right?

## What We're Building

We're creating an automated system that:

* Grabs stock data from the internet (Google & Microsoft)
* Stores it safely in Snowflake (our data warehouse)
* Uses machine learning to predict future prices
* Runs everything automatically with Airflow

## Files You'll Need

```
Stock_Price_Prediction/
├── etl_pipeline.py          # Gets the stock data and puts it into Snowflake
├── forecast_pipeline.py     # Makes predictions using ML
├── credentials_setup.md     # Instructions to set up your Snowflake account
└── requirements.txt         # Python packages needed
```

## Getting Started

1. Set up your Snowflake credentials (check `credentials_setup.md`).
2. Install the Python packages: `pip install -r requirements.txt`.
3. Copy the pipeline files to your Airflow setup.
4. Let it run and watch the magic happen! ✨

That's it! The system will automatically fetch data daily and make predictions for the next 7 days. It's pretty straightforward once you get your Snowflake account connected.

Happy coding! 🚀

---

# Architecture Diagrams

Below are Mermaid diagrams for both the **ETL** and **Forecast** pipelines. You can paste these into GitHub/VS Code previewers that support Mermaid.

## ETL Pipeline (Airflow DAG & Data Flow)

```mermaid
flowchart TD
    subgraph Airflow[Airflow DAG: etl_pipeline]
        A[Schedule @ daily \n (Cron/Timetable)] --> B[Task: Extract Prices\n(Yahoo/Alpha Vantage API)]
        B --> C[Task: Transform\n(Clean, Parse, Normalize, Add Tickers)]
        C --> D[Task: Load to Snowflake\nPUT/COPY INTO or Snowflake Connector]
        D --> E[Task: Quality Checks\n(Row counts, Nulls, Ranges)]
    end

    subgraph Snowflake[Snowflake]
        STG[(STG.PRICES_RAW)]
        DIM[(DIM.SYMBOL)]
        FCT[(FCT.DAILY_PRICES)]
    end

    B -.raw json/csv.-> STG
    C -.upserts/merge.-> FCT
    D --> STG
    E --> FCT

    classDef store fill:#e8f3ff,stroke:#4f83ff,stroke-width:1px;
    class STG,DIM,FCT store;
```

### ETL Notes

* **Extract**: Pulls OHLCV for tickers like `GOOGL` and `MSFT`.
* **Transform**: Cleans columns, standardizes datatypes/timezones, and tags with tickers.
* **Load**: Uses the Snowflake Python connector or `COPY INTO` for efficient loading.
* **DQ Checks**: Simple assertions to catch missing days or out-of-range values.

## Forecast Pipeline (Airflow DAG & ML Flow)

```mermaid
flowchart TD
    subgraph Airflow2[Airflow DAG: forecast_pipeline]
        S[Schedule @ daily after ETL] --> R[Task: Read Training Data\n(Snowflake SELECT)]
        R --> F[Task: Feature Engineering\n(Lags, Rolling Means, Returns)]
        F --> M{Train/Load Model?}
        M -->|Cold Start| T[Task: Train Model\n(e.g., Prophet/XGBoost)]
        M -->|Reuse| L[Task: Load Saved Model]
        T --> P[Task: Predict 7 Days]
        L --> P
        P --> W[Task: Write Forecasts\n(to SNOWFLAKE FCT.FORECASTS)]
        W --> V[Task: Validate/Drift Checks]
    end

    subgraph Snowflake2[Snowflake]
        SRC[(FCT.DAILY_PRICES)]
        OUT[(FCT.FORECASTS)]
        META[(ML.MODEL_REGISTRY)]
    end

    R --> SRC
    W --> OUT
    T -.persist.-> META
    L -.read model.-> META

    classDef store fill:#e8f3ff,stroke:#4f83ff,stroke-width:1px;
    class SRC,OUT,META store;
```

### Forecast Notes

* Pulls recent history from `FCT.DAILY_PRICES` and engineers time-series features.
* Trains (or reuses) a model and writes 7-day forecasts into `FCT.FORECASTS`.
* Optional drift/accuracy checks compare recent predictions vs. actuals.

---

## Airflow Task Naming (Suggested)

* `extract_prices` → `transform_prices` → `load_to_snowflake` → `dq_checks`
* `read_training_data` → `feature_engineering` → `train_or_load_model` → `predict_next_7d` → `write_forecasts` → `validate_forecasts`

## Environments & Credentials

* Store Snowflake credentials in Airflow Connections/Variables.
* Parameterize tickers (default: `GOOGL`, `MSFT`) and forecast horizon (default: 7).

## Troubleshooting

* Verify network egress from Airflow to Snowflake and to market data APIs.
* Check task logs for API quota issues and Snowflake permission errors.
* Ensure warehouse auto-resume and correct role/database/schema permissions.
