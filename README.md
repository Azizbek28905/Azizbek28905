- 👋 Hi, I’m @Azizbek28905
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Azizbek28905/Azizbek28905 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
from threading import get_ident
from telegram import Update, ReplyKeyboardMarkup, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext, ConversationHandler
import math  # Import the math module



keep_alive()

TOKEN = '8065872471:AAF3N6q8W2n6LjLckcZVJ5EsiAZJvSGPqgo'

ADMENS = [1325023990, 9903205321]  # Fixed the leading zero issue

# Majburiy obuna bo`lishi kerak bo`lgan kanal
CHANNEL_ID = " t.me/ast_communityy "

# /start komandasi
def start(update: Update, context: CallbackContext):
    user = update.message.from_user
    chat_id = update.message.chat_id

    # Majburiy obuna bo`lishi kerak bo`lgan kanal
    CHANNEL_ID = " t.me/ast_communityy "
    
    # Foydalanuvchi kanalga obuna bo‘lganmi yoki yo‘qligini tekshirish
    member_status = context.bot.get_chat_member(2283229639, chat_id).status
    if member_status in ["member", "administrator", "creator"]:
        update.message.reply_text("✅ Siz kanalga obuna bo‘lgansiz! Botdan foydalanishingiz mumkin.")
    else: 
        # Kanalga obuna bo‘lish uchun xabar
        keyboard = [[InlineKeyboardButton("📢 Kanalga qo‘shilish", url=f"https://t.me/{str(2283229639)[1:]}")],
                    [InlineKeyboardButton("✅ Tekshirish", callback_data="check_sub")]]
        reply_markup = InlineKeyboardMarkup(keyboard)

        update.message.reply_text(
            "⚠️ Botdan foydalanish uchun quyidagi kanalga qo‘shiling:\n"
            f"👉 {2283229639}\n\n"
            "✅ Kanalga qo‘shilgandan keyin pastdagi 'Tekshirish' tugmasini bosing.",
            reply_markup=reply_markup
        )

# Bosqichlar
TREND, MAX, MIN, CANDLES = range(4)

def main():
    """Botni ishga tushirish"""
    updater = Updater(TOKEN, use_context=True)  # Use TOKEN variable
    dp = updater.dispatcher

    conv_handler = ConversationHandler(
        entry_points=[CommandHandler("start", start)],
        states={
            TREND: [MessageHandler(Filters.text & ~Filters.command, select_trend)],
            MAX: [MessageHandler(Filters.text & ~Filters.command, get_max)],
            MIN: [MessageHandler(Filters.text & ~Filters.command, get_min)],
            CANDLES: [MessageHandler(Filters.text & ~Filters.command, get_candles)],
        },
        fallbacks=[CommandHandler("cancel", cancel)]
    )

    dp.add_handler(conv_handler)

    updater.start_polling()
    updater.idle()

# Trend tanlash tugmalari
trend_buttons = ReplyKeyboardMarkup([["Buy", "Sell"]], one_time_keyboard=True, resize_keyboard=True)

def start(update: Update, context: CallbackContext):
    update.message.reply_text("Assalomu alaykum! Narx yo`nalishini tanlang? 'Buy' yoki 'Sell'?", reply_markup=trend_buttons)
    return TREND

def select_trend(update: Update, context: CallbackContext):
    trend = update.message.text
    if trend not in ["Buy", "Sell"]:
        update.message.reply_text("Iltimos, faqat 'Buy' yoki 'Sell' ni tanlang.")
        return TREND

    context.user_data["trend"] = trend
    update.message.reply_text(" ☑ Maksimum narxni yuboring:")
    return MAX

def get_max(update: Update, context: CallbackContext):
    try:
        context.user_data["max"] = float(update.message.text)
    except ValueError:
        update.message.reply_text("Iltimos, faqat son kiriting!")
        return MAX

    update.message.reply_text(" ✔ Minimum narxni yuboring:")
    return MIN

def get_min(update: Update, context: CallbackContext):
    try:
        context.user_data["min"] = float(update.message.text)
    except ValueError:
        update.message.reply_text("Iltimos, faqat son kiriting!")
        return MIN

    update.message.reply_text("Maksimum va minimum orasidagi shamchalar sonini yuboring:")
    return CANDLES

def get_candles(update: Update, context: CallbackContext):
    try:
        context.user_data["candles"] = float(update.message.text)
    except ValueError:
        update.message.reply_text("Iltimos, faqat son kiriting!")
        return CANDLES

    trend = context.user_data["trend"]
    max_price = context.user_data["max"]
    min_price = context.user_data["min"]
    candles = context.user_data["candles"]

    diff = (max_price - min_price) / candles
    if trend == "Sell":
        results = [
            max_price - ((diff * 0.5) * candles),
            max_price - ((diff * 2) * 2),
            max_price - ((diff * 0.33) * candles),
            max_price - ((diff * 3) * 3),
            max_price - ((diff * 0.25) * candles),
            max_price - ((diff * 4) * 4),
             (((math.sqrt(max_price) - math.sqrt(min_price)) * 180) / 360 - math.sqrt(min_price)) ** 2,
        ]
    else:  # Buy trend
        results = [
            min_price + ((diff * 0.5) * candles),
            min_price + ((diff * 2) * 2),
            min_price + ((diff * 0.33) * candles),
            min_price + ((diff * 3) * 3),
            min_price + ((diff * 0.25) * candles),
            min_price + ((diff * 4) * 4),
            (((math.sqrt(max_price) - math.sqrt(min_price)) * 180) / 360 + math.sqrt(max_price)) ** 2,
        ]

    # Natijalarni formatlash
    response = f"📊 Natijalar ({trend} trend):\n\n"
    response += f"🔹 <b>{results[0]:.4f}</b>\n"
    response += f"🔹 <b>{results[1]:.4f}</b>\n"
    response += f"🔸 {results[2]:.4f}\n"
    response += f"🔸 {results[3]:.4f}\n"
    response += f"🔸 {results[4]:.4f}\n"
    response += f"🔸 {results[5]:.4f}\n"
    response += f"T/p {results[6]:.4f}\n"

    update.message.reply_text(response, parse_mode="HTML")
    return ConversationHandler.END

def cancel(update: Update, context: CallbackContext):
    update.message.reply_text("Amal bekor qilindi.")
    return ConversationHandler.END

if __name__ == "__main__":
    main()

from keep_alive import keep_alive  # keep_alive.py faylidan import qilish

if __name__ == '__main__':
   
 
