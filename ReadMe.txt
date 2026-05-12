1. Purpose : The Price Determination Engine calculates prices for Blue Yonder planning inputs using two pricing approaches:
  Weighted-average pricing
  Price-list based pricing for France, Japan, Amazon Centralized, and Produo
2. What the Pipeline Does
    The pipeline executes the following steps:
      1. Build price_det_input from Blue Yonder source data
      2. Build price_list_input from SAP price-list data
      3. Build transaction history input
      4. Create weighted-average reference tables
      5. Split population into weighted-average and price-list groups
      6. Apply pricing logic:
          - Weighted Average Standard
          - Weighted Average Special
          - France Price List
          - Japan Price List
          - Amazon Centralized Price List
          - Produo Price List
      7. Combine all outputs into final_output
      8. Validate final_output
      9. Write final output to price_det_output_v1
      10. Write Blue Yonder ingestion output to by_pricespecification_output
3. Project Folder Structure : 
      Price_Determination_Engine/
    │
    ├── src/
    │   ├── main.py
    │   ├── config.py
    │   ├── io_tables.py
    │   ├── validations.py
    │   ├── input_builder/
    │   ├── weighted_average/
    │   ├── price_list/
    │   └── by_ingestion/
    │
    ├── sql/
    │   └── ddl/
    │
    └── run/
        └── run_dev_pipeline
4. DDL Dependencies (Must Be Executed Before Run)
   Before running the Price Determination Engine, all required Delta tables must exist.These tables are created using SQL DDL scripts located under:
   sql/ddl/
   The following tables must be created before running the pipeline:
    | Table                         | Purpose                  |
    | -------------------           | ------------------------ |
    | `price_det_by_base`           | BlueYonder Data          |
    | `price_det_input`             | Needs pricing population |
    | `price_list_input`            | SAP price list input     |
    | `input_trans_price`           | Transaction history      |
    | `price_det_output_v1`         | Final pricing output     |
    | `by_pricespecification_output`| Blue Yonder ingestion    |
5. Environment Parameter: 
    The env parameter controls which schema is used.
      env="dev"
      env="test"
      env="prod"
    Current dev configuration points to:
      catalog = usecase_finance_ml
      schema  = dev
    Example for test:
      final_output = run_price_engine(
          env="test",
          persist_by_base=False
        )
6. Key Configurations
      Configuration is managed in:src/config.py
      Important Settings : 
          japan_multiplier: Multiplier applied to Japan price-list values to adjust for currency/unit scaling differences.
          use_japan_ean_fallback: Enables fallback to EAN-based matching when primary Japan price-list matching fails.
          use_amzn_ean_fallback: Enables fallback to EAN-based matching for Amazon Centralized pricing when primary match fails.
          use_amzn_ssku_fallback: Enables fallback to SSKU-based matching for Amazon Centralized pricing after EAN fallback.
          use_prd_ean_fallback: Enables fallback to EAN-based matching for Produo pricing when primary match fails.
          use_prd_ssku_fallback: Enables fallback to SSKU-based matching for Produo pricing after EAN fallback.
          hist_start_date: Start date defining the lower bound of the historical and forecast data window used in the BY input query.
          hist_end_date: End date defining the upper bound of the historical and forecast data window used in the BY input query.
          write_mode_by_base: Controls how the intermediate BY base dataset is written (typically overwrite to keep only the latest snapshot).
          write_mode_needs_pricing: Defines write behavior for the needs_pricing table, usually overwritten each run to reflect the latest routing output.
          write_mode_price_list_input: Determines how the price list input table is refreshed, typically overwritten to load the latest source data.
          write_mode_trans_hist: Controls write behavior for the transaction history input table, generally overwritten per run for consistency.
          write_mode_output: Defines how the final pricing output table is written, typically append to maintain historical run data.
          write_mode_by_output: Controls write behavior for the Blue Yonder ingestion table, usually append with batch-level overwrite logic handled separately.
          routing_dimension: Dimension used to determine pricing logic (e.g., COUNTRY decides whether to use price list or weighted average).
          routing_rules: Mapping of routing logic to dimension values (e.g., specific countries mapped to price-list processing).
          default_routing_logic: Default pricing approach applied when no routing rule matches (typically weighted_average).
7. How to Run : Open the Databricks notebook:/Price_Determination_Engine/run/run_dev_pipeline
   Run the below code:
        from src.main import run_price_engine

        final_output = run_price_engine(
            env="dev",
            persist_by_base=False
        )

        print("Price Determination Engine run completed.")
8. Outputs : The output is written to - usecase_finance_ml.dev.price_det_output_v1 & and the Blue Yonder ingestion-ready table is written to - usecase_finance_ml.dev.by_pricespecification_output
