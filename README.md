# Streamlit crypto analytics 

[https://app-crypto-dashboard.streamlit.app/](https://app-crypto-dashboard.streamlit.app/)

**The present project consist of a dashboard of cryptocurrencies with data obtained by calling a FastAPI endpoint**
    
This API connect natively to a database stored in Aiven cloud-managed services using timescaledb extension of Aiven. The application of hypertable was done using SQL inside the code to convert the SQLModel table to hypertable, defining the **chunks with an interval of 1 day**. Aiven free service doesn't allow some of the properties of timescaledb (the drop_after function is not available), so the chunk drops was manually setted inside Aiven with pg_cron extension

```sql
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
CREATE EXTENSION IF NOT EXISTS pg_cron CASCADE;

-- Create an automated schedule inside Aiven to remove chunks after 1 month
SELECT cron.schedule(
    'chunk clean',
    '0 0 * * *',
    $$ SELECT drop_chunks('public.eventmodel', INTERVAL '1 month') $$ 
);
```

The database gets the data through a workflow with github actions using a script to extract data from **coingecko API every 5 minutes**, transform it and clean it, then finally inyect the data to the database using the POST private method of the crypto analytics API created.

The API only publicly allows to use the GET  method to extract data from the database. The data extracted consist of time buckets setted trough the parameters used on the API call.

Data columns and values:
* **`bucket`**: the time bucket representing the start of the specific time interval.
* **`symbol`**: the unique ticker or identifier of the cryptocurrency (e.g., `btc`, `eth`).
* **`avg_price`**: the average price of the data calculated within that specific time bucket.
* **`count`**: the total number of raw data points collected and aggregated inside the bucket during that interval.
* **`market_cap`**: the total market capitalization of the cryptocurrency recorded at that specific period.
* **`volume_24h`**: the 24-hour rolling trading volume.
* **`change_24h`**: the percentage price change of the cryptocurrency over the last 24 hours.

---
**params:**

- symbol (required): this accept multiple symbols to extract more data

- duration: select the time bucket (allowed buckets: **`15 minutes`, `1 hour`, `4 hours`, `1 day`** )

- history: select the time history of the data (allowed values: **`1 day`, `7 days`, `30 days`**)

Example extracting the data of the day about btc with intervals of 15 minutes:

[https://analytics-api-hg65.onrender.com/api/v1/analytics?symbol=btc&duration=15%20minutes&history=1%20day](https://analytics-api-hg65.onrender.com/api/v1/analytics?symbol=btc&duration=15%20minutes&history=1%20day)



> **Credits**: 
    > this project was inspired following the FastAPI videotutorial created by **Codingforentrepreneurs**

> [video](https://www.youtube.com/watch?v=tiBeLLv5GJo)
> [github repo](https://github.com/codingforentrepreneurs/analytics-api) 
