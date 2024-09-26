# Документация таблиц Clickhouse

Owner: Ilnur Ibatullin
Tags: Clickhouse, Документация
Last edited time: April 19, 2024 6:47 PM

# Обновления

- 2024-09-26
    - Перенос документации на гитхаб
- 2024-04-19
    - searchesDict: preset
    - 
- 2024-02-12. Добавлено описание `selectedSeo`
- 2024-01-29. добавлено описание таблицы `feedbacksInfoHistory`
- 2024-01-16. добавлены таблицы `autoAds, hourlySeo`.
    - `productHistories`(added `stocks.*`),
    - `feedbacks`(added `colors`, `updatedAt`, removed `answerState`)
    - `productTexts`(added `contents`, `compositions`)
    - `searchInfoHistory`(added `normQuery`, `preset`)
    
    Удалена таблица `productStocks`, теперь сами остатки хранятся в таблице `productHistories` и нет нужды три раза в день парсить остатки и хранить избыточно остатки. высокочастотные остатки по-прежнему есть в таблице `stocks`
    

- 2023-10-18: в таблице `productStats` добавлены `country`, `volume`, `weight`, `numberOfItems`
- 2023-11-08
    - в таблице `searchesDict` добавлены три столбца - `normQuery, lastDate, goodsCount, subjects.id, subjects.count`
- 2023-11-17: `productHistories` и `productStats` добавлен столбец `promoText` - акция где товар участвует

# Таблицы

## `searchesDict`

Каждый день скачивается 1млн самых популярных запросов из портала ВБ, частотность берется за месяц

|имя столбца|Тип|Описание|
|--|--|--|
|`id`|`Int32`|Айди поискового запроса, в других таблицах назывется `searchKeyId`|
|`text`|`String`|текст запроса|
|`requestCount`|`Int32`|частотность запроса за 30 дней|
|`normQuery`|`String`||
|`lastDate`|`DateTime`|дата последнего обновления|
|`subjects.id`|`Array(Int32)`|Массив айди предметов|
|`subjects.count`|`Array(Int32)`|Массив количества предметов в выдаче по данному запросу|
|`goodsCount`|`Int32`|Количество товаров в выдаче|
|`preset`|`String`||
|`lemmas`|`Array(String)`|строки, полученные из yandex mystem3|

## `subjects`

Таблица с названиями предметов. Предметы - двухуровневые, не путать с деревом категорий на ВБ.

