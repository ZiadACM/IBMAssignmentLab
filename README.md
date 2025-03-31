
# 📘 IBMAssignmentLab

As part of the IBM Data Science Professional Certificate, this project demonstrates an end-to-end data analysis pipeline using Python.

## 📌 Objective

Analyze the relationship between stock prices and company revenue for **Tesla** and **GameStop** by:

- Scraping quarterly revenue data from an HTML page using `BeautifulSoup`
- Fetching historical stock data using `yfinance`
- Cleaning and preparing data with `pandas`
- Visualizing trends using interactive plots with `plotly`

## 🧪 Web Scraping Process (How We Got the Revenue Data)

### 🔹 Initial Setup

1. **Store the webpage URL into a variable**

```python
url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0220EN-SkillsNetwork/labs/project/revenue.htm"
```

2. **Use the `requests.get()` method to retrieve the HTML content of the URL**

```python
import requests
data = requests.get(url).text
```

3. **Use `BeautifulSoup` to parse the raw HTML so we can navigate and extract data from it easily**

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(data, 'html.parser')
```

4. **Initialize an empty DataFrame using `pandas`. This DataFrame includes only the column headers (e.g., `"Date"` and `"Revenue"`), with no data yet**

```python
import pandas as pd
tesla_revenue = pd.DataFrame(columns=["Date", "Revenue"])
```

---

### 🔍 How We Extracted the Table Data

1. **Use your browser's Inspect Tool** (`Ctrl + Shift + C`) to locate the `<tbody>` tag containing the table data.
   - There may be multiple tables. Choose the one that contains the revenue data quarterly.

```python
table = soup.find_all("tbody")[1]
```

2. **Loop through each `<tr>` (table row) inside the selected `<tbody>`**

```python
for row in table.find_all("tr"):
```

3. **Inside each `<tr>`, find all `<td>` elements and extract text. In this step we are collecting the data from the table**

```python
    col = row.find_all("td")
    date = col[0].text.strip()
    revenue = col[1].text.strip()
```

4. **Append it to the DataFrame**

```python
 tesla_revenue = pd.concat([
            tesla_revenue,
            pd.DataFrame({"Date": [date], "Revenue": [revenue]})
        ], ignore_index=True)
```

5. **Clean the data revenue ("$" and "," signs, empty values)**

```python
tesla_revenue["Revenue"] = tesla_revenue["Revenue"].str.replace("$", "").str.replace(",", "")
tesla_revenue = tesla_revenue[tesla_revenue["Revenue"] != ""]  
tesla_revenue["Revenue"] = pd.to_numeric(tesla_revenue["Revenue"])
```

---

### 📄 Viewing the Final Revenue DataFrame

After this, we will have the DataFrame ready to retrieve and analyze data from it.  
Here are some useful methods to inspect the data:

- `tesla_revenue.head()` – Shows the first 5 rows  
- `tesla_revenue.tail()` – Shows the last 5 rows  
- `tesla_revenue.info()` – Gives a summary of columns, data types, and non-null values  
- `tesla_revenue.describe()` – Provides basic statistics for numerical columns (e.g. mean, max, min, etc.)

---

## 📈 Extracting Stock Data Using yfinance

1. Import the `yfinance` library and create a `Ticker` object for the stock you're analyzing  
   - **Example:** For Tesla, the ticker symbol is `"TSLA"`

```python
import yfinance as yf

tesla = yf.Ticker("TSLA")
```

---

### 🔍 Extracting and Preparing the Stock Data

1. Use the `.history()` method to fetch the full historical stock data  
   - `"max"` returns all available historical data

```python
tesla_data = tesla.history(period="max")
```

2. Reset the index so the `Date` becomes a column instead of the index  
   - This makes it easier to work with time-series data later on

```python
tesla_data.reset_index(inplace=True)
```

---

### 📄 Viewing the Final Stock DataFrame

Once the data is fetched and the index is reset, you can explore it using:

- `tesla_data.head()` – Displays the first 5 rows  
- `tesla_data.tail()` – Displays the last 5 rows  
- `tesla_data.columns` – Lists all available columns (e.g., Open, High, Low, Close, Volume)

Now we're ready to move to the visualization of the data step and explore trends:

## 📊 Visualization Step

visualize the relationship between historical stock prices and revenue over

```python
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import plotly.io as pio

pio.renderers.default = "iframe"

def make_graph(stock_data, revenue_data, stock):
    fig = make_subplots(rows=2, cols=1, shared_xaxes=True,
                        subplot_titles=("Historical Share Price", "Historical Revenue"),
                        vertical_spacing=0.3)
    
    stock_data_specific = stock_data[stock_data.Date <= '2021-06-14']
    revenue_data_specific = revenue_data[revenue_data.Date <= '2021-04-30']

    fig.add_trace(go.Scatter(x=pd.to_datetime(stock_data_specific.Date, infer_datetime_format=True),
                             y=stock_data_specific.Close.astype("float"),
                             name="Share Price"), row=1, col=1)

    fig.add_trace(go.Scatter(x=pd.to_datetime(revenue_data_specific.Date, infer_datetime_format=True),
                             y=revenue_data_specific.Revenue.astype("float"),
                             name="Revenue"), row=2, col=1)

    fig.update_xaxes(title_text="Date", row=1, col=1)
    fig.update_xaxes(title_text="Date", row=2, col=1)
    fig.update_yaxes(title_text="Price ($US)", row=1, col=1)
    fig.update_yaxes(title_text="Revenue ($US Millions)", row=2, col=1)

    fig.update_layout(showlegend=False,
                      height=900,
                      title=stock,
                      xaxis_rangeslider_visible=True)

    fig.show()

    from IPython.display import display, HTML
    fig_html = fig.to_html()
    display(HTML(fig_html))
```
then we can call the function
```python
make_graph(tesla_data, tesla_revenue, "Tesla")
```
---

### 📌 Note

You can apply the **same steps** shown above to analyze **GameStop** by:

- Replacing the Tesla ticker (`"TSLA"`) with GameStop's ticker (`"GME"`)
- Using the appropriate revenue table for GameStop (from the same or a different URL)
- Calling the function with `make_graph(gme_data, gme_revenue, "GameStop")`

This makes the workflow flexible and reusable for other companies or financial analysis tasks.


## ✅ Conclusion

This project showcased a complete data analysis pipeline using real-world financial data.  
By combining **web scraping**, **data extraction**, **data cleaning**, and **interactive visualization**, we were able to explore the relationship between stock price and revenue for companies like **Tesla** and **GameStop**.

