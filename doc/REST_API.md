# REST API

## Открытие сделок

Ниже приведен пример запроса для открытия сделки. Не все параметры являются обязательными

Минимальный вариант с заданным размером ставки:

```json
{
	"symbol":"EURUSD",
	"duration":180,
	"direction":"SELL",
	"amount":50
}
```

Минимальный вариант с заданным размером ставки в виде процента от депозита:

```json
{
	"symbol":"EURUSD",
	"duration":180,
	"direction":"SELL",
	"rate":0.01
}
```

Перечисление всех параметров:

```json
{
	"symbol":"EURUSD",				// Имя валютной пары, обязательный параметр
	"duration":180,					// Длительность экспирации в секундах, обязательный параметр
	"direction":"SELL",				// Направление ставки, обязательный параметр 
	"payout_limit"0.78,				// Лимит процента выплат (не обязательный параметр)
	"id":567,						// Уникальный номер ставки (не обязательный параметр)
	"winrate":0.6,					// Винрейт (только для критерия Келли)
	"kelly_attenuation":0.2,		// Коэффициент ослабления критерия Келли (только для критерия Келли)
	"amount":50,					// Размер ставки (можно использовать другой параметр)
	"rate":0.01,					// Процент ставки от депозита (можно использовать другой параметр)
	"broker":"intrade.bar",			// Брокер (только если нужен конкретный брокер или заданное условие)
	"strategy_name":"BB+RSI",		// Имя стратегии или индикатора (не обязательный параметр)
	"signal_timestamp":1560540600,	// Время возникновения сигнала (не обязательный параметр)
	"note":"Для сигналов в группе", // Заметка для сделки (не обязательный параметр)
	"lifetime":5,					// Время жизни сделки (не обязательный параметр)
	"attempts":3,					// Количество попыток открыть сделку (не обязательный параметр)
}
```

Более подробное описание

* symbol - Имя символа (валютная пара, криптовалюта и т.д.). **Обязательный параметр**.
* duration - Длительность экспирации в секундах, обязательный параметр. Секунды выбраны для стандартизации времени. **Обязательный параметр**.
* direction - Направление ставки. Можно использовать слова BUY, PUT, CALL, SELL, UP, DN, DOWN, а так же числа 1 или -1. **Обязательный параметр**.
* payout_limit - Лимит процента выплат. Указывать, если нужно. По умолчанию соответствует настройкам в программе. 
* id - Уникальный номер ставки. Параметр может быть нужен для стратегий мани менеджмента. Данный номер будет возвращаться в ответах о состоянии сделки.
* winrate - Винрейт. Не обязательный параметр, только для критерия Келли. По умолчанию соответствует настройкам в программе. Нужен для расчета размера ставки от депозита.
* kelly_attenuation -Коэффициент ослабления критерия Келли. По умолчанию соответствует настройкам в программе.  Не обязательный параметр, только для критерия Келли. Нужен для расчета размера ставки от депозита.
* amount - Размер ставки. Если параметр указан, используется заданный размер. Иначе используются настройки в программе.
* rate - Процент ставки от депозита. Если параметр указан, используется заданный размер в виде процента от депозита. Иначе используются настройки в программе.
* broker - Брокер, где открыть сделку. По умолчанию используются настройки в программе, так что параметр можно не указывать. Если указать *best* - значит будут использованы все брокеры для поиска наилучшего входа в сделку. Параметр *duplication* продублирует сделку по всем доступным брокерам.
* strategy_name - Имя стратегии или индикатора. Может быть нужен для фильтра стратегии, если используется фильтрация стратегий. Также имя записывается в базу данных сделок.
* signal_timestamp - Время возникновения сигнала на открытие сделки. Указывать не обязательно. Нужно для замера задержек. Время строго UTC.
* note - Заметка для сделки. Записывается в базу данных сделок. Не обязательный параметр.
* lifetime - Время жизни сделки. По умолчанию соответствует настройкам в программе. Этот параметр влияет на то, сколько времени робот может пытаться открыть сделку у брокеров. Параметр не обязателен. Однако, если нужно несколько попыток открыть сделку в случае неудачи, а время ограничено, указывать обязательно. Также можно указать вместо числа слово *close*, что будет означать - "до закрытия бара". Если до закрытия бара еще есть время, робот может пытаться открыть сделку по наилучшему условию. Под одним баром подразумевается минутный бар.
* attempts - Количество попыток открыть сделку. По умолчанию соответствует настройкам в программе. 

## Состояние сделок

Чтобы получать состояние сделок, необходимо подписаться на состояния.

```json
{
	"subscription": {
		"deals":{
			"use":true,
		}
	}
}
```

Далее после открытия сделки будут приходить следующие сообщения

```json
{
	"update":{ // сообщение обновления состояния параметров
		"deals":[	// массив сделок
			{
				"id":567,					// ID сделки, который был указан в запросе
				"broker_id":123123123,		// ID сделки брокера
				"broker":"intrade.bar",		// Брокер, где была открыта сделка
				"balance_broker":950,		// Баланс брокера после открытия сделки
				"status":"open",			// состояния сделки: open, wait, close, error, cancel
				"result":"none",			// состояние результата сделки: win, loss, standoff, none
				"payout":0.82,				// процент выплат по сделке
				"amount":50,				// размер ставки
				"opening_time":1560540600,	// Время открытия сделки UTC
				"closing_time":1560540780,	// Время закрытия сделки UTC
				"opening_delay":1.2,		// Задержка на открытие сделки
				"signal_delay":1.25,	    // Задержка между сигналом и реальным временем открытия
			}
		]
	}
}
```