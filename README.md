import streamlit as st
from openai import OpenAI
import base64
import os

st.set_page_config(
    page_title="Advanced Trading AI",
    page_icon="📈",
    layout="wide"
)

st.title("📈 Advanced Trading AI")
st.caption("Chart screenshot → AI Vision → Technical Analysis → Trade Setup")

# -----------------------------
# API 8f1fde7d-c5e1-4697-9032-eb303c283720
# -----------------------------

api_key = st.secrets.get("OPENAI_API_KEY", os.getenv("OPENAI_API_KEY"))

if not api_key:
    st.error("OPENAI_API_KEY missing hai. Replit Secrets mein API key add karo.")
    st.stop()

client = OpenAI(api_key=api_key)

# -----------------------------
# Sidebar
# -----------------------------

st.sidebar.header("⚙️ Analysis Settings")

market = st.sidebar.selectbox(
    "Market",
    [
        "Forex",
        "Crypto",
        "Stocks",
        "Indices",
        "Commodities",
        "Other"
    ]
)

risk = st.sidebar.selectbox(
    "Risk Profile",
    [
        "Conservative",
        "Balanced",
        "Aggressive"
    ]
)

analysis_depth = st.sidebar.selectbox(
    "Analysis Depth",
    [
        "Deep",
        "Very Deep"
    ]
)

st.sidebar.info(
    "AI screenshot mein visible information ko analyze karega. "
    "Jo indicator screenshot mein available nahi hai usko invent nahi karega."
)

# -----------------------------
# Upload
# -----------------------------

uploaded_file = st.file_uploader(
    "📷 Chart Screenshot Upload Karo",
    type=["png", "jpg", "jpeg", "webp"]
)

# -----------------------------
# Analysis Prompt
# -----------------------------

SYSTEM_PROMPT = """
You are an advanced professional technical-analysis assistant.

Your job is to analyze a trading-chart screenshot extremely carefully.

IMPORTANT:
Never invent indicator values that are not visible or derivable from the image.
If an indicator is not visible, write N/A.
Do not claim guaranteed accuracy.
Do not say that a trade is certain.

Analyze as many of the following as the screenshot supports:

PRICE ACTION
- Market structure
- Higher highs / higher lows
- Lower highs / lower lows
- Trend
- Break of structure
- Change of character
- Breakout
- Breakdown
- Retest
- Consolidation
- Liquidity zones
- Swing highs/lows

MOVING AVERAGES
- EMA 9
- EMA 20
- EMA 21
- EMA 50
- EMA 100
- EMA 200
- SMA where visible
- MA crossovers
- MA alignment
- Dynamic support/resistance

MOMENTUM
- RSI
- MACD
- Stochastic
- Stochastic RSI
- CCI
- ROC
- Momentum
- Williams %R
- ADX / DI

VOLATILITY
- ATR
- Bollinger Bands
- Keltner Channels
- Volatility expansion/contraction

VOLUME
- Volume
- Relative volume
- OBV
- VWAP
- VWMA
- Money Flow indicators
- Volume-price divergence

PRICE LEVELS
- Support
- Resistance
- Supply
- Demand
- Swing levels
- Breakout levels
- Retest levels

FIBONACCI
- Fibonacci retracement
- Fibonacci extension
- 0.236
- 0.382
- 0.500
- 0.618
- 0.786
- 1.000
- 1.272
- 1.618

PATTERNS
- Doji
- Engulfing
- Hammer
- Shooting star
- Pin bar
- Morning star
- Evening star
- Inside bar
- Three soldiers/crows
- Double top/bottom
- Head and shoulders
- Inverse head and shoulders
- Triangle
- Wedge
- Flag
- Pennant
- Rectangle
- Cup and handle
- Channels

MULTI-TIMEFRAME:
If multiple timeframes are visible, compare them.
If only one timeframe is visible, say that higher/lower timeframe confirmation is unavailable.

TRADE DECISION:

Return exactly this structure:

MARKET:
TIMEFRAME:
CURRENT PRICE:

OVERALL TREND:
BULLISH / BEARISH / SIDEWAYS / UNCLEAR

TRADE:
BUY / SELL / WAIT

SIGNAL STRENGTH:
0-100

CONFIDENCE:
0-100

ENTRY ZONE:
...

STOP LOSS:
...

TARGET 1:
...

TARGET 2:
...

TARGET 3:
...

RISK REWARD:
...

TECHNICAL SCORE:
Trend:
EMA:
RSI:
MACD:
Volume:
VWAP:
Support/Resistance:
Fibonacci:
Price Action:
Candlestick:
Pattern:
ADX:
Volatility:

KEY SUPPORT LEVELS:
...

KEY RESISTANCE LEVELS:
...

BULLISH CONFIRMATIONS:
...

BEARISH CONFIRMATIONS:
...

INVALIDATION:
...

WHY THIS TRADE:
Give concise but technically meaningful reasoning.

MISSING DATA:
List indicators that cannot be reliably evaluated from the screenshot.

IMPORTANT:
If signals conflict strongly, choose WAIT.
Do not force BUY or SELL.
Do not manufacture numerical values.
"""

# -----------------------------
# Analyze
# -----------------------------

if uploaded_file:

    image_bytes = uploaded_file.getvalue()

    st.image(
        image_bytes,
        caption="Uploaded Trading Chart",
        use_container_width=True
    )

    if st.button("🚀 ANALYZE CHART", use_container_width=True):

        with st.spinner("AI chart ko deeply analyze kar raha hai..."):

            try:

                base64_image = base64.b64encode(
                    image_bytes
                ).decode("utf-8")

                mime = uploaded_file.type

                response = client.responses.create(

                    model="gpt-5.6",

                    input=[
                        {
                            "role": "system",
                            "content": SYSTEM_PROMPT
                        },
                        {
                            "role": "user",
                            "content": [
                                {
                                    "type": "input_text",
                                    "text": f"""
Analyze this {market} trading chart.

Risk profile: {risk}

Analysis depth: {analysis_depth}

Do not guess missing indicator values.

I want a practical trade setup if the chart provides enough evidence.
Otherwise give WAIT.

Analyze the screenshot at high visual detail.
"""
                                },
                                {
                                    "type": "input_image",
                                    "image_url": (
                                        f"data:{mime};base64,{base64_image}"
                                    ),
                                    "detail": "high"
                                }
                            ]
                        }
                    ]
                )

                result = response.output_text

                st.success("✅ Analysis Complete")

                st.markdown("## 📊 AI Trading Analysis")

                st.markdown(result)

                st.warning(
                    "⚠️ This is technical analysis, not a guaranteed prediction. "
                    "Always validate with live market data and risk management."
                )

            except Exception as e:

                st.error("Analysis mein error aaya.")

                st.code(str(e))
