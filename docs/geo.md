# География и её получение

Для привязки пользователя к городу нужен список координат городов. Для привязки
будет выбран ближайший город.

Далее, для группировки по регионам нужно соответствие города - муниципальному
району, того - субъекту РФ. И для каждой территориальной единицы нужны
координаты центра для отображения на карте.

## Получение всех населённых пунктов

Для получения всех населённых пунктов используем wikidata и следующий запрос SPARQL:

```sparql
SELECT ?settlement ?settlementLabel ?okato ?lat ?lon
WHERE
{ 
  ?settlement wdt:P764 ?okato .
  ?settlement p:P625 ?coord .
  ?coord psv:P625 ?node .
  ?node wikibase:geoLatitude  ?lat .
  ?node wikibase:geoLongitude ?lon .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],ru,en". }
}
```

Выполнив запрос в https://query.wikidata.org/, можем скачать результаты (92632
строки) и обрабатывать их дальше. Для группировки данных мы будем использовать
то, что код ОКАТО иерархичен, и районам и субъектам соответствуют более короткие
коды, чем городам.

## Очистка данных

Проверим, что с данными всё хорошо, используя [mlr](https://johnkerl.org/miller/doc/index.html):

```sh
mlr --csv sort -f okato then count-similar -g okato then filter '$count > 1' cities.csv | wc -l
```

Получаем 2607 строк с повторяющимися ОКАТО. Судя по выборочной проверке, это
из-за импорта из разных баз координат. Поэтому просто оставим по одной строке с
каждым ОКАТО:

```sh
mlr --csv sort -f okato then head -n 1 -g okato cities.csv > uniq-cities.csv
```

## Группировка по районам и субъектам и готовые данные

Для группировки используем то, что ОКАТО иерархичен и [этот скрипт](/attic/geo.R).

Готовые данные лежат в [example-geo.csv](/example-geo.csv).

## Формат файла

Итоговый файл - csv-таблица с разделителями-запятыми и первой строкой-заголовком.

Каждая строка описывает один населённый пункт и содержит следующие столбцы (названия полей не важны):

- `okato` - идентификатор населённого пункта (в примере - ОКАТО)
- `name.4` - название населённого пункта
- `lat.4` - его широта
- `lon.4` - его долгота
- `name.3` - название муниципального образования, в котором расположен населённый пункт
- `lat.3` - его широта
- `lon.3` - его долгота
- `name.2` - название муниципального района, к которому относится населённый пункт
- `lat.2` - его широта
- `lon.2` - его долгота
- `name.1` - название субъекта России, к которому относится населённый пункт
- `lat.1` - его широта
- `lon.1` - его долгота
