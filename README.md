# Btcbot2
# Ready Files for BTC Auto Trading Bot

## File 1: app.py

```python
from flask import Flask, request, jsonify
from binance.client import Client
import os
import requests

app = Flask(__name__)

API_KEY = os.getenv("BINANCE_API_KEY")
API_SECRET = os.getenv("BINANCE_API_SECRET")

client = Client(API_KEY, API_SECRET)

BOT_TOKEN = os.getenv("TG_BOT_TOKEN")
CHAT_ID = os.getenv("TG_CHAT_ID")

last_signal = None

def send_telegram(msg):
    if BOT_TOKEN and CHAT_ID:
        url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
        requests.post(url, data={"chat_id": CHAT_ID, "text": msg})

def get_btc_balance():
    bal = client.get_asset_balance(asset='BTC')
    return float(bal['free'])

@app.route("/")
def home():
    return "BTC Auto Bot Running"

@app.route("/webhook", methods=["POST"])
def webhook():
    global last_signal

    data = request.json
    signal = data.get("action")

    if signal == last_signal:
        return jsonify({"msg":"Duplicate ignored"})

    try:
        if signal == "buy":
            btc = get_btc_balance()

            if btc > 0.00001:
                return jsonify({"msg":"Already holding BTC"})

            order = client.order_market_buy(
                symbol="BTCUSDT",
                quoteOrderQty=50
            )

            send_telegram("BUY Executed")
            last_signal = "buy"
            return jsonify(order)

        elif signal == "sell":
            btc = get_btc_balance()

            if btc < 0.00001:
                return jsonify({"msg":"No BTC to sell"})

            order = client.order_market_sell(
                symbol="BTCUSDT",
                quantity=round(btc,5)
            )

            send_telegram("SELL Executed")
            last_signal = "sell"
            return jsonify(order)

        return jsonify({"msg":"Invalid Signal"})

    except Exception as e:
        send_telegram(f"Error: {str(e)}")
        return jsonify({"error": str(e)})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

## File 2: requirements.txt

```txt
flask
python-binance
requests
gunicorn
```

---

## Render Start Command

```txt
gunicorn app:app
```

---

## Webhook URL Format

```txt
https://yourapp.onrender.com/webhook
```

---

## TradingView BUY Message

```json
{"action":"buy"}
```

## TradingView SELL Message

```json
{"action":"sell"}
```
