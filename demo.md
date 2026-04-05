# Project Demo
This project is a domain driven financial data framework. This framework demonstrates how domain-driven design can be applied to financial data pipelines, enabling clean abstractions and composable transformations from raw data to model-ready inputs. In its current state, it handles time-series, signals and collections. The focus is on clean architecture to ensure extensibility and scalability


## 1. Data ingestion and out-of-domain representation
Data is ingested via the DataLoader module. It can handle CSV and API calls and is extensible to other formats such as JSON etc. For demonstration purposes, a yahoo Finance API call is presented

### 1.1 TimeSeries Object
A DataLoader call outputs a TimeSeries object, the most basic data object within this framework. A TimeSeries represents structured financial observations (e.g. OHLCV data) indexed by time. In this example, the stock data for SAP SE is fetched, starting from 1st of Jan. 2026 (Ticker: SAP.DE)


```python
# Importing from backend 

ts = DataLoader.load(YfinanceSource(["SAP.DE"], start="2026-01-05", end="2026-03-31")) # TimeSeries obj = ts

# Returns a simple representation of the time series including:
# - ticker label (name), 
# - index format (datatime),
# - number of observations (length)
print(ts) 
```

    TimeSeries(name=SAP.DE, type=datetime, length=61)


### 1.2 Collectable Interface and the .collect() method
The Collectable interface applies a logic, that is similar to Java-Streams. TimeSeries objects implement the interface and the contract is called with the **terminal** .collect() method. Inside this method, various adapters can be called for Data representation. This way, the domain is kept clean with minimal use of external libraries. 

Example: The PandasAdapter module allows representing any TimeSeries object in a pandas dataframe. Any pandas-native operation can be performed this way, outside of the platform's domain. 


