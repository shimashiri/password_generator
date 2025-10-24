# password_generator
from flask import Flask, render_template_string, request
import requests

app = Flask(__name__)

HTML = """
<!DOCTYPE html>
<html>
<head>
    <title>💸 Crypto Price Tracker</title>
    <style>
        body { font-family: Arial; background: #101820; color: #f1f1f1; text-align: center; padding: 50px; }
        .card { background: #1e293b; padding: 20px; border-radius: 10px; display: inline-block; }
        input, select { padding: 10px; margin: 5px; border-radius: 5px; border: none; }
        button { padding: 10px; background: #00bcd4; color: white; border: none; border-radius: 5px; cursor: pointer; }
        button:hover { background: #008ba3; }
    </style>
</head>
<body>
    <h1>💰 Crypto Price Tracker</h1>
    <form method="POST">
        <select name="symbol">
            <option value="bitcoin">Bitcoin</option>
            <option value="ethereum">Ethereum</option>
            <option value="dogecoin">Dogecoin</option>
            <option value="solana">Solana</option>
            <option value="cardano">Cardano</option>
        </select>
        <button type="submit">Get Price</button>
    </form>

    {% if price %}
    <div class="card">
        <h2>{{ name }}</h2>
        <p>💵 Current Price: <b>${{ price }}</b></p>
        <p>📊 24h Change: {{ change }}%</p>
    </div>
    {% endif %}
</body>
</html>
"""

def get_crypto_price(symbol):
    url = f"https://api.coingecko.com/api/v3/coins/markets"
    params = {"vs_currency": "usd", "ids": symbol}
    res = requests.get(url, params=params)
    if res.status_code == 200 and len(res.json()) > 0:
        data = res.json()[0]
        return {
            "name": data["name"],
            "price": round(data["current_price"], 2),
            "change": round(data["price_change_percentage_24h"], 2)
        }
    return None

@app.route("/", methods=["GET", "POST"])
def index():
    price_data = None
    if request.method == "POST":
        symbol = request.form["symbol"]
        price_data = get_crypto_price(symbol)
    return render_template_string(HTML, **(price_data or {}))

if __name__ == "__main__":
    app.run(debug=True)
