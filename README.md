# tg-bot-translate
from telebot import TeleBot
from telebot import types
import requests
import sqlite3


IAM_TOKEN = "AQVNzqTEphXyWg4NljgVvHkV7f8cnEbgpVo2Wvts"
folder_id = "b1g4ntevkh4c9ld7mvhl"


def translate(text: str, target_language="ru"):
    body = {
        "targetLanguageCode": target_language,
        "texts": text,
        "folderId": folder_id,
    }

    headers = {
        "Content-Type": "application/json",
        "Authorization": "Api-Key {0}".format(IAM_TOKEN),
    }

    response = requests.post(
        "https://translate.api.cloud.yandex.net/translate/v2/translate",
        json=body,
        headers=headers,
    )

    data = response.json()
    text = data.get("translations")[0].get("text")
    print(text)
    return text


connection = sqlite3.connect("text.db")

cursor_object = connection.execute(
  """
    CREATE TABLE IF NOT EXISTS trns_text (
        id INTEGER PRIMARY KEY,
        orig NULL,
        trns NULL
    )
  """
)

def save(text, t_rns):
    
    connection = sqlite3.connect("text.db")
    cursor_object = connection.execute(
        """
            INSERT INTO trns_text(orig, trns)
            VALUES(?, ?)

        """,
        [text, t_rns]
    )
    connection.commit()

TOKEN = 'hahahha'
bot = TeleBot(TOKEN)


@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(message.chat.id, "Привет! Я бот-переводчик. Напиши мне текст, а я переведу его.")

@bot.message_handler(func=lambda message: True)
def translate_text(message):
    text = message.text
    t_rns = f'{translate(text)}'
    bot.send_message(message.chat.id, t_rns)
    save(text, t_rns)


print("server start")
bot.polling(none_stop=True)
connection.close()