```python
import PandasAdapter as pa

ts.collect(pa)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Close</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Volume</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2026-01-05</th>
      <td>206.199997</td>
      <td>200.199997</td>
      <td>206.199997</td>
      <td>199.600006</td>
      <td>1313820</td>
    </tr>
    <tr>
      <th>2026-01-06</th>
      <td>202.050003</td>
      <td>203.600006</td>
      <td>203.949997</td>
      <td>200.600006</td>
      <td>1513269</td>
    </tr>
    <tr>
      <th>2026-01-07</th>
      <td>208.399994</td>
      <td>203.550003</td>
      <td>208.399994</td>
      <td>202.750000</td>
      <td>1606761</td>
    </tr>
    <tr>
      <th>2026-01-08</th>
      <td>206.500000</td>
      <td>208.649994</td>
      <td>209.250000</td>
      <td>205.899994</td>
      <td>1215756</td>
    </tr>
    <tr>
      <th>2026-01-09</th>
      <td>212.350006</td>
      <td>207.000000</td>
      <td>214.250000</td>
      <td>206.000000</td>
      <td>1743178</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2026-03-24</th>
      <td>147.619995</td>
      <td>149.759995</td>
      <td>151.039993</td>
      <td>146.000000</td>
      <td>4380715</td>
    </tr>
    <tr>
      <th>2026-03-25</th>
      <td>146.899994</td>
      <td>148.779999</td>
      <td>150.539993</td>
      <td>145.360001</td>
      <td>3757697</td>
    </tr>
    <tr>
      <th>2026-03-26</th>
      <td>144.639999</td>
      <td>145.399994</td>
      <td>148.080002</td>
      <td>143.520004</td>
      <td>3752998</td>
    </tr>
    <tr>
      <th>2026-03-27</th>
      <td>142.559998</td>
      <td>145.740005</td>
      <td>147.320007</td>
      <td>142.100006</td>
      <td>3568581</td>
    </tr>
    <tr>
      <th>2026-03-30</th>
      <td>147.020004</td>
      <td>142.940002</td>
      <td>147.199997</td>
      <td>142.779999</td>
      <td>3074669</td>
    </tr>
  </tbody>
</table>
<p>61 rows × 5 columns</p>
</div>



### 1.3 TimeSeriesCollection Object

In addition to a TimeSeries Object, DataLoader can also return a TimeSeriesCollection, where each time series represents a single asset. Such an object, includes an align() method, to align each timestamp onto one shared index. This allows for operations on multiple assets. 

Example:
yfinance tickers for Infineon AG and NVIDIA are IFX.DE and NVDA respectively. 
The start date is arbitrarily chosen as the 5th of Jan. 2026


```python
# TimeSeriesCollection obj = tsc
tsc = DataLoader.load(YfinanceSource(["IFX.DE", "NVDA"], start="2026-01-05", end="2026-03-31")) 
print(tsc)
```

    TimeSeriesCollection(n_assets=2, lengths=['IFX.DE: 61', 'NVDA: 59'])


As shown in the above example, the NVDIA stock data has 2 observations less, than the Infineon stock data.
Using .collect(PandasAdapter), pandas can be used natively. Using a simple pandas operation, the NaN values are extracted within the collection. 
The 2 missing datapoints of NVDIA are both US bank holidays (MLK day and President's day)


```python
df = tsc.collect(pa)
df[df.isnull().any(axis=1)]
```

<table border="1" class="dataframe">
  <thead>
    <tr>
      <th>Feature</th>
      <th>Close</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Volume</th>
      <th>Close</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Volume</th>
    </tr>
    <tr>
      <th>Asset</th>
      <th>IFX.DE</th>
      <th>IFX.DE</th>
      <th>IFX.DE</th>
      <th>IFX.DE</th>
      <th>IFX.DE</th>
      <th>NVDA</th>
      <th>NVDA</th>
      <th>NVDA</th>
      <th>NVDA</th>
      <th>NVDA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2026-01-19</th>
      <td>40.324707</td>
      <td>40.607531</td>
      <td>40.905243</td>
      <td>40.066694</td>
      <td>3181623</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2026-02-16</th>
      <td>42.885006</td>
      <td>43.177751</td>
      <td>43.391110</td>
      <td>42.557524</td>
      <td>1502991</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



.align() enforces a shared index across all assets by removing timestamps with missing data. If NaNs are dropped in the process, a UserWarning is returned for the respective timestamps for logging and debugging reasons. After alignment, the TimeSeriesCollection has complete data and can be used for transformation purposes.


```python
tsc.align()
```

    UserWarning: Alignment removed timestamp(s): [Timestamp('2026-01-19 00:00:00'), Timestamp('2026-02-16 00:00:00')]
      warnings.warn(TimeSeriesCollection(n_assets=2, lengths=['IFX.DE: 59', 'NVDA: 59'])



## 2. Signals

Signals represent typed transformations of time series data. 
They encapsulate financial meaning (e.g. prices, returns) and define valid operations on those representations. 
In its current state, this project has the following signals:
- Base Signal
- Price Signal
- Return Signal 
- Interest Rate Signal and Cash Flow Signal (both still work in progress)

The idea behind this architecture is categorizing raw data, and transforming it into clearly defined types. A clear data-transformation pipeline is the result of this approach. 

### 2.1 Price Signal (single asset case)

Taking the TimeSeries example from before (SAP stock data), one can clearly see that the API call returned Open, close, high, low and volume columns. All of these, can be simplified down to being a price. A vectorized operation can be called on a TimeSeries object, transforming the raw TimeSeries data, into a Price signal, in the example below .close() returns the closing prices of the stock.


```python
sap_close = ts.close()
sap_close.collect(pa).head()
```
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SAP.DE.Close</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2026-01-05</th>
      <td>206.199997</td>
    </tr>
    <tr>
      <th>2026-01-06</th>
      <td>202.050003</td>
    </tr>
    <tr>
      <th>2026-01-07</th>
      <td>208.399994</td>
    </tr>
    <tr>
      <th>2026-01-08</th>
      <td>206.500000</td>
    </tr>
    <tr>
      <th>2026-01-09</th>
      <td>212.350006</td>
    </tr>
  </tbody>
</table>
</div>



On a price signal, basic statistical operations can now be performed. The stats modules introduce dependence, moments and regression. Below, an example of the basic, but extendable operations that can be done. The demonstration shows, how pandas operations lead to the same results 

Note: pandas uses sample statistics (ddof=1) by default,
as does this implementation.

Using ddof=1 yields identical results.


```python