На данный момент(сентябрь 2023) обновляется в ручном режиме. Если каких-то предметов нет в списке - пишите мнe в телеграм [t.me/ilnur_ibatullin](http://t.me/ilnur_ibatullin)

|имя столбца|Тип|Описание|
|--|--|--|
|`id`|`Int32`|Айди предмета|
|`name`|`String`|Название предмета|
|`parentId`|`Int32`|айди родительского предмета|
|`parentName`|`String`|Название родительского предмета|

## `searchPositions`

Поисковая выдача

Сбор данных по поисковой выдаче по 1 млн популярных запросов из портала ВБ.

Если частотность выше 5 тыс в месяц, то парсится 10 разных ПВЗ. Из них 5 в Москве - центр, север, юг, запад и восток и 5 городов - Санкт-Петербург, Казань, Краснодар, Екатеринбург, Новосибирск. Плюс, парсится до 10 страниц в глубину

список 10 пвз, откуда делается парсинг:

| Название | Адрес |
| --- | --- |
| `MOSCOW_CENTER` | [большой Афанасьевский переулок, 22](https://go.2gis.com/daydtv)  |
| `MOSCOW_EAST` | [Оршанская, 9](https://go.2gis.com/8hbyq7) |
| `MOSCOW_SOUTH` | [Красного Маяка, 4к1](https://go.2gis.com/stbkb) |
| `MOSCOW_WEST` | [Зеленый проспект, 83](https://go.2gis.com/i28hd) |
| `MOSCOW_NORTH` | [Полярная, 27к2](https://go.2gis.com/5je78) |
| `PITER` | [Коллонтай 5/1](https://go.2gis.com/3piqxs) |
| `KRASNODAR` | [Восточно-Кругликовская, 28](https://go.2gis.com/rhqv9) |
| `KAZAN` | [Пушкина 16/2](https://go.2gis.com/c30z1) |
| `EKAT` | [Куйбышева, 121](https://go.2gis.com/tfnn8) |
| `NOVOSIB` | [Кошурникова 22/2](https://go.2gis.com/peuec) |

Если частотность ниже 5 тысяч в месяц, то парсится только по Москва.Центр, до 10 страниц в глубину

Если частотность ниже 500 то парсится только первая страница в выдаче по пвз Москва.Центр

Автокампании тоже здесь учитываются

|имя столбца|Тип|Описание|
|--|--|--|
|`productId`|`Int32`|Артикул товара|
|`date`|`Date`|Дата парсинга|
|`dest`|`MOSCOW_NORTH \| PITER \| KRASNODAR \| KAZAN \| EKAT \| NOVOSIB \| MOSCOW_CENTER \| MOSCOW_EAST \| MOSCOW_SOUTH \| MOSCOW_WEST`|ПВЗ, с которого идет парсинг|
|`searchKeyId`|`Int32`|Айди поискового запроса|
|`position`|`Int32`|Позиция в выдаче|
|`sellerId`|`Int32`|Айди селлера|
|`autoAdCpm`|`Int32`|ставка авторекламы|
|`autoAdPromotion`|`Int32`||
|`autoAdPosition`|`Int32`|Позиция без рекламы|
|`autoAdPromoPosition`|`Int32`|Позиция с рекламой|
|`autoAdAdvertId`|`Int32`|айди рекламной кампании|
|`version`|`Int64`||

В этой таблице очень много данных, в среднем каждый день появляется порядка 500млн строк, поэтому запросы нужно делать строго с фильтром по артикулам. Например `select * from searchPositions where id = 12341234`

Если нужно выбрать все артикулы конкретного селлера, то лучше всего сходить в таблицу `productStats` за артикулами и по ним сделать запрос. Или вот так:

```sql
select * 
from searchPositions 
where id in (
	select id 
	from productStats
	where sellerId = <sellerId>
)
```

Все строки этой таблицы в одной `power Bi` вряд ли кому-то пригодятся, с другой стороны это просто не влезет в память компьютера. Поэтому нужно выбирать данные строго по фильтрам

## `hourlySeo`

Имеет аналогичную струткуру, как и `searchPositions`, исключение: столбец `date(Date)` → `parsedAt(DateTime)`

Парсится каждый час, только выбранные поисковые запросы. Изначально было примерно 4000 выбранных поисковых запросов, парсинг по всем гео-позициям, до 10 страниц в глубину. Список запросов может меняться по желанию заказчика

## `autoAds`

Имеет аналогичную струткуру, как и `searchPositions`, исключение: столбец `date(Date)` → `parsedAt(DateTime)`

Парсится только первые страницы поисковых запросов с частотностью более 5000 в месяц и с `dest: MOSCOW_CENTER`

Парсится раз в час. Хранится только рекламные позиции, остальные позиции, не авторекламные, игнорируются

## `selectedSeo`

Имеет похожую на `searchPositions`. Таблица заполняется по мере запуска парсинга через бота

## `productHistories`

Таблица с историей товаров. Обновляется раз в день, если нужно следить за изменением СПП три раза в день - смотрите в `productStocks`, там больше 1 раза в день можно отслеживать. 

|имя столбца|Тип|Описание|
|--|--|--|
|`id`|`Int32`|Артикул|
|`date`|`Date`|Дата парсинга|
|`rootId`|`Int32`|Идентификатор, по которому товары связаны по цветам|
|`parsedAt`|`DateTime`|Дата парсинга|
|`prevParsedAt`|`DateTime`||
|`kindId`|`Int32`||
|`subjectId`|`Int32`|Айди предмета|
|`brandId`|`Int32`|Айди бренда|
|`sellerId`|`Int32`|Айди селлера|
|`basePrice`|`Int32`|Базовая цена до скидок|
|`price`|`Int32`|конечная цена, с учетом скидки продавцаи СПП, без учета ВБ Кошелька|
|`basicSale`|`Int32`||
|`clientSale`|`Int32`||
|`feedbacksCount`|`Int32`|количество оценок|
|`allSalesAmount`|`Int32`|(устарело)|
|`stocksCount`|`Int32`|Кол-во остатков по всем размерам и на всех складах|
|`promoText`|`String`|Название акции, в котором участвует|
|`stocks.wh`|`Array(Int32)`|Массив чисел - айди складов|
|`stocks.qty`|`Array(Int32)`|Массив чисел - количество остатков на этом складе этого размера|
|`stocks.optionId`|`Array(Int32)`|Массив чисел - айди размера|
|`stocks.name`|`Array(String)`|Массив строк - название размера|
|`stocks.origName`|`Array(String)`|Массив строк - название размера|
|`stocks.price`|`Array(Int32)`|Массив чисел - цена данного размера|
|`stocks.basePrice`|`Array(Int32)`|Массив чисел - базовая цена данного размера|
|`reviewRating`|`Float32`|оценка товара|

## `brands`

Список брендов

|имя столбца|Тип|Описание|
|--|--|--|
|`id`|`Int32`|айди бренда|
|`name`|`String`|название бренда|
|`url`|`String`|ссылка на страницу бренда|

## `sellers`

Список селлеров

|имя столбца|Тип|Описание|
|--|--|--|
|`id`|`Int32`|айди селлера|
|`feedbacksCount`|`Int32`|Количество оценок|
|`valuation`|`Float32`|оценка селлера|
|`allSalesAmount`|`Int32`|общее число продаж по всем товарам|
|`updatedAt`|`DateTime`|последнее обновление|
|`registrationDate`|`DateTime`|Дата регистрации|
|`defectPercent`|`Float32`|Процент брака|
|`hasLogo`|`Int32`||
|`hasBanner`|`Int32`||
|`rating`|`Float32`|Рейтинг|
|`deliveryDuration`|`Int32`|Среднее время доставки|
|`ratingIsInvisible`|`Int32`||
|`name`|`String`|Название селлера|
|`trademark`|`String`|Торговый знак|
|`ogrn`|`String`|ОГРН|
|`address`|`String`|Адрес|
|`inn`|`String`|ИНН|
|`fullName`|`String`|Полное наименование|
|`kpp`|`String`|КПП|
|`unp`|`String`||
|`bin`|`String`||
|`taxpayerCode`|`String`||

## `feedbacks`

отзывы на товары. Только те, что видно в клиентской стороне

Фильтровать нужно по `rootId`

|имя столбца|Тип|Описание|
|--|--|--|
|`nmId`|`Int32`|Артикул товара|
|`id`|`String`|Идентификатор отзыва|
|`wbUserId`|`Int32`|Айди Юзера|
|`wbUserName`|`String`|Название Юзера|
|`wbUserCountry`|`String`|Страна Юзера|
|`text`|`String`|Текст отзыва|
|`productValuation`|`Int32`|Оценка отзыва|
|`color`|`String`|Цвет товара|
|`size`|`String`|Размер|
|`createdAt`|`DateTime`|Отзыв создан|
|`updatedAt`|`DateTime`|Отзыв измене|
|`feedbackHelpfulness.wbUserId`|`Array(Int32)`|Массив айди - юзеры, которые оставили свои лайки/дизлайки|
|`feedbackHelpfulness.helpfulness`|`Array(Enum8('plus' = 1, 'minus' = 2, 'none' = 3))`|Массив строк - лайк/дизлайк|
|`feedbackHelpfulness.time`|`Array(DateTime)`|Массив дат, когда были оставлены реакции|
|`answerCreatedAt`|`DateTime`|Дата ответа бренда|
|`answerSupplierId`|`Int32`|Айди селлера, который оставил ответ|
|`answerText`|`String`|Текст ответа бренда|
|`photosCount`|`Int32`|Количество фотографий, которые оставил юзер|
|`answerState`|`String`||
|`rootId`|`Int32`|Идентификатор, по которому товары связаны по цветам|

## `productStats`

Актуальные данные об общих характеристиках товара. Здесь все данные о товаре собраны в одной таблице, удобно при работе с названиями и другими полями

поля `volume`, `country`, `numberOfItems`, `weight` извлекаются из характеристик, если в характеристиках они не заполнены, значит будут пустыми

Таблица обновляется один раз в день, где-то к 8-9 утра по мск. Дубликатов данных быть не должно, если есть - пишите [t.me/ilnur_ibatullin](http://t.me/ilnur_ibatullin)

|имя столбца|Тип|Описание|
|--|--|--|
|`id`|`Int32`|Артикул|
|`version`|`Int64`||
|`firstParsedAt`|`DateTime`|Дата начала отслеживания|
|`lastParsedAt`|`DateTime`|Последний парсинг|
|`rootId`|`Int32`|Идентификатор, по которому товары связаны по цветам|
|`subjectId`|`Int32`|Айди предмета|
|`brandId`|`Int32`|Айди бренда|
|`sellerId`|`Int32`|Айди селлера|
|`basePrice`|`Int32`|Цена без скидки|
|`price`|`Int32`|Цена с учетом скидок кроме ВБ Кошелька|
|`basicSale`|`Int32`||
|`clientSale`|`Int32`||
|`feedbacksCount`|`Int32`|Количество оценок|
|`allSalesAmount`|`Int32`|(deprecated) Купили более раз, устарело|
|`stocksCount`|`Int32`|Кол-во остатков по всем размерам и на всех складах|
|`brandName`|`String`|Название бренда|
|`brandUrl`|`String`|Ссылка на бренд|
|`sellerName`|`String`|Название селлера|
|`sellerInn`|`String`|ИНН|
|`sellerOgrn`|`String`|ОГРН|
|`name`|`String`|Название товара|
|`description`|`String`|Описание товара|
|`contents`|`String`||
|`composition`|`String`||
|`options_name`|`Array(String)`||
|`options_value`|`Array(String)`||
|`volume`|`String`|Объем товара, вычисляется из характеристик|
|`country`|`String`|Страна производства|
|`weight`|`String`|Вес|
|`numberOfItems`|`String`|количество предметов, вычисляется из характеристик|
|`promoText`|`String`|Название акции, в котором участвует|
|`last30DayFeedbacks`|`Int32`|Количество отзывов на данный артикул(не на rootId) за 30 дней|
|`options.name`|`Array(String)`|Массив названий характеристик|
|`options.value`|`Array(String)`|Массив значений характеристик, соотвественно|

## `productTexts`

История изменений контента товара

|имя столбца|Тип|Описание|
|--|--|--|
|`id`|`Int32`|Артикул|
|`lastModifiedAt`|`DateTime`|Последнеее обновление на ВБ|
|`date`|`DateTime`|дата парсинга|
|`name`|`String`|Название товара|
|`description`|`String`|Описание товара|
|`contents`|`String`|Состав?|
|`composition`|`String`|Состав?|
|`photosCount`|`Int32`|Количество фотографий|
|`hasVideo`|`Int32`|Имеется ли видео|
|`options.name`|`Array(String)`|Массив строк - названия характеристик|
|`options.value`|`Array(String)`|Массив строк - значения характеристик|
|`vendorCode`|`String`|Артикул поставщика|
|`volume`|`String`|значение “Объем товара” из характеристик, в отдельном поле|
|`weight`|`String`|значение “Вес товара с упаковкой (г)” или “Вес с упаковкой (кг)” из характеристик |
|`numberOfItems`|`String`|значение “Количество предметов в упаковке” из характеристик |
|`country`|`String`|значение “Страна производства” из характеристик|
|`etag`|`String`||
|`colors`|`Array(Int32)`|Массив чисел - Артикулы товаров, связанные по данной группе цветов|


## `searchInfoHistory`

история изменений частотностей запросов. пока что работает с ошибками
|имя столбца|Тип|Описание|
|--|--|--|
|`searchKeyId`|`Int32`|айди поискового запроса|
|`date`|`Date`|дата парсинга|
|`requestCount`|`Int32`|частотность запроса за 30 дней|
|`goodsCount`|`Int32`|Количество товаров в выдаче|
|`subjects.id`|`Array(Int32)`|Массив айди предметов|
|`subjects.count`|`Array(Int32)`|Массив количества предметов в выдаче по данному запросу|
|`normQuery`|`String`||
|`preset`|`String`||
|`version`|`Int64`||

## `feedbacksInfoHistory`

Сводные данные по отзывам, обновляются только если у товара изменился счетчик отзывов

|имя столбца|Тип|Описание|
|--|--|--|
|`rootId`|`Int32`|Идентификатор, по которому товары связаны по цветам|
|`feedbacksCount`|`Int32`|количество оценок|
|`feedbacksCountWithPhoto`|`Int32`|Количество отзывов с фотографиями|
|`feedbacksCountWithText`|`Int32`|Количество отзывов с текстом|
|`valuation`|`Float32`|сумма оценок|
|`valuation1Count`|`Int32`|количество оценок с 1 звездой|
|`valuation2Count`|`Int32`|количество оценок с 2 звездами|
|`valuation3Count`|`Int32`|количество оценок с 3 звездами|
|`valuation4Count`|`Int32`|количество оценок с 4 звездами|
|`valuation5Count`|`Int32`|количество оценок с 5 звездами|
|`parsedAt`|`DateTime`|дата парсинга|
|`productListFeedbacksCount`|`Int32`||
|`feedbacksPhotoUrisCount`|`Int32`|количество фотографий у последних 1000 отзывов|
