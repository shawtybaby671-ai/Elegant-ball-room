import asyncio
import logging
import re
import json
import hmac
import hashlib
import threading
import random
from datetime import datetime, timedelta
from random import shuffle, sample
from io import BytesIO

from aiogram import Bot, Dispatcher, types
from aiogram.filters import Command
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton
from flask import Flask, request, jsonify, abort
from replit import db
from PIL import Image, ImageDraw, ImageFont

logging.basicConfig(level=logging.INFO)

# Load from environment or fallback (Render uses env vars)
import os
BOT_TOKEN = os.getenv("BOT_TOKEN", "YOUR_BOT_TOKEN")
ADMIN_ID = int(os.getenv("ADMIN_ID", "123456789"))

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher()

app = Flask(__name__)

# Constants
TIP_REGEX = re.compile(r"Amount\s*:\s*([\d,]+\.?\d*)\s*USDT", re.IGNORECASE)
BINGOPAYS_USERNAME = "bingopays"
DIAMONDS_PER_USDT = 1000
GAME_INTERVAL = 10  # average seconds per call

# DB keys
CURRENT_GAME = "current_game"
PLAYER_PREFIX = "player_"
JACKPOT_POOL = "jackpot_pool"
JACKPOT_LIMIT = "jackpot_limit"
GAME_COUNT = "game_count"

# Initialize jackpot
if JACKPOT_POOL not in db:
    db[JACKPOT_POOL] = 0
    db[JACKPOT_LIMIT] = 50
    db[GAME_COUNT] = 0

# Patterns (expanded example)
PATTERNS = {
    "single_line": {"name": "Single Line", "difficulty": "easy", "wilds": 0, "marks": 5},
    "four_corners": {"name": "Four Corners", "difficulty": "medium", "wilds": 1, "marks": 4},
    "full_house": {"name": "Full House", "difficulty": "hard", "wilds": 2, "marks": 25},
    # Add more patterns as needed
}

# Helper functions
def get_player(user_id):
    key = f"{PLAYER_PREFIX}{user_id}"
    if key not in db:
        db[key] = {
            "diamonds": 50,
            "xp": 0,
            "vip": 1,
            "referral_parent": None,
            "aff_earnings": 0,
            "last_diamond": None,
            "cards": []
        }
    return db[key]

def validate_init_data(init_data: str):
    # Basic validation – improve with full HMAC in production
    if not init_data:
        return False
    # Extract user from init_data (simplified)
    try:
        data = json.loads(hashlib.sha256(init_data.encode()).hexdigest())  # Placeholder
        return True
    except:
        return False

def generate_card(double_action=False):
    card = []
    ranges = [(1,15), (16,30), (31,45), (46,60), (61,75)]
    for r in ranges:
        nums = random.sample(range(r[0], r[1]+1), 5)
        if double_action:
            card.append([(n, random.choice(range(r[0], r[1]+1))) for n in nums])
        else:
            card.append(nums)
    card[2][2] = "FREE"
    return card

def pattern_complete(card, pattern, called, wilds_outside):
    # Simplified Full House check – expand for all patterns
    if pattern == "full_house":
        marked = 24  # exclude free
        for row in card:
            for cell in row:
                if cell != "FREE" and cell not in called:
                    marked -= 1
        return marked == 24
    # Add other pattern logic
    return False

def calculate_prize_pool(game):
    # Simplified – use dynamic allocation
    return 1000  # Example

def get_near_win_tease(game):
    # Count players 1-3 away
    close = random.randint(0, 5)
    if close >= 3:
        return "Someone's getting hot – multiple close calls!"
    elif close >= 1:
        return "Someone's ONE away! Hold your breath!"
    return None

def create_ball_image(ball, letter):
    img = Image.new('RGB', (300, 300), color='#F5F5DC')
    draw = ImageDraw.Draw(img)
    draw.ellipse((30,30,270,270), fill='#D4AF37', outline='#8B0000', width=8)
    try:
        font = ImageFont.truetype("arial.ttf", 80)
    except:
        font = ImageFont.load_default()
    draw.text((150, 130), f"{letter}-{ball}", fill='#4A2C2A', font=font, anchor="mm")
    buf = BytesIO()
    img.save(buf, format='PNG')
    buf.seek(0)
    return buf

# Flask API endpoints
@app.route('/api/player', methods=['GET'])
def api_player():
    if not validate_init_data(request.headers.get('auth', '')):
        abort(401)
    # Extract user_id from init_data (implement parsing)
    user_id = 123456  # Placeholder
    player = get_player(user_id)
    return jsonify({
        "diamonds": player["diamonds"],
        "xp": player["xp"],
        "vip": player["vip"],
        "affEarnings": player["aff_earnings"],
        "canClaimFree": player["last_diamond"] is None or (datetime.now().timestamp() - player["last_diamond"]) > 604800
    })

# Add other API endpoints from previous parts (join_game, game_state, tip_caller, etc.)

# Group handlers
@dp.message(Command("play"))
async def cmd_play(message: types.Message):
    keyboard = InlineKeyboardMarkup(inline_keyboard=[[
        InlineKeyboardButton(text="🎟️ Enter the Bingo Hall", web_app=types.WebAppInfo(url="https://your-miniapp.onrender.com"))
    ]])
    await message.reply("Welcome to DegenFamous Bingo! Tap below to enter your personal hall.", reply_markup=keyboard)

@dp.message(Command("newgame"))
async def cmd_newgame(message: types.Message):
    if message.from_user.id != ADMIN_ID:
        await message.reply("Only the hall manager can start games!")
        return
    db[CURRENT_GAME] = {
        "chat_id": message.chat.id,
        "pattern": "single_line",
        "difficulty": "easy",
        "wilds": 0,
        "called": [],
        "players": {},
        "ball_bag": list(range(1,76)),
        "start_time": datetime.now().timestamp()
    }
    shuffle(db[CURRENT_GAME]["ball_bag"])
    await message.reply("🔔 New game starting! Pattern: Single Line\nJoin via the Mini App – good luck!")

# Game loop
async def game_loop():
    while True:
        game = db.get(CURRENT_GAME)
        if game and len(game["called"]) < 75:
            ball = game["ball_bag"].pop(0)
            game["called"].append(ball)
            db[CURRENT_GAME] = game

            letter = "BINGO"[(ball-1)//15]
            await bot.send_message(game["chat_id"], f"Calling **{letter}-{ball}**!", parse_mode="Markdown")
            img = create_ball_image(ball, letter)
            await bot.send_photo(game["chat_id"], photo=img)

            winners = []  # check_wins logic
            if winners:
                await handle_wins(winners, game)
            else:
                tease = get_near_win_tease(game)
                if tease:
                    await bot.send_message(game["chat_id"], tease)

            await asyncio.sleep(GAME_INTERVAL + random.uniform(-3, 3))
        else:
            await asyncio.sleep(5)

async def handle_wins(winners, game):
    pool = calculate_prize_pool(game)
    share = pool // len(winners)
    for user_id, _ in winners:
        player = get_player(user_id)
        player["diamonds"] += share
        db[f"{PLAYER_PREFIX}{user_id}"] = player
        await bot.send_message(user_id, f"🎉 BINGO! You win {share} diamonds!")
    await bot.send_message(game["chat_id"], f"🎊 We have {len(winners)} winner(s)! Congratulations!")

# Start tasks
asyncio.create_task(game_loop())

def run_flask():
    app.run(host='0.0.0.0', port=8080)

threading.Thread(target=run_flask, daemon=True).start()

if __name__ == "__main__":
    asyncio.run(main())