mean = sap_close.mean()
std = sap_close.std()
var = sap_close.var()

df = sap_close.collect(pa)

print(f"Mean: {mean:.6f}")
print(f"Pandas: {df.mean()}\n")

print(f"STD: {std:.6f}")
print(f"Pandas: {df.std()}\n")

print(f"Variance: {var:.6f}")
print(f"Pandas: {df.var()}")
```

    Mean: 176.398197
    Pandas: SAP.DE.Close    176.398197
    dtype: float64
    
    STD: 18.693546
    Pandas: SAP.DE.Close    18.693546
    dtype: float64
    
    Variance: 349.448648
    Pandas: SAP.DE.Close    349.448648
    dtype: float64


### 2.2 Return signal (single asset case)

Following this transformation principle, a price signal can become a return signal. This is simply done by applying the .returns() method. Since statistical operations are implemented in the base signal, it's inherited by not only Price Signals but also all other signals (including return signals)



```python
# Uncommenting reveals all return functions
# Discrete returns with .returns() as type ReturnSignal
print(sap_close.returns())

sap_close.returns().collect(pa).head()

# # Absolute returns
# sap_close.abs_returns().collect(pa).head()

# # Continuous returns (log-returns)
# sap_close.log_returns().collect(pa).head()
```

    <ReturnSignal object>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SAP.DE.Close</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2026-01-06</th>
      <td>-0.020126</td>
    </tr>
    <tr>
      <th>2026-01-07</th>
      <td>0.031428</td>
    </tr>
    <tr>
      <th>2026-01-08</th>
      <td>-0.009117</td>
    </tr>
    <tr>
      <th>2026-01-09</th>
      <td>0.028329</td>
    </tr>
    <tr>
      <th>2026-01-12</th>
      <td>0.003532</td>
    </tr>
  </tbody>
</table>
</div>



### 2.3 Vectorized operations on Multi-Asset Signals (SignalCollection)
The same methods can also be applied in a vectorized form in the Multi-Asset Case (SignalCollection). 
Below an example, where the TimeSeriesCollection is aligned, then closing prices are selected. On the price signals, log returns are calculated and ultimately collected. 


```python
signals = tsc.align().close()
print(signals)       
print(signals.mean())        
print(signals.std())         
print(signals["NVDA"].mean())       # single signal access
```

    SignalCollection(n_assets=2, assets=['IFX.DE', 'NVDA'])
    {'IFX.DE': 41.61141295352225, 'NVDA': 183.51674846067266}
    {'IFX.DE': 2.4165504181295234, 'NVDA': 6.27098274606509}
    183.51674846067266


   Alignment removed timestamp(s): [Timestamp('2026-01-19 00:00:00'), Timestamp('2026-02-16 00:00:00')]
      warnings.warn(


## 3. Models
In this section, models are presented. The actual motivation, for this framework, is to simplify the transformation pipelines, so that models only care about signals, not raw data. This enhances speed and simplifies model creation, testing and simulation!

For now, a few key components of the CAPM have been implemented.

### 3.1 Capital Asset Pricing Model - Security Market Line (SML)
In this model, the security market line is calculated. Important components like the beta, or jensen's alpha can be used to determine the performance and valuation of a stock, but the results can also be reused, for example in the calculation of the WACC inside a DCF Model. 


```python
## model imported from internal framework

# Let's say we want to decide, how the stock of Infineon has been performing in the past 5 years.
# We use the DAX30 as the benchmark, since Infineon is one of the constituents. 

tsc_sml = DataLoader.load(YfinanceSource(["IFX.DE", "^GDAXI"], start="2021-01-05", end="2026-01-05"))

# fetch historic closing prices
hist_prices = tsc_sml.align().close()

# Plug the closing prices into the sml model, returns are automatically generated inside the function
result = sml(
    hist_prices["IFX.DE"], 
    hist_prices["^GDAXI"], 
    risk_free_rate=0.02
    )

print(result)
```

    SMLResult(
      expected_return=-0.008070,
      beta=1.440638,
      alpha=0.008522,
    )


As can be seen, with this pipeline, from fetching the data to transforming the raw data to feeding it into a model works fast and produces efficient results
