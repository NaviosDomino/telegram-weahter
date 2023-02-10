# telegram-weahter

import requests
import random
from config import tg_bot_token, open_weather_token
from aiogram import Bot, types
from aiogram.dispatcher import Dispatcher
from aiogram.utils import executor

bot = Bot(token=tg_bot_token)
dp = Dispatcher(bot)


@dp.message_handler(commands=['start'])
async def start_command(message: types.Message):
    await message.reply("Привет")


@dp.message_handler()
async def get_weather(message: types.Message):
    try:
        advice_list = [
            "Будьте уверены в себе и своих силах.",
            "Никогда не сдавайтесь.",
            "Будьте спокойны и уравновешенны.",
            "Радуйтесь каждый день.",
            "Будьте благодарны за то, что у вас есть.",
            "Никогда не останавливайтесь на достигнутом.",
            "Общайтесь с людьми и поддерживайте связи.",
            "Будьте добры и внимательны к другим."
        ]
        code_to_smile = {
            "Clear": "Ясно \U00002600",
            "Clouds": "Облачно \U00002601",
            "Rain": "Дождь \U00002614",
            "Drizzle": "Дождь \U00002614",
            "Thunderstorm": "Гроза \U000026A1",
            "Snow": "Снег \U0001F328",
            "Mist": "Туман \U0001F32B"
        }

        url = f"http://api.openweathermap.org/data/2.5/weather?q={message.text}&appid={open_weather_token}&units=metric"
        response = requests.get(url)
        data = response.json()
        city = data["name"]
        cur_weather = data["main"]["temp"]

        if cur_weather < 0:
            temp = "мороз"
        elif 0 < cur_weather < 10:
            temp = "холодно"
        elif 10 <= cur_weather < 17:
            temp = "прохладно"
        elif 17 <= cur_weather < 23:
            temp = "тепло"
        elif 23 <= cur_weather < 50:
            temp = "жарко"

        weather_description = data["weather"][0]["main"]
        if weather_description in code_to_smile:
            wd = code_to_smile[weather_description]
        else:
            wd = 'Посмотри в окно, не пойму что там за погода!'
        wind = data["wind"]["speed"]

        await message.reply(f"В {city} {temp}: {int(cur_weather)}С°\n{wd}\nВетер: {int(wind)} м/с\n"
                            f"Совет дня: {random.choice(advice_list)}")
    except:
        await message.reply("Проверьте название города")



if __name__ == '__main__':
    executor.start_polling(dp)
