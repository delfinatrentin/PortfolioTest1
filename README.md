# PortfolioTest1
import pandas as pd
import streamlit as st
import yfinance as yf
import statsmodels.api as sm


st.header("Portfolio Data")
st.sidebar.text("Select your period")
import datetime

import datetime

# Calculate one month ago
one_month_ago = datetime.datetime.now() - datetime.timedelta(days=30)

# Select the since date
since_date = st.sidebar.date_input("Since date", value=one_month_ago)

# Select until date
until_date = st.sidebar.date_input("Until date")

# Convert the dates to strings in the format "YYYY-MM-DD"
since = since_date.strftime("%Y-%m-%d")
until = until_date.strftime("%Y-%m-%d")

# Download the financial data for Block, Shopfy, and Salesforce from Yahoo Finance
Block = yf.Ticker("SQ").history(start= since, end=until)
Shopfy = yf.Ticker("SHOP").history(start= since, end=until)
Salesforce = yf.Ticker("CRM").history(start=since, end=until)

st.sidebar.text("Graphs")

#Create a button widget to select the stock
selected_stock = st.sidebar.radio("Select your stock", ["Block", "Shopfy", "Salesforce"])

# Display the financial data
#st.write("Block:")
#st.write(Block)

#st.write("SHOP:")
#st.write(Shopfy)

#st.write("CRM:")
#st.write(Salesforce)

# Select the time series to plot
time_series = st.sidebar.selectbox("Select a time series", ["Open", "High", "Low", "Close", "Volume"])

if selected_stock == "Block":
    X = Block[time_series].to_numpy().reshape(-1, 1)
    y = Block["Close"].to_numpy()
elif selected_stock == "Shopfy":
   X = Shopfy[time_series].to_numpy().reshape(-1, 1)
   y = Shopfy["Close"].to_numpy()
else:
    X = Salesforce[time_series].to_numpy().reshape(-1, 1)
    y = Salesforce["Close"].to_numpy()

# Extract the time series from the financial data
if selected_stock == "Block":
    series = Block[time_series]
elif selected_stock == "Shopfy":
   series = Shopfy[time_series]
else:
    series = Salesforce[time_series]

st.line_chart(series)

#Now, i want to create a table only with the last price
df_last_prices = pd.DataFrame([{"Ticker": "Block", 'Price': Block['Close'].iloc[-1]},
                                {"Ticker": "Shopfy", "Price": Shopfy['Close'].iloc[-1]},
                                {"Ticker": "Salesforce", "Price": Salesforce["Close"].iloc[-1]}])


# Display the dataframe as a table in Streamlit
st.dataframe(df_last_prices)

#Set the alert
# Create an alert button
if st.button('Check decision according to max and min'):
    # Get the last prices for the stocks
    tickers = ['SQ', 'SHOP', 'CRM']
    min_prices = [60, 25, 125]
    max_prices = [100, 78, 190]
    last_prices = {}

    for i, ticker in enumerate(tickers):
        # Get the stock info
        stock = yf.Ticker(ticker)
        last_price = stock.info['regularMarketPrice']
        last_prices[ticker] = last_price

        # Check if the last price is below the min price
        if last_price < min_prices[i]:
            st.warning(f'ALERT: {ticker} is trading at {last_price}, which is below the threshold of {min_prices[i]}. Consider buying.')

        # Check if the last price is above the max price
        elif last_price > max_prices[i]:
            st.warning(f'ALERT: {ticker} is trading at {last_price}, which is above the target price of {max_prices[i]}. Consider selling.')

#Analyze what move to do according to Bollinger Bands:
#Calculate the rolling mean and standard deviation for each stock.
# Calculate the rolling mean and standard deviation for each stock

