import telebot
import requests
from datetime import datetime, timedelta
import json
import os

# ===== Config =====
BOT_API_KEY = "7284426053:AAEtdE6HDALM3y7OWJ3VyMlF1OOAaNd3BPg"
FOOTBALL_API_KEY = "1b95d3be100e0d6872959a0f0265f0d6"
LEAGUES = {
    "Premier League": 39,
    "La Liga": 140,
    "Champions League": 2
}
SEASON = 2024
USER_FILE = "users.json"

bot = telebot.TeleBot(BOT_API_KEY)
HEADERS = {
    "X-RapidAPI-Key": FOOTBALL_API_KEY,
    "X-RapidAPI-Host": "api-football-v1.p.rapidapi.com"
}

# ===== User Tracking =====
def load_users():
    if os.path.exists(USER_FILE):
        with open(USER_FILE, "r") as f:
            return json.load(f)
    return []

def save_users(users):
    with open(USER_FILE, "w") as f:
        json.dump(users, f)

def add_user(user_id):
    users = load_users()
    if user_id not in users:
        users.append(user_id)
        save_users(users)

# ===== API Call =====
def fetch_fixtures(date):
    results = []
    for league_name, league_id in LEAGUES.items():
        url = "https://api-football-v1.p.rapidapi.com/v3/fixtures"
        params = {
            "league": league_id,
            "season": SEASON,
            "date": date
        }
        resp = requests.get(url, headers=HEADERS, params=params)
        if resp.status_code == 200:
            data = resp.json()
            if "response" in data:
                for f in data["response"]:
                    f["league_name"] = league_name
                    results.append(f)
    return results

# ===== AI Prediction Logic =====
def predict_match(fixture):
    home = fixture["teams"]["home"]["name"]
    away = fixture["teams"]["away"]["name"]
    league = fixture.get("league_name", "Unknown League")

    # Use some basic stats and odds for prediction
    tip_lines = []
    tip_lines.append(f"League: {league}")
    tip_lines.append(f"{home} vs {away}")

    # Odds and goals info (use some dummy or safe fallback)
    odds = fixture.get("odds", {})
    # RapidAPI football odds sometimes not included in free plan, so let's fallback

    # Example simple rules:

    # Over/Under (dummy logic, because real averages require extra requests)
    tip_lines.append("Tip: Over 2.5 Goals (Assumed)")

    # W1/W2 based on league rank (dummy, because rank sometimes missing)
    home_rank = fixture.get("league", {}).get("rank", None)
    away_rank = fixture.get("league", {}).get("rank", None)
    if home_rank and away_rank:
        if home_rank < away_rank:
            tip_lines.append("Tip: Home Win (W1)")
        elif away_rank < home_rank:
            tip_lines.append("Tip: Away Win (W2)")
        else:
            tip_lines.append("Tip: Draw or Double Chance")

    # BTTS (Both Teams To Score) dummy logic
    tip_lines.append("BTTS: Yes")

    # Handicap dummy
    tip_lines.append("Handicap: Home -1")

    return "\n".join(tip_lines)

# ===== Generate Prediction Message for the Day =====
def generate_daily_tips(date):
    fixtures = fetch_fixtures(date)
    if not fixtures:
        return f"No matches found for {date}."

    messages = []
    for f in fixtures:
        pred = predict_match(f)
        messages.append(pred)
        messages.append("-" * 30)
    return "\n".join(messages)

# ===== Telegram Commands =====
@bot.message_handler(commands=["start"])
def start_command(message):
    add_user(message.chat.id)
    welcome_text = ("Welcome to AI Football Accumulator Bot!\n"
                    "Commands:\n"
                    "/predict - Get today's AI predictions\n"
                    "/tomorrow - Get tomorrow's predictions\n")
    bot.send_message(message.chat.id, welcome_text)

@bot.message_handler(commands=["predict"])
def predict_command(message):
    add_user(message.chat.id)
    date = datetime.now().strftime("%Y-%m-%d")
    bot.send_message(message.chat.id, "Fetching today's predictions...")
    tips = generate_daily_tips(date)
    bot.send_message(message.chat.id, tips)

@bot.message_handler(commands=["tomorrow"])
def tomorrow_command(message):
    add_user(message.chat.id)
    date = (datetime.now() + timedelta(days=1)).strftime("%Y-%m-%d")
    bot.send_message(message.chat.id, "Fetching tomorrow's predictions...")
    tips = generate_daily_tips(date)
    bot.send_message(message.chat.id, tips)

# ===== Auto Send Daily Tips (To all users) =====
def auto_send_daily_tips():
    users = load_users()
    date = datetime.now().strftime("%Y-%m-%d")
    tips = generate_daily_tips(date)
    for user_id in users:
        try:
            bot.send_message(user_id, f"Daily Football Tips for {date}:\n\n{tips}")
        except Exception as e:
            print(f"Failed to send message to {user_id}: {e}")

# ===== Run Bot =====
if __name__ == "__main__":
    # For PythonAnywhere scheduler, run this script daily with command:
    # python3 football_bot.py send_daily
    import sys
    if len(sys.argv) > 1 and sys.argv[1] == "send_daily":
        auto_send_daily_tips()
    else:
        bot.polling(none_stop=True)
