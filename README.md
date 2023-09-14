# python
Всем привет, этот код для поиска крипто связок между биржами информацию пытался выдернуть из CoinMarketCap , но бот в телеграмме не реагирует на команду /arbitrage, в PyCharm выходит (Ошибка получения данных с CoinMarketCap API - Неверные идентификаторы бирж.) Хотя идентификаторы указал верно  для пробы две биржи Binance и ByBit, в чём проблема?
"""""""""""""""""""""""""""""""""""""""""""""
import telebot
import requests
from telebot import types

bot = telebot.TeleBot("Ключ Телеграмм")

min_diff = 0.05

api_key = "Ключ CoinMarketCap"  # Ваш API ключ от CoinMarketCap


@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(message.chat.id, "Привет! Я бот для поиска арбитража на криптобиржах.")


@bot.message_handler(commands=['help'])
def show_help(message):
    bot.send_message(message.chat.id, "Чтобы начать поиск арбитража, введите /arbitrage")


def get_exchange_ids():
    url = "https://pro-api.coinmarketcap.com/v1/exchanges/listings"
    headers = {'X-CMC_PRO_API_KEY': api_key}
    response = requests.get(url, headers=headers)
    data = response.json()
    if 'data' in data:
        exchanges = data['data']
        exchange_ids = {exchange['name']: exchange['id'] for exchange in exchanges}
        return exchange_ids
    else:
        print("Ошибка получения данных с CoinMarketCap API")
        return {}


def get_price_on_cmc(symbol, exchange_id):
    url = "https://pro-api.coinmarketcap.com/v1/cryptocurrency/quotes/latest"
    params = {'symbol': symbol, 'convert': 'USD', 'aux': f"market_id:{exchange_id}"}
    headers = {'X-CMC_PRO_API_KEY': api_key}
    response = requests.get(url, params=params, headers=headers)
    data = response.json()

    # Проверяем, есть ли ключ 'data' в ответе
    if 'data' in data:
        price = data['data'][symbol]['quote']['USD']['price']
        return price
    else:
        print("Ошибка получения данных с CoinMarketCap API")
        return None


def get_price_diff(symbol, exchange1, exchange2):
    exchange_ids = get_exchange_ids()

    if exchange1 not in exchange_ids or exchange2 not in exchange_ids:
        print("Неверные идентификаторы бирж.")
        return None

    price1 = get_price_on_cmc(symbol, exchange1)
    price2 = get_price_on_cmc(symbol, exchange2)

    if price1 is not None and price2 is not None:
        if price1 != price2:
            diff = abs(price1 - price2) / ((price1 + price2) / 2) * 100
            return diff
        else:
            print(f"Цена одинакова для монеты {symbol} на обеих биржах. Пропускаем.")
            return None
    else:
        print(f"Не удалось получить цену для монеты {symbol}")
        return None


@bot.message_handler(commands=['arbitrage'])
def arbitrage(message):
    for coin in ["BTC", "ETH", "XRP", "VET", "SOL", "MATIC", "BCH"]:
        price_diff = get_price_diff(coin, "Binance", "Bybit")
        # Замените "EXCHANGE1_ID" и "EXCHANGE2_ID" на идентификаторы бирж, которые вы хотите использовать

        if price_diff is not None and price_diff > min_diff:
            msg = f"Найден арбитраж для {coin}: разница в цене между EXCHANGE1 и EXCHANGE2 составляет {price_diff}%"
            bot.send_message(message.chat.id, msg)

bot.polling()
"""""""""""""""""""""""""""""""""""""""""""""""""