Block['rolling_mean'] = Block['Close'].rolling(20).mean()
Block['rolling_std'] = Block['Close'].rolling(20).std()
Shopfy['rolling_mean'] = Shopfy['Close'].rolling(20).mean()
Shopfy['rolling_std'] = Shopfy['Close'].rolling(20).std()
Salesforce['rolling_mean'] = Salesforce['Close'].rolling(20).mean()
Salesforce['rolling_std'] = Salesforce['Close'].rolling(20).std()

# Calculate the upper and lower Bollinger Bands for each stock
Block['upper_band'] = Block['rolling_mean'] + 2 * Block['rolling_std']
Block['lower_band'] = Block['rolling_mean'] - 2 * Block['rolling_std']
Shopfy['upper_band'] = Shopfy['rolling_mean'] + 2 * Shopfy['rolling_std']
Shopfy['lower_band'] = Shopfy['rolling_mean'] - 2
Salesforce['upper_band'] = Salesforce['rolling_mean'] + 2 * Salesforce['rolling_std']
Salesforce['lower_band'] = Salesforce['rolling_mean'] - 2

st.sidebar.text("Analysis")

# Create a dropdown widget to select the stock
selected_stock = st.sidebar.selectbox("Select a stock for Bollinger Bands Analysis", ["Block", "Shopfy", "Salesforce"])

# Create an alert button
if st.button("Check decision according to Bollinger Bands"):
  # Check the current price of the selected stock
  if selected_stock == "Block":
    current_price = Block['Close'].iloc[-1]
    upper_band = Block['upper_band'].iloc[-1]
    lower_band = Block['lower_band'].iloc[-1]
  elif selected_stock == "Shopfy":
    current_price = Shopfy['Close'].iloc[-1]
    upper_band = Shopfy['upper_band'].iloc[-1]
    lower_band = Shopfy['lower_band'].iloc[-1]
  else:
    current_price = Salesforce['Close'].iloc[-1]
    upper_band = Salesforce['upper_band'].iloc[-1]
    lower_band = Salesforce['lower_band'].iloc[-1]

  # Check if the current price is above the upper Bollinger Band
  if current_price > upper_band:
    st.warning(f"ALERT: {selected_stock} is trading at {current_price}, which is above the upper Bollinger Band of {upper_band}. Consider selling.")

  # Check if the current price is below the lower Bollinger Band
  elif current_price < lower_band:
    st.warning(f"ALERT: {selected_stock} is trading at {current_price}, which is below the lower Bollinger Band of {lower_band}. Consider buying.")
  else:
    st.success(f"{selected_stock} is trading within the Bollinger Bands. No action is required.")

#Moving Average Analysis

tickers = ['SQ', 'SHOP', 'CRM']
stocks = {}

#Download the financial data for the three stocks from Yahoo Finance. To do this, I can use a loop to iterate over the ticker symbols and retrieve the data for each stock:
for ticker in tickers:
    stock = yf.Ticker(ticker).history()
    stocks[ticker] = stock
#Calculate the moving average for each stock. You can use the rolling function from pandas to calculate the rolling mean for a specific time period:
for ticker, data in stocks.items():
    data['rolling_mean'] = data['Close'].rolling(20).mean()

# Create an alert button
if st.button("Check decision according to Moving Average"):
    for ticker, data in stocks.items():
        # Get the current price
        current_price = data['Close'].iloc[-1]
        # Get the moving average
        moving_average = data['rolling_mean'].iloc[-1]
        # Check if the current price is above the moving average
        if current_price > moving_average:
            st.warning(f'ALERT: {ticker} is trading at {current_price}, which is above the moving average of {moving_average}. Consider selling.')
        # Check if the current price is below the moving average
        elif current_price < moving_average:
            st.warning(f'ALERT: {ticker} is trading at {current_price}, which is below the moving average of {moving_average}. Consider buying.')
        # If the current price is equal to the moving average, do nothing
        else:
            st.info(f'{ticker} is trading at {current_price}, which is equal to the moving average of {moving_average}. Do nothing.')
