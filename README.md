# Currency Exchange Rate Pipeline

A serverless ETL pipeline for fetching, storing, and processing currency exchange rates from the Open Exchange Rates API to Snowflake using AWS Lambda and S3.

## Overview

This project offers an automated solution for retrieving real-time currency exchange rates and storing them in a structured format in Snowflake. The pipeline includes:

1. Fetching exchange rates from the Open Exchange Rates API using AWS Lambda  
2. Storing the raw JSON response in an S3 bucket  
3. Processing and loading the data into Snowflake tables for analysis  

## Architecture

![Architecture Diagram](https://github.com/FahadGhaffar/OpenExchangeAPi_TO_Snowflake_and_S3/blob/main/architecture-diagram.jpeg)

- **AWS Lambda**: Executes the main ETL process on a defined schedule  
- **Amazon S3**: Stores raw exchange rate data as JSON files  
- **AWS Secrets Manager**: Securely stores and manages database credentials  
- **Snowflake**: Hosts and processes the exchange rate data  

## Prerequisites

- An AWS account with necessary permissions  
- A Snowflake account with database creation privileges  
- An Open Exchange Rates API key: [https://openexchangerates.org/](https://openexchangerates.org/)  
- Python 3.9 installed  

## Setup Instructions

### AWS Setup

1. Create an IAM role with the following permissions:
   - S3 Full Access  
   - Lambda Execution Role  
   - EventBridge Access  
   - Lambda Full Access  

2. In AWS Secrets Manager, create a secret with the ID `db/currency-exchange-rate` and the following content:
   ```json
   {
     "fusion_snowflake": {
       "username": "your_username",
       "password": "your_password",
       "account_name": "your_snowflake_account"
     }
   }
   ```

3. Create an S3 bucket to store the raw exchange rate data.  

### Snowflake Setup

1. Run the SQL script located at `code/snowflake.sql` to:
   - Create the `CURRENCY_DB` database and `CURRENCY` schema  
   - Create the required tables (`EXCHANGE_RATES_RAW`, `EXCHANGE_RATES_STG`, `EXCHANGE_RATES`)  
   - Define the stored procedure for processing the data  

### Lambda Function Setup

1. Create a new AWS Lambda function using Python 3.8 or later  
2. Upload the code from `code/lambda_function.py` and `code/snowflake_provider.py`  
3. Set the following environment variables:
   - `environment`: DEV  
   - `oer_app_id`: YOUR-APP-KEY  
   - `oer_base_currency`: USD  
   - `oer_base_url`: https://openexchangerates.org/api/latest.json  
   - `region_name`: us-east-1  
   - `s3_bucket_name`: s3-currency-exchange-rate-fg  
   - `snowflake_db`: CURRENCY_DB  
   - `snowflake_role`: ACCOUNTADMIN  
   - `snowflake_wh`: COMPUTE_WH  

4. Set up an EventBridge (CloudWatch Events) trigger to run the Lambda function on your desired schedule  

## Project Structure

```
.
├── code/
│   ├── lambda_function.py     # Main AWS Lambda function
│   ├── snowflake_provider.py  # Snowflake connection and utility functions
│   └── snowflake.sql          # SQL scripts for setting up Snowflake
├── environment-variables.txt  # Required environment variable definitions
├── roles.txt                  # IAM roles required by AWS
├── secret-manager.txt         # Instructions for configuring secrets
└── convention.txt             # Project structure and naming conventions
```

## How It Works

1. The Lambda function is triggered on a predefined schedule  
2. It fetches the latest exchange rates from the Open Exchange Rates API  
3. The raw JSON response is saved in the S3 bucket using the path format: `exchange_rates/{year}/{month}/{day}/exchange-rates-{hour}.json`  
4. The raw data is inserted into the `EXCHANGE_RATES_RAW` table in Snowflake  
5. A stored procedure (`SP_EXCHANGE_RATE_LOADING`) processes the data:
   - Extracts relevant data into the staging table  
   - Merges the staged data into the final `EXCHANGE_RATES` table  

## Monitoring and Maintenance

- Use AWS CloudWatch to monitor Lambda executions  
- Use Snowflake’s query history and monitoring features for tracking data processing  
- Periodically review Snowflake tables to ensure data is being loaded correctly  

## Contributing

1. Fork the repository  
2. Create a new feature branch  
3. Commit your changes  
4. Push the branch to your fork  
5. Open a Pull Request  
