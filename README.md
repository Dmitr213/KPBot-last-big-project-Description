# KPBot-last-big-project-Description

Это мой большой и полноценный проект, написанный на 99% мной. Точно не считал, но в нём более 10 тысяч строк кода и десятки файлов с множеством функционала

Что бы посмотреть полный код этого проекта нужно написать мне (например в телеге @Sindoray), я открою доступ




# Описание

Система работает на связке Telegram + DB Google Firestore + Google Funсtions.

Для периодического запуска и триггеров функций используется Google PubSub / Scheduler.

Система основана на абстрагировании чата, канала и контента магазина до единого типа `Product`, доступ к которому предоставляется путем создания объекта `Access`, все транзакции записываются как объект `Transaction`. Пользователь представлен объектом `User`, провайдер платежей - `Provider`, также существует ряд вспомогательных объектов

### Telegram

Чат-бот и админ-чат, из которых бот принимает команды

### Firestore DB

Root базы всегда в документе `db/v1`

`users` коллекция пользователей `User`

`product` коллекция продуктов `Product`

`accesses` коллекция доступов `Access`

`providers` коллекция платежных провайдеров `Provider`

`transactions` коллекция транзакций `Transaction`

`notifications` коллекция нотификаций об истечении подписки

`messages` коллекция сообщений для рассылки

`menu` один документ `Menu`

`settings` прочие настройки

`utils` вспомогательные документы

`help` контент команды `/help`

`utils/currency_conversion` документ, где хранится актуальный курс рубля для оптимизации запросов к API [https://coinmarketcap.com/](https://coinmarketcap.com/)

`utils/transactions_events` коллекция событий в модели `Transaction` (фильтр повторных событий - однособытийность не гарантирована базой). Для этой коллекции включен TTL.

### Google Functions

`main_bot` основная функция, принимает webhook от телеграмма. Основной функционал находится в этой функции

`check_subscription` периодически проверяет `Access` c `accessType == subscription` и обрабатывает случаи истекшей подписки 

`payment_completion` обрабатывает завершение платежа, триггерится из Firestore при изменении `Transaction.status` на `success`, абстрагирована от провайдера

`yoomoney_callback` обрабатывает callback платежа Yoomoney

`cryptopay_callback` обрабатывает callback CryptoPay (CryptoBot)

`check_transaction` периодически проверяет транзакции в TonWallet

`notif_subscription_expired` периодически предупреждает пользователей имеющих подписку об ее истечении примерно за 72 и за 24 часа

`messenger` периодически обрабатывает сообщения для рассылки пользователям

`send_menu_to_all` периодически обрабатывает меню для рассылки пользователям

### Google PubSub

`check_subscription` триггерит функцию `check_subscription`. Используется Google Scheduler

`check_transaction` триггерит функцию `check_transaction`. Используется Google Scheduler

`notif_subscription_expired` триггерит функцию `notif_subscription_expired`. Используется Google Scheduler

`messenger` триггерит функцию `messenger`. Используется Google Scheduler

`send_menu_to_all` триггерит функцию `send_menu_to_all`. Используется Google Scheduler

### Google Scheduler

`check_subscription` запускает топик `check_subscription` раз в час `0 * * * * (Etc/UTC)`

`check_transaction` запускает топик `check_transaction` раз в 5 минут `*/5 * * * * (Etc/UTC)`

`notif_subscription_expired` запускает топик `notif_subscription_expired` раз в 4 часа `0 */4 * * * (Etc/UTC)`

`messenger` запускает топик `messenger` раз в 10 минут `0/10 * * * * (Etc/UTC)`

`send_menu_to_all` запускает топик `send_menu_to_all` раз в 15 минут `0/15 * * * * (Etc/UTC)`

### Платежные провайдеры

Подключены 4 платежных провайдера. Подключены как оплаты через кошельки, так и через крипту. Более подробно описано в полном репозитории с кодом.

### Конвертер валют

Для RUB и TON используется конвертация из EUR. Работает через API [https://coinmarketcap.com/api/](https://coinmarketcap.com/api/)

### Shop

Магазин состоит из двух частей - публичный канал-витрина и закрытый канал-хранилище.

Продукты типа content привязаны к постам в канале-хранилище и пересылаются пользователю в бота при покупке.
