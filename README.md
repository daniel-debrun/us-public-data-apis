# US public data APIs: free access and rate limits

A reference to the US data APIs evaluated as research data providers for qfold (now verdex) in
September–October 2026. Each entry records what the API provides, what is available free of charge, its rate
limits, and any notable behaviour observed during integration.

Two kinds of rate limit appear in the tables:

- **Published:** the limit stated by the provider.
- **Paced at:** the request rate used in practice without throttling. This is an observation, not a
  guarantee from the provider.

Every API listed is usable with a free account or with no account at all. Free keys are generally issued
within a minute of registration. The exception is BEA, whose key must be activated from the registration
email before it works.

## Contents

- [Macro and economic statistics](#macro-and-economic-statistics)
- [Government finance and spending](#government-finance-and-spending)
- [Energy, weather, agriculture, environment](#energy-weather-agriculture-environment)
- [SEC filings and market regulation](#sec-filings-and-market-regulation)
- [Exchanges and market statistics](#exchanges-and-market-statistics)
- [Political and disclosure data](#political-and-disclosure-data)
- [Commercial APIs with a free tier](#commercial-apis-with-a-free-tier)
- [Publication lags](#publication-lags)
- [Findings](#findings)
- [Evaluated but not integrated](#evaluated-but-not-integrated)

---

## Macro and economic statistics

| API | Description | Free access | Rate limit | Notes |
|---|---|---|---|---|
| [FRED](https://fred.stlouisfed.org/docs/api/fred/) (St. Louis Fed) | About 800,000 US and international economic time series | All data, with a [free key](https://fredaccount.stlouisfed.org/apikey) | Published: 120 requests/min per key | **ALFRED vintages** (`realtime_start`) return each value as it stood on a past date. Among the macro sources listed, FRED is the only one that allows a backtest to avoid revised values. Up to 100,000 observations per request. |
| [FRED release calendar](https://fred.stlouisfed.org/docs/api/fred/releases_dates.html) | Actual (past) and scheduled (future) dates of each statistical release, and the series each release carries | Same key as FRED | Same as FRED | Past dates are actual release dates. Scheduled dates are subject to change. |
| [BLS](https://www.bls.gov/developers/) (Bureau of Labor Statistics) | CPI, unemployment, payrolls, wages, JOLTS and any other BLS series by ID | All data, with a [free v2 key](https://data.bls.gov/registrationEngine/) | Published (keyed): 50 requests per 10 s, **500 per day** | The daily cap is the binding limit. Multiple series IDs can be batched into one request. |
| [BEA](https://apps.bea.gov/api/signup/) (Bureau of Economic Analysis) | NIPA tables: GDP and its components, income, prices, saving | All data, with a free key | Published: 100 requests, 100 MB and 30 errors per minute. **Exceeding any of these triggers a one-hour lockout.** | The key must be **activated from the registration email** before it works. Every response echoes the key, so it should be removed from responses before caching or logging. |
| [Census](https://www.census.gov/data/developers.html) | Economic indicator time series (`timeseries/eits`: retail sales, housing starts, durable goods, trade) and every other Census dataset (ACS, CBP and others) through a single query format | All data, with a [free key](https://api.census.gov/data/key_signup.html) | Paced at 5/s | Predicates are echoed as extra columns, and all values are returned as strings. |

## Government finance and spending

| API | Description | Free access | Rate limit | Notes |
|---|---|---|---|---|
| [Treasury daily par yield curve](https://home.treasury.gov/resource-center/data-chart-center/interest-rates/TextView) | Daily par yields from 1 month to 30 years, one CSV per year | All data, no key | Paced at 4/s | Missing values appear as `N/A`. |
| [Treasury Fiscal Data](https://fiscaldata.treasury.gov/api-documentation/) | Average interest rates on the debt, Debt to the Penny, auction results, and around 80 other fiscal datasets | All data, no key | Paced at 4/s | All values are returned as text, with the string `"null"` for missing values. The field types in the response `meta` can be used to cast columns. |
| [USAspending](https://api.usaspending.gov/) | Federal awards (contracts, grants, loans) by recipient, and obligations over time | All data, no key | Paced at 4/s | Search is **POST-only**, so a cache must key on the URL and the request body together. |
| [FDIC BankFind](https://banks.data.fdic.gov/docs/) | Every FDIC-insured institution, active or closed, with its quarterly Call Report financials | All data, no key | Paced at 5/s | Up to 10,000 rows per page. Dollar amounts are reported **in thousands**. |

## Energy, weather, agriculture, environment

| API | Description | Free access | Rate limit | Notes |
|---|---|---|---|---|
| [EIA v2](https://www.eia.gov/opendata/) (Energy Information Administration) | Any EIA series by route and facets: crude and gasoline stocks, spot prices (WTI, Brent, Henry Hub), power, coal | All data, with a [free key](https://www.eia.gov/opendata/register.php) | Paced at 5/s | **5,000 rows per request**. Long series require paging. |
| [NOAA NCEI Access Data Service](https://www.ncei.noaa.gov/support/access-data-service-api-user-documentation) | GHCN daily weather summaries by station | All data, **no token** | Paced at 4/s | The preferred route for daily data: wide rows, and multi-year spans in a single request. |
| [NOAA CDO v2](https://www.ncdc.noaa.gov/cdo-web/webservices/v2) | Climate Data Online: station search and datasets | Free [token](https://www.ncdc.noaa.gov/cdo-web/token) | Published: 5 requests/s and 10,000/day per token | Each request is capped at **1,000 rows and one year**, so a decade of data for one station takes about 40 requests. Used here for station search only. The token is sent as a header. |
| [USDA NASS Quick Stats](https://quickstats.nass.usda.gov/api) | Crop and livestock production, acreage, yields, prices, stocks and condition, by year and geography | All data, with a free key | Paced at 2/s | |
| [EPA Envirofacts (GHGRP)](https://www.epa.gov/enviro/envirofacts-data-service-api) | Greenhouse Gas Reporting Program: emissions by facility, year, gas and sector for every US facility emitting over 25,000 t CO2e/yr, with parent companies | All data, no key | Paced at 2/s | As of October 2026, the latest available reporting year was **2023**. |

## SEC filings and market regulation

| API | Description | Free access | Rate limit | Notes |
|---|---|---|---|---|
| [SEC EDGAR (data.sec.gov)](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) | Ticker-to-CIK map, each company's full filing history, XBRL `companyfacts`, and XBRL `frames` (a single fact for every filer at once) | All data, no key | Published: **10 requests/s** | A **User-Agent containing a contact email is required**. Requests without one are refused. `acceptanceDateTime` records the exact minute a filing was received, which point-in-time studies depend on. `frames` provides a cross-sectional fundamentals panel. XBRL coverage begins in 2009–2011, phased in by filer size. |
| [EDGAR full-text search](https://efts.sec.gov/LATEST/search-index?q=%22example%22) | Filings from 2001 onward containing a given phrase | All data, no key | Same SEC limit (paced at 5/s) | Same User-Agent requirement. |
| [SEC insider transactions data sets](https://www.sec.gov/data-research/sec-markets-data/insider-transactions-data-sets) | Quarterly bulk Form 3/4/5 data | All data, no key | Same SEC limit | The bulk files are faster than parsing individual Form 4 filings. |
| [SEC fails-to-deliver](https://www.sec.gov/data-research/sec-markets-data/fails-deliver-data) | NSCC net fails by security and settlement date | All data, no key | Same SEC limit | Zipped, pipe-delimited files published twice a month (quarterly files for 2004 to mid-2009). The files **have moved folders before**, so the list page should be read on each run rather than the URLs hard-coded. |
| [FINRA Query API](https://developer.finra.org/docs) | Twice-monthly consolidated short interest, and Reg SHO daily short-sale volume | Both datasets are **open: no credentials required** | Paced at 5/s | Queries are POST requests with a JSON filter body, at most 5,000 rows per page. Short volume through the API covers **only the most recent year**. Earlier days are available as daily files at `cdn.finra.org/equity/regsho/daily/CNMSshvol<YYYYMMDD>.txt`. |
| [CFTC Commitments of Traders](https://publicreporting.cftc.gov/) | Weekly futures positions by trader type: legacy, disaggregated and traders-in-financial-futures (TFF) reports | All data, no key (Socrata API) | Paced at 4/s | Up to 50,000 rows per page with `$limit`/`$offset`. Only the futures-only reports were integrated. |

## Exchanges and market statistics

| API | Description | Free access | Rate limit | Notes |
|---|---|---|---|---|
| [Cboe market statistics](https://www.cboe.com/us/options/market_statistics/) | Daily put/call volume ratios, and the settlement history of each VIX futures contract | All data, no key (public files) | Paced at 5/s | Put/call data **up to 2019-10-04** is one CSV per product. **From 2019-10-07** it is one JSON request per trading day, so date ranges should be kept short. VIX futures are a contract list plus one CSV per expiry. Cboe has relocated these files before. |

## Political and disclosure data

| API | Description | Free access | Rate limit | Notes |
|---|---|---|---|---|
| [Lobbying Disclosure Act API](https://lda.gov/api/) (formerly lda.senate.gov) | Quarterly reports of which lobbyists represented which clients, on which issues, and for what amount | All data, with a free key | Published (keyed): 120 requests/min | The key is sent as a header. |
| [House Clerk financial disclosures](https://disclosures-clerk.house.gov/FinancialDisclosure) | Yearly index of every House member's filings, including Periodic Transaction Reports (stock trades), each linked to its PDF | All data, no key | Paced at 2/s | **Individual trades appear only inside the PDFs**: the index lists filings, not transactions. **Senate disclosures (efdsearch.senate.gov) offer no bulk download** and were not integrated. |

## Commercial APIs with a free tier

These are not government sources but were used alongside those above.

| API | Description | Free access | Rate limit | Notes |
|---|---|---|---|---|
| [Alpha Vantage](https://www.alphavantage.co/documentation/) | Prices, news with sentiment, insider trades, earnings and IPO calendars, listing status (including delisted tickers), commodities, FX, crypto, market status | Most endpoints, with a free key | Published (free): **25 requests/day**, at most about 1 request/s | Errors, rate limits and premium-only endpoints all return **HTTP 200** with a message, at times as a CSV with "Information" spread one letter per column. Every response body must be checked. Free daily prices are **not adjusted** for splits or dividends, and full history (`outputsize=full`) is premium. Historical options chains are premium only. |
| [Tiingo](https://www.tiingo.com/documentation/general/overview) | End-of-day prices adjusted for splits and dividends, ticker metadata, IEX intraday bars, crypto | Prices, metadata, intraday and crypto, with a free key | Published (free): **50 requests/hour, 1,000/day** | The **news API returns 403 on the free plan**. Best suited to long adjusted price histories. |
| [Wikimedia pageviews](https://doc.wikimedia.org/generated-data-platform/aqs/analytics-api/) | Daily views of any Wikipedia article, a public measure of attention | All data, no key | Paced at 10/s | Requests require a **User-Agent with contact details**. |
| [GDELT DOC 2.0](https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/) | Volume and tone of worldwide online news coverage matching a query, over time | All data, no key | Published: **1 request per 5 s per IP** | One request succeeded, after which the **client IP address was blocked**. Throttling returns plain text with HTTP 200, or a 429. Considered unreliable. |

---

## Publication lags

Most of this data is published some time after the period it describes. Backtests must use the publication
date, not the period date.

| Data | Public availability |
|---|---|
| CFTC COT positions | Positions as of Tuesday, published Friday at 15:30 ET |
| EIA weekly petroleum stocks | Wednesday, for the week ending the previous Friday |
| FINRA short interest | About 8 business days after the settlement date |
| FRED, BLS, BEA and Census macro series | On the release date (see the FRED release calendar), **with later revisions**. ALFRED vintages provide the values as originally published. |
| SEC filings | At `acceptanceDateTime`: a filing accepted at 16:31 was not tradeable at that day's close |

## Findings

1. **Most US government data is free, either keyless or behind a free key issued in about a minute.** The
   limits that bind in practice are BLS (500/day), Alpha Vantage (25/day), Tiingo (50/hour) and GDELT (one
   request per 5 s).
2. **Rate limiting should be enforced per host across the whole machine**, not per process. Parallel agents
   or notebooks otherwise multiply the request rate and risk blocking the key or IP. Quotas are best stored
   as (calls, window) pairs, with any call that would exceed a quota refused rather than sent.
3. **Caches should be keyed by URL with credentials removed.** BEA echoes the key in its responses, and
   other providers carry it in the query string. Parameters such as `api_key`, `key`, `token` and `UserID`
   should be removed before anything is written to disk or included in an error message.
4. **Many refusals resemble successful responses.** Alpha Vantage and GDELT return HTTP 200 for rate limits
   and errors. Response bodies must be validated and refusals excluded from the cache.
5. **Paging limits determine client design:** EIA 5,000 rows, FINRA 5,000, FDIC 10,000, CFTC 50,000, and
   NOAA CDO 1,000 rows and one year per request.
6. **Some data exists only as bulk files:** FINRA short volume older than one year, SEC fails-to-deliver and
   insider data sets, and Cboe put/call history. These file locations change, so each should be discovered
   from the index page on each run.
7. **The SEC requires a contact email in the User-Agent.** sec.gov also failed to resolve through one
   university network's DNS, so SEC retrieval may need to run from a different network.
8. **Daily weather data is best retrieved from NOAA's Access Data Service rather than CDO v2**: it needs no
   token and replaces about 40 requests with one.

## Evaluated but not integrated

- **BEA**: an adapter was written but remains untested, pending activation of the key.
- **USPTO PatentsView**: deferred.
- **Polymarket, Kalshi** (prediction markets): deferred.
- **Senate financial disclosures (eFD)**: no bulk download available.
- **Paid options history** (Alpha Vantage premium, ORATS, Polygon, Databento, Cboe DataShop): unnecessary,
  as an academic WRDS licence (OptionMetrics) covered the requirement. That licence is not free and its data
  cannot be redistributed.

Rate limits and free tiers reflect published terms or observed behaviour as of October 2026. Providers change
these terms, so the linked documentation should be checked before relying on any figure.
