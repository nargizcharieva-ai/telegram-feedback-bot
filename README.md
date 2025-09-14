import telebot
import gspread
from oauth2client.service_account import ServiceAccountCredentials
import os
import json

# === Конфигурация ===
TELEGRAM_TOKEN = "8265806421:AAHvRrCBCADVboVJBvO5MTgBH6LNQ9ihQ3M"
SHEET_NAME = "Restaurant Feedback"  # название вашей Google таблицы

# Получаем данные сервисного аккаунта из переменной окружения
service_account_info = json.loads(os.environ["GOOGLE_CREDS_JSON"])
scope = ["https://spreadsheets.google.com/feeds", "https://www.googleapis.com/auth/drive"]
creds = ServiceAccountCredentials.from_json_keyfile_dict(service_account_info, scope)
client = gspread.authorize(creds)
sheet = client.open(SHEET_NAME).sheet1

# === Телеграм-бот ===
bot = telebot.TeleBot(TELEGRAM_TOKEN)

@bot.message_handler(content_types=['text'])
def handle_message(message):
    feedback = message.text
    username = message.from_user.username if message.from_user.username else ""
    user_id = message.from_user.id
    sheet.append_row([str(user_id), username, feedback])
    bot.reply_to(message, "Спасибо за ваш отзыв! Он отправлен в ресторан.")

if __name__ == "__main__":
    bot.polling(none_stop=True)
