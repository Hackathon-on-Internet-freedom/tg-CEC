# tg-stat --- сбор статистики о чём угодно

> Легко и сладостно говорить правду в лицо королю

# Что это такое и с чем его едят?

Пишем описание данных в json, выполняем пару команд и получаем бота для
периодического получения альтернативных данных и сайт, на котором видна
суммарная статистика для анализа.

- [Презентация на хакатоне ОЗИ - 7 июня 2020 г.](/docs/tg-cec-demo.pdf),
  демонстрационный [бот](https://t.me/cec_demo_bot) (может быть остановлен).
- После завершения хакатона проект превратился в бота, который использует
  гугл-таблицы для описания форм и сбора ответов на них:
  [сайт](https://gltronred.github.io/tg-form/),
  [репозиторий](https://sr.ht/~rd/tg-form/) и его
  [зеркало](https://github.com/gltronred/tg-form)

## Как это можно использовать?

- Представьте ["Белый счётчик"](/docs/cases.md#белый-счётчик), который не просто раз в три часа публикует,
  сколько людей на митинге, а складывает данные в децентрализованную сеть - и
  все желающие могут наблюдать в реальном времени, сколько человек пришли не
  просто в разных городах, но и даже через разные проходы.
- Представьте активистов с [самодельными станциями мониторинга воздуха](/docs/cases.md#мониторинг-воздуха), которые
  выкладывают результаты каждый час на общую карту.
- Представьте [медработников](/docs/cases.md#данные-о-covid), которые могут одним сообщением отправить статистику
  о свободных койках и новых случаях внебольничной пневмонии - давая возможность
  отслеживать эту информацию всем желающим.
- Представьте наблюдателей на выборах, которые нажатием одной кнопки (например,
  [такой](https://www.aliexpress.com/item/32973667926.html)) фиксируют время
  прихода голосующих на часы раньше официальной статистики, или проводят
  экзит-поллы, отправляя информацию в проверяемом виде.
- Представьте [ЗАГСы](/docs/cases.md#демография), которые могут сказать, сколько
  жителей в России в онлайн-режиме

В любом случае, [результаты показываются](/docs/cases.md#показ-результатов) на
сайте, где можно агрегировать данные по регионам, фильтровать и скачивать
исходные данные.

## Что уже работает?

- конфигурирование бота
- бот в telegram
- [собранные данные](/example-geo.csv) о координатах центров муниципалитетов и
  регионов России; [процедура сбора](/docs/geo.md) задокументирована
- сохранение результатов в Google Sheets
- цепочка хэшей в результатах
- зашифрование данных в результатах
- подпись данных в результатах

## Что ещё не работает?

- удобное создание конфигураций
- проверка правильности конфигурации (использованы только описанные ранее поля, нет неиспользованных полей и т.п.)
- запуск бота в рамках бесплатной квоты AWS Lambda
- проверка подписей в таблице
- расшифрование данных
- отправка изменений через SSB
- хранение результатов в IPFS
- сайт с отображением данных
- различные варианты агрегации (например, на нижнем уровне максимум, затем сумма и т.п.)
- дополнительные типы вопросов: с вариантами ответов, с ограниченным диапазоном и т.д.
- возможность навешивать шифрование/хэширование и на другие поля кроме источника

# Как это запустить?

## Исходники и компиляция

1. Скачайте исходный код бота
2. Установите (cabal)[https://www.haskell.org/downloads/#minimal]
3. Соберите бота при помощи команд

``` sh
cd bot
cabal v2-update
cabal v2-build
```

## Доступ к Google Sheets

- TBA: Подключение google sheets http://hackage.haskell.org/package/gogol-0.5.0/docs/Network-Google-Auth.html#g:2
- TBA: Формат таблицы

## Генерация ключей и получение токена

Для генерации ключей запустите команду `cabal v2-run -- tgstat-keygen --gen`. В ответ будут сгенерированы случайные ключи подписи и шифрования:

```
Up to date
{"encrypt-key":"A0BCDefGh1IJ2KlMnOpqrSTUvw3Xyz/aB4cdefGHijk=","secret-key":"A01Bc234DefGhI5jk67l8/MNoPQRstuV9WxY01zabc2=","public-key":"ABcdEFghIj0kLMNO1pqRS12TuVwXy3zabC4defg5hij="}
```

Сохраните эти ключи в соответствующих полях файла конфигурации.

Секретный ключ подписи (`secret-key`) и ключ шифрования (`encrypt-key`) нужно
держать в секрете:
- любой, кто узнает секретный ключ подписи, сможет подписывать данные от вашего имени;
- любой, кто получит ключ шифрования, сможет расшифровать данные или зашифровать другие данные.

Знание открытого ключа подписи (`public-key`) позволяет проверить, что данные не были подменены (точнее, что данные записаны кем-то, у кого есть доступ к секретному ключу подписи).

Для получения токена telegram-бота начните диалог с ботом Botfather. В процессе вы сможете настроить название, ник, список команд и прочую информацию. Botfather при создании бота сообщит вам токен, примерно таким сообщением:

```
Use this token to access the HTTP API:
1234567890:AABcdeFGHhgiJk1LmNoP2QrstuVwxYzabcD
```

Внесите этот токен в конфигурацию и держите его в секрете. Любой, кто получит этот токен, сможет подменить вашего бота своим.

## Написание конфигурации и запуск бота

Конфигурация находится в json-файле. Примеры можно найти в папке [examples](/examples/). 

В конфигурации описываются поля таблицы, куда сохраняются ответы; вопросы, которые по сценарию задаются пользователю; криптографические ключи для шифрования и подписи данных. 

На данный момент название полей используются только для справочного текста. В дальнейшем их будет использовать ещё и программа инициализации таблицы.

В файле конфигурации должен находиться JSON-объект со следующими полями (**жирным** выделены названия обязательных полей, *курсивом* - опциональных).

### Описание таблицы

- **time-field** - название столбца метки времени
- **geo-field** - название столбца географии
- **source-field** - название столбца с источником данных (id пользователя в telegram)
- **source-type** - тип столбца с источником. Допустимые значения: 
  + `open`: id хранится незашифрованным, в открытом виде),
  + `encrypted`: id шифруется, организатор может расшифровать его или выложить ключ, чтобы любой мог расшифровать,
  + `hashed`: id хэшируется, никто не может восстановить его, но можно сравнить, равен ли он известному
- **fields** - описание остальных полей таблицы; список объектов, каждый объект имеет следующий формат.

Поля объектов списка `fields`:

- **name** - название поля
- **desc** - человекочитаемое описание поля (используется в справке)
- **type** - тип поля; возможные значения:
  + `int`: целое число
  + `float`: вещественное число
  + `text`: текст
  + `encrypt`: зашифрованный текст

### Описание сценария

TBA: описание работы сценария

- **welcome** - текст, который выдаётся пользователю при запуске бота командой `/start`
- *register-button* - текст на кнопке отправки геолокации (по умолчанию: `Register location`)
- *register-answer* - текст, выдаваемый после отправки геолокации (по умолчанию: `Thank you!`)
- *location-text* - текст, предваряющий записанную локацию (по умолчанию: `Location: `)
- *trust-answer* - текст, выдаваемый после команды `/trust <id>` (по умолчанию: `Started trusting user-id `)
- **questions** - список вопросов сценария; список объектов, каждый объект имеет следующий формат.

Поля объектов списка `questions`:

- **text** - текст вопроса
- **answer** - список названий полей, которые заполняются ответом на этот вопрос; должен содержать единственное поле, если тип поля `text` или `encrypt` (иначе непонятно, как делить по словам)
- *error* - текст сообщения об ошибке, если не удалось распарсить ответ (по умолчанию содержит описание ошибки и список названий нужных полей)

### Прочая информация

- **geo-file** - csv-файл с географией [следующего формата](/docs/geo.md#формат-файла)
- **bot** - информация для работы бота, объект со следующими полями:
  + **token** - токен telegram-бота
  + **public-key** - открытый ключ для подписи, base64-строка
  + **secret-key** - секретный ключ для подписи, base64-строка
  + **encrypt-key** - ключ для шифрования, base64-строка
- **targets** - куда отправляются результаты; объект с полями:
  + **sheets** - идентификатор гугл-таблицы (длинная строка между `/spreadsheets/d/` и `/edit` в адресе)
- **result** - адрес, где можно увидеть результат (строка; на данный момент используется только в справке; предлагается указать адрес гугл-таблицы с результатами)

### Запуск бота

Если доступ к Google-таблицам сохранён в файле `path/to/google-creds.json`, то бот запускается командой `GOOGLE_APPLICATION_CREDENTIALS=path/to/google-creds.json cabal v2-run tgstat`.

# Более технические детали

## Требования к формату данных

Чтобы набор данных было интереснее анализировать, имеет смысл сохранять: 

- географию поступающих сообщений - например, при первой отправке данных
  запрашивая регион, город и т.д. (детализация и геоструктура настраивается
  создателем бота)
- время поступления сообщения (возможно, загрублённо). Для проверяемости того,
  что сообщение поступило не раньше определённого времени можно использовать
  timestamping authority, либо централизованный, либо привязку к блокчейну.
  Имеет смысл указывать также последнее известное отправителю сообщение

Ещё о формате данных:
- каждого участника просят сообщить свою геолокацию, а в таблицу записывается
  ближайший муниципалитет;
- сохраняется время отправки - можно видеть динамику;
- данные можно только дополнять - поэтому все исправления видны;
- участники могут отправлять данные открыто, под псевдонимами или
  конфиденциально - на усмотрение организатора сбора;
- организатор сбора может построить сеть доверия источникам, но при этом
  отправлять данные всё ещё могут все.

## Структуры

Чтобы весь этот набор данных можно было обрабатывать, данные должны быть структурированы.

- Геоструктура образует дерево. Населённые пункты входят в регионы, регионы в
  страны и т.п. В каждом узле графа для агрегации нужно выполнять какую-то
  операцию (например, максимум или минимум - для разных источников на одном
  участке, сумму - для разных участков в одном городе). Уровни и операции
  фиксированы и задаются в конфигурационном файле.
- Доверие - орграф общего вида с выделенным истоком.
- Каждое сообщение подписывается и содержит хэш предыдущего
  сообщения, чтобы обеспечить линейную историю.
  
# Итого

Бот работает по сценарию вопросов и записывает результаты в гуглотаблицу. 

Если периодически получать данные из гуглотаблицы, можно будет проверить, что
данные только добавлялись.

## Допустимые типы собираемых данных:

- время (собирается автоматически)
- id пользователя (автоматически; можно сохранять открыто, анонимно (хэш) или конфиденциально (в зашифрованном виде))
- город (автоматически, ближайший в [файле](/example-geo.csv) к отправленной локации)
- целое число
- вещественное число
- текст
- текст в зашифрованном виде (например, для открытия результатов после завершения работа бота)
- хэш предыдущего сообщения (автоматически; сейчас в таблицу пишет один
  источник, консенсуса нод не требуется; для IPFS или SSB будет отдельно
  собираться список источников)
- подпись строки (собираем csv-строку с разделителями-запятыми и без
  экранирования из всего, что выше, подписываем; нужно будет улучшить формат,
  сейчас возможно использовать запятые в тексте)

## Криптография:

- хэш (для авторства и цепочки сообщений): SHA-256
- подпись (каждой строки): Ed25519
- шифр (для авторства и зашифрованных строк): ChaCha-Poly1305

Предполагалось, что обновления данных будут публиковаться через протокол
[SSB](https://scuttlebutt.nz/), а результаты складываться в
[IPFS](https://ipfs.io/). Тогда любой мог бы подключиться к хранению и анализу,
и чем больше людей это бы сделали, тем сложнее заблокировать сайт или
манипулировать результатами.

- TBA: Описание цепочки хэшей
- TBA: Описание подписей
- TBA: Описание криптографии
