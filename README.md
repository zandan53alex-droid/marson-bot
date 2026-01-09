import telebot
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton
from flask import Flask, request
import os
import random
from datetime import datetime, timedelta

app = Flask(__name__)

TELEGRAM_TOKEN = '8214345375:AAEBZPTMMXe3D0Fi59lSFnLV0cBIctcqNYk'
bot = telebot.TeleBot(TELEGRAM_TOKEN)

FOREX_PAIRS = [
    "AUD/CAD", "AUD/CHF", "AUD/JPY", "AUD/USD", 
    "CAD/CHF", "CAD/JPY", "CHF/JPY", "EUR/AUD", 
    "EUR/CAD", "EUR/CHF", "EUR/GBP", "EUR/JPY", 
    "EUR/USD", "GBP/AUD", "GBP/CAD", "GBP/CHF", 
    "GBP/JPY", "GBP/USD", "USD/CAD"
]

FOREX_OTC_PAIRS = [
    "EURUSD-OTC", "GBPUSD-OTC", "USDJPY-OTC", "AUDUSD-OTC", 
    "USDCHF-OTC", "USDCAD-OTC", "EURGBP-OTC", "GBPJPY-OTC"
]

CRYPTO_PAIRS = [
    "Avalanche OTC", "Bitcoin ETF OTC", "BNB OTC", 
    "Bitcoin OTC", "Polkadot OTC", "Chainlink OTC", 
    "Litecoin OTC", "Solana OTC", "Ethereum OTC"
]

user_pairs = {}
user_expirations = {}

def generate_signal():
    if random.random() > 0.48:
        return "ВВЕРХ"
    return "ВНИЗ"

@bot.message_handler(commands=['start'])
def start(message):
    markup = InlineKeyboardMarkup(row_width=2)
    markup.add(InlineKeyboardButton("📈 ФОРЕКС", callback_data="menu_forex"))
    markup.add(InlineKeyboardButton("💱 OTC ВАЛЮТА", callback_data="menu_otc"))
    markup.add(InlineKeyboardButton("₿ КРИПТА", callback_data="menu_crypto"))
    bot.send_message(message.chat.id, "🚀 Выберите актив:", reply_markup=markup)

@bot.callback_query_handler(func=lambda call: True)
def callback_query(call):
    if call.data == "menu_main":
        markup = InlineKeyboardMarkup(row_width=2)
        markup.add(InlineKeyboardButton("📈 ФОРЕКС", callback_data="menu_forex"))
        markup.add(InlineKeyboardButton("💱 OTC ВАЛЮТА", callback_data="menu_otc"))
        markup.add(InlineKeyboardButton("₿ КРИПТА", callback_data="menu_crypto"))
        bot.edit_message_text("🚀 Выберите актив:", call.message.chat.id, call.message.message_id, reply_markup=markup)

    elif call.data == "menu_forex":
        markup = InlineKeyboardMarkup(row_width=2)
        for i in range(0, len(FOREX_PAIRS

