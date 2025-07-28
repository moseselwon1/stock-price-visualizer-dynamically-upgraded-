import streamlit as st
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

# -------------------------------
# 🌐 Page Configuration
# -------------------------------
st.set_page_config(page_title="Stock Price Visualizer Pro", layout="wide")

# -------------------------------
# 🎛 Sidebar Inputs
# -------------------------------
st.sidebar.title("📊 Stock Price Visualizer Pro")
ticker = st.sidebar.text_input("Enter Ticker Symbol (e.g. AAPL, TSLA)", value="AAPL")

start_date = st.sidebar.date_input(
    "Start Date", value=datetime.now() - timedelta(days=180)
)
end_date = st.sidebar.date_input("End Date", value=datetime.now())

moving_averages = st.sidebar.multiselect(
    "Select Moving Averages (Days)",
    options=[10, 20, 50, 100, 200],
    default=[20, 50, 100]
)

# -------------------------------
# 📥 Load Price Data
# -------------------------------
@st.cache_data
def load_data(ticker, start, end):
    df = yf.download(ticker, start=start, end=end)
    return df

df = load_data(ticker, start_date, end_date)

# -------------------------------
# ⚠️ Handle Missing Data
# -------------------------------
if df.empty:
    st.warning("⚠️ No data found. Check the ticker symbol or date range.")
    st.stop()

# -------------------------------
# 📐 Calculate Moving Averages
# -------------------------------
for ma in moving_averages:
    df[f"MA{ma}"] = df["Close"].rolling(window=ma).mean()

# -------------------------------
# 📈 Plot Stock Chart
# -------------------------------
st.title(f"📈 {ticker.upper()} Stock Price Chart")
st.caption(f"Data from {start_date} to {end_date}")

fig, ax = plt.subplots(figsize=(12, 6))
ax.plot(df["Close"], label="Close Price", color="black", linewidth=2)

for ma in moving_averages:
    ax.plot(df[f"MA{ma}"], label=f"MA {ma}", linewidth=1.5)

ax.set_xlabel("Date")
ax.set_ylabel("Price ($)")
ax.grid(True)
ax.legend()
st.pyplot(fig)

# -------------------------------
# 📋 Show Recent Table
# -------------------------------
st.subheader("📋 Recent Price + MA Data")
st.dataframe(df.tail(20), use_container_width=True)
