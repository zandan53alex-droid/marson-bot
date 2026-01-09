import telebot
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton
import random
from datetime import datetime, timedelta

TELEGRAM_TOKEN = '8214345375:AAEBZPTMMXe3D0Fi59lSFnLV0cBIctcqNYk'

# ФОРЕКС ПАРЫ (новый раздел)
FOREX_PAIRS = [
    "AUD/CAD", "AUD/CHF", "AUD/JPY", "AUD/USD",
    "CAD/CHF", "CAD/JPY", "CHF/JPY", "EUR/AUD",
    "EUR/CAD", "EUR/CHF", "EUR/GBP", "EUR/JPY", "EUR/USD",
    "GBP/AUD", "GBP/CAD", "GBP/CHF", "GBP/JPY", "GBP/USD",
    "USD/CAD"
]

# ВАЛЮТНЫЕ OTC (убрали USDSEK, добавили новые)
FOREX_OTC_PAIRS = [
    "EURUSD-OTC", "GBPUSD-OTC", "USDJPY-OTC", "AUDUSD-OTC", 
    "USDCHF-OTC", "USDCAD-OTC", "EURGBP-OTC", "GBPJPY-OTC",
    "AUDJPY-OTC", "EURJPY-OTC", "EURCHF-OTC", "CADJPY-OTC",
    "AUDCHF-OTC", "NZDUSD-OTC", "USD/BRL OTC", "AED/CNY OTC"
]

# КРИПТОВАЛЮТЫ OTC
CRYPTO_PAIRS = [
    "Avalanche OTC", "Bitcoin ETF OTC", "BNB OTC", "Bitcoin OTC", 
    "Polkadot OTC", "Chainlink OTC", "Litecoin OTC", "Solana OTC", "Ethereum OTC"
]

bot = telebot.TeleBot(TELEGRAM_TOKEN)
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
    bot.send_message(message.chat.id, "Выбери категорию:", reply_markup=markup)

@bot.callback_query_handler(func=lambda call: True)
def callback_query(call):
    # ГЛАВНОЕ МЕНЮ
    if call.data == "menu_main":
        markup = InlineKeyboardMarkup(row_width=2)
        markup.add(InlineKeyboardButton("📈 ФОРЕКС", callback_data="menu_forex"))
        markup.add(InlineKeyboardButton("💱 OTC ВАЛЮТА", callback_data="menu_otc"))
        markup.add(InlineKeyboardButton("₿ КРИПТА", callback_data="menu_crypto"))
        bot.edit_message_text("Выбери категорию:", 
                             call.message.chat.id, call.message.message_id, 
                             reply_markup=markup)
    
    # ФОРЕКС ПАРЫ (2 столбца)
    elif call.data == "menu_forex":
        markup = InlineKeyboardMarkup(row_width=2)
        for i in range(0, len(FOREX_PAIRS), 2):
            if i+1 < len(FOREX_PAIRS):
                markup.add(InlineKeyboardButton(FOREX_PAIRS[i], callback_data=f"pair_{FOREX_PAIRS[i]}"),
                          InlineKeyboardButton(FOREX_PAIRS[i+1], callback_data=f"pair_{FOREX_PAIRS[i+1]}"))
            else:
                markup.add(InlineKeyboardButton(FOREX_PAIRS[i], callback_data=f"pair_{FOREX_PAIRS[i]}"))
        markup.add(InlineKeyboardButton("🔙 Главное меню", callback_data="menu_main"))
        bot.edit_message_text("📈 ФОРЕКС ПАРЫ:", 
                             call.message.chat.id, call.message.message_id, 
                             reply_markup=markup)
    
    # OTC ВАЛЮТА (2 столбца)
    elif call.data == "menu_otc":
        markup = InlineKeyboardMarkup(row_width=2)
        for i in range(0, len(FOREX_OTC_PAIRS), 2):
            if i+1 < len(FOREX_OTC_PAIRS):
                markup.add(InlineKeyboardButton(FOREX_OTC_PAIRS[i], callback_data=f"pair_{FOREX_OTC_PAIRS[i]}"),
                          InlineKeyboardButton(FOREX_OTC_PAIRS[i+1], callback_data=f"pair_{FOREX_OTC_PAIRS[i+1]}"))
            else:
                markup.add(InlineKeyboardButton(FOREX_OTC_PAIRS[i], callback_data=f"pair_{FOREX_OTC_PAIRS[i]}"))
        markup.add(InlineKeyboardButton("🔙 Главное меню", callback_data="menu_main"))
        bot.edit_message_text("💱 OTC ВАЛЮТНЫЕ ПАРЫ:", 
                             call.message.chat.id, call.message.message_id, 
                             reply_markup=markup)
    
    # КРИПТА (2 столбца)
    elif call.data == "menu_crypto":
        markup = InlineKeyboardMarkup(row_width=2)
        for i in range(0, len(CRYPTO_PAIRS), 2):
            if i+1 < len(CRYPTO_PAIRS):
                markup.add(InlineKeyboardButton(CRYPTO_PAIRS[i], callback_data=f"pair_{CRYPTO_PAIRS[i]}"),
                          InlineKeyboardButton(CRYPTO_PAIRS[i+1], callback_data=f"pair_{CRYPTO_PAIRS[i+1]}"))
            else:
                markup.add(InlineKeyboardButton(CRYPTO_PAIRS[i], callback_data=f"pair_{CRYPTO_PAIRS[i]}"))
        markup.add(InlineKeyboardButton("🔙 Главное меню", callback_data="menu_main"))
        bot.edit_message_text("₿ КРИПТОВАЛЮТЫ OTC:", 
                             call.message.chat.id, call.message.message_id, 
                             reply_markup=markup)
    
    # ВЫБОР ПАРЫ
    elif call.data.startswith("pair_"):
        pair = call.data[5:]
        user_pairs[call.message.chat.id] = pair
        
        markup = InlineKeyboardMarkup(row_width=2)
        markup.add(InlineKeyboardButton("1 МИН", callback_data=f"time_{call.message.chat.id}_1"))
        markup.add(InlineKeyboardButton("2 МИН", callback_data=f"time_{call.message.chat.id}_2"))
        markup.add(InlineKeyboardButton("3 МИН", callback_data=f"time_{call.message.chat.id}_3"))
        markup.add(InlineKeyboardButton("5 МИН", callback_data=f"time_{call.message.chat.id}_5"))
        
        bot.edit_message_text(f"✅ {pair}\nВыбери время экспирации:", 
                             call.message.chat.id, call.message.message_id, 
                             reply_markup=markup)
    
    # ВЫБОР ВРЕМЕНИ
    elif call.data.startswith("time_"):
        parts = call.data.split("_")
        chat_id = int(parts[1])
        minutes = int(parts[2])
        
        user_expirations[chat_id] = minutes
        pair = user_pairs.get(chat_id, "EUR/USD")
        
        direction = generate_signal()
        now = datetime.now()
        expire_time = now + timedelta(minutes=minutes)
        
        bot.answer_callback_query(call.id)
        bot.edit_message_text(
            f"🚀 СИГНАЛ {pair}\n"
            f"📈 НАПРАВЛЕНИЕ: **{direction}**\n"
            f"⏰ ЭКСПИРАЦИЯ: {minutes} МИН\n"
            f"🕐 ЗАКРЫТИЕ: {expire_time.strftime('%H:%M:%S')}",
            call.message.chat.id, call.message.message_id,
            parse_mode='Markdown'
        )

if __name__ == '__main__':
    print("🚀 MarsonTrade Bot запущен 24/7!")
    bot.polling(none_stop=True)


