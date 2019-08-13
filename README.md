# Документация к API KazanExpress для интеграции сторонних сервисов
Данный документ содержит в себе информацию о функционале и методах REST-API маркетплейса KazanExpress, которая необходима для успешной интеграции товарной выгрузки и оформления заказов.

---
## Содержание
1. [ Объекты передачи данных ](#dto)
    * [ Category ](#category)
    * [ Characteristic ](#characteristic)
    * [ CharacteristicMapping ](#characteristic_mapping)
    * [ Photo ](#photo)
2. [ Товарные фиды ](#feeds)
    * [ Формат содержимого ](#format)
    * [ Пример фида ](#example)
    * [ Пример фида для категорий ](#example-categories)
3. [ Работа с заказами ](#order)
    * [ Создание заказа ](#create)
    * [ Обновление статуса платежа ](#payment)
    * [ Получение заказа ](#order)
    * [ Статусы заказа ](#statuses)
4. [ Коды ответов ](#responses)

---
<a name="dto"></a>
## Объекты передачи данных
<a name="category"></a>
##### Category - модель объекта товарной категории
```
{
  "id": long,               // ID категории в системе KazanExpress
  "title": string,          // Название категории
  "children": Category[]    // Родительская категория 
}
```

<a name="characteristic"></a>
##### Characteristic - модель объекта выбираемой характеристики для товара
```
{
  "title": string,     // Название характеристики
  "values": [          // Список значений характеристики
    {          
      "title": string, // Название значения характеристики
      "value": string  // Значение характеристики (для характеристики "Цвет" сюда пишутся Hex-коды цветов для отображения)
    }
  ]
}
```

<a name="characteristic_mapping"></a>
##### CharacteristicMapping - модель соотношения выбранных характеристики к SKU
```
{
  "charIndex": int,    // Индекс названия характеристики
  "valueIndex": int    // Индекс значения характеристики 
}
```

<a name="photo"></a>
##### Photo - модель ссылок на фотографии товара в разных размерах и качествах
```
{
  "80": {
    "high": string,     // Ссылка на изображение товара в высоком качестве размера 80:80  
    "low": string       // Ссылка на изображение товара в низком качестве размера 80:80  
  }, 
  "240": {
    "high": string,
    "low": string
  },
  "540": {
    "high": string,
    "low": string
  },
  "800": {
    "high": string,
    "low": string
  },
  "color": string        // Если у товара поле colorPhotoPreview == true, то фото будет соотноситься
                         // с характеристикой с названием, соответствующим значению поля "color"
}
```

<a name="sku"></a>
##### SKU - модель отдельной вариации товара (SKU)
```
{
  "id": long,                                    // ID SKU
  "characteristics": CharacteristicMapping[],    // Соотношение характеристик к SKU (см. пример)
  "availableAmount": int,                        // Кол-во, доступное к покупке для данного SKU или выбранных характеристик
  "price": float                                 // Cтоимость выбранной вариации SKU в рублях
}
```

---
<a name="feeds"></a>
### Товарные фиды для актуализации цен и остатков товаров на площадке
На стороне приложения **KazanExpress** будут генерироваться 2 JSON-файла по следующим адресам:
1. Ежедневный фид - https://hb.bizmrg.com/ke-archive/price-list/rshb-daily.json обновляется в **04:11 ежедневно**
2. Ежечасный фид - https://hb.bizmrg.com/ke-archive/price-list/rshb-hourly.json обновляется **каждую 26-ую минуту часа** и содержит в себе информацию об измененных товарах за прошедший час
3. Ежедневный фид для категорий - https://hb.bizmrg.com/ke-archive/price-list/rshb-categories.json обновляется в **04:11 ежедневно**
<a name="format"></a>
##### Формат содержимого JSON-фида
```
{
  "products": [
    {
      "id": long,                           // ID товара в системе KazanExpress
      "title": string,                      // Название товара
      "categoryId": long,                   // Идентификатор категории, в которой находится товар
      "description": string,                // Описание товара в формате HTML
      "attributes": string[],               // Краткие характеристики товара
      "photos": Photo[],                    // Фото товаров
      "characteristics": Characteristic[],  // Список назавний и значений характеристик
      "skuList": Sku[],                     // Список вариаций товара (SKU)
      "usePreview": bool                    // Поле, указывающее, нужно ли соотносить фото с характеристикой "Цвет"
    }
  ]
}
```

<a name="example"></a>
##### Пример фида
```
{
    "products": [
        {
            "id": 38254,
            "title": "Увлажняющая сыворотка «BIOAQUA» для волос с эфирными маслами",
            "category": 1438,
            "totalAvailableAmount": 5,
            "description": "<p>Применение&nbsp;масла обеспечивает интенсивное лечение сухим и ломким волосам,&nbsp;возвращая им жизненные силы и красоту.&nbsp;Эфирные масла стимулируют рост новых клеток, способствует расслаблению, благотворно влияет на состояние кожи и волос.&nbsp;</p>,
            "attributes": [
                "Мгновенно впитывается, не придавая им жирного блеска волосам",
                "Активно питает, интенсивно увлажняет, быстро восстанавливает волосы и кожу головы",
                "Предотвращает спутанность и «пушение» волос",
                "Интенсивное лечение сухих и ломких волос.",
                "Объем: 50 мл"
            ],
            "photos": [
                {
                    "photo": {
                        "80": {
                            "high": "https://kazanexpress.hb.bizmrg.com/bi5ok10s5smuh80iudfg/t_product_80_high.jpg",
                            "low": "https://kazanexpress.hb.bizmrg.com/bi5ok10s5smuh80iudfg/t_product_80_low.jpg"
                        },
                        "240": {
                            "high": "https://kazanexpress.hb.bizmrg.com/bi5ok10s5smuh80iudfg/t_product_240_high.jpg",
                            "low": "https://kazanexpress.hb.bizmrg.com/bi5ok10s5smuh80iudfg/t_product_240_low.jpg"
                        },
                        "540": {
                            "high": "https://kazanexpress.hb.bizmrg.com/bi5ok10s5smuh80iudfg/t_product_540_high.jpg",
                            "low": "https://kazanexpress.hb.bizmrg.com/bi5ok10s5smuh80iudfg/t_product_540_low.jpg"
                        },
                        "800": {
                            "high": "https://kazanexpress.hb.bizmrg.com/bi5ok10s5smuh80iudfg/original.jpg",
                            "low": "https://kazanexpress.hb.bizmrg.com/bi5ok10s5smuh80iudfg/t_product_low.jpg"
                        }
                    },
                    "color": null
                }
            ],
            "characteristics": [
                {
                    "title": "Масло:",
                    "values": [
                        {
                            "title": "Лаванда",
                            "value": "Лаванда"
                        },
                        {
                            "title": "Роза",
                            "value": "Роза"
                        }
                    ]
                }
            ],
            "skuList": [
                {
                    "id": 102291,
                    "characteristics": [       // Сыворотка с маслом Лаванда
                        {
                            "charIndex": 0,    // characteristics[0] = Масло
                            "valueIndex": 0    // characteristics[0].values[0] = Лаванда
                        }
                    ],
                    "amount": 2,
                    "price": 155
                },
                {
                    "id": 98790,
                    "characteristics": [       // Сыворотка с маслом Розы
                        {
                            "charIndex": 0,    // characteristics[0] = Масло
                            "valueIndex": 1    // characteristics[0].values[1] = Роза 
                        }
                    ],
                    "amount": 3,
                    "price": 155
                }
            ],
            "colorPhotoPreview": false
        }
    ]
}
```
---
<a name="order"></a>

<a name="example-categories"></a>
##### Пример фида для категорий
```
{
    "categories": {
        "id": 1140,
        "title": "Красота и Здоровье",
        "children": [
            {
                "id": 1411,
                "title": "Для волос",
                "children": [
                    {
                        "id": 1438,
                        "title": "Сыворотки и Элексиры",
                        "children": []
                    }, {
                        "id": 1487,
                        "title": "Масла",
                        "children": []
                    }
                ]
            }
        ]
    }
}
```
### Работа с заказами на площадке

1. Заказ должен быть размещен на KazanExpress и на площадке партнера одновременно, не должно возникать ситуаций, когда заказ есть в одной из систем и нет в другой.
2. Запросы к API требуют авторизации авторизация. Вам должен быть выдан токен, который надо передавать вместе с каждым запросом.

Пример передачи авторизации для токена: `912e8227-677f-431e-b74c-988cba313772`

```
GET https://kazanexpress.ru/api/partner/order/
Accept: application/json
Authorization: Bearer 912e8227-677f-431e-b74c-988cba313772
Cache-Control: no-cache
Content-Type: application/json;charset=UTF-8;

```

<a name="create"></a>
#### Размещение заказа на площадке
Пример запроса: 
```
POST https://kazanexpress.ru/api/partner/order
Cache-Control: no-cache
Content-Type: application/json
Authorization: Bearer %TOKEN%

{
    "price": 120.0,
    "comment": "",
    "contacts": {
        "name": "Владимир",
        "lastName": "Иванов",
        "email": "v.ivanov@mail.ru",
        "phone": "9998887766"
    },
    "delivery": {
        "type": "postmail",
        "cityName": "Казань",
        "address": "ул. Туктамышева, д. 14, кв. 132",
        "zipCode": 462400
    },
    "orderItems": [
        {
            "skuId": 143,
            "purchasePrice": 30.0,
            "amount": 2
        },
        {
            "skuId": 1664,
            "purchasePrice": 60.0,
            "amount": 1
        }
    ]
}
```

##### Поля

* **price*** - общая цена заказа в рублях
* **comment** - комментарий к заказу
* **contacts*** - контактные данные покупателя
    * **name*** - имя
    * **lastName*** - фамилия
    * **phone** - телефон
    * **email** - адрес электронной почты
* **delivery*** - информация о доставке
    * **type*** - обязательное поле, содержит один из двух видов типа доставки:
        * **post** - доставка Почтой России
        * **deliveryPoint** - доставка до пункта выдачи заказов KazanExpress
    * **cityName*** - название города получателя
    * **address** - обязательное поле для типа **post**, адрес получателя
    * **zipCode** - обязательное поле для типа **post**, почтовый индекс получателя
    * **cityId** - обязательное поле для типа **deliveryPoint**, идентификатор города получателя в системе KazanExpress
    * **deliveryPointId** - обязательное поле для типа **deliveryPoint**, идентификатор пункта выдачи в системе KazanExpress
* **orderItems*** - не пустой массив позиций в заказе
    * **skuId*** - идентификатор СКУ в системе KazanExpress
    * **purchasePrice*** - цена покупки в рублях
    * **amount*** - количество товара
    
Все поля, помеченные звездочкой, являются обязательными.

##### Пример ответа

Успешное создание заказа:
```
POST https://kazanexpress.ru/api/partner/order

HTTP/1.1 200 OK
Server: nginx/1.15.6
Content-encoding: gzip
Content-Type: application/json;charset=UTF-8

{
    "status": {
        "code": 0,
        "message": "Success",
        "localizedMessage": "Успех"
    },
    "payload": {
        "orderId": 129709,
        "status": "CREATED",
        "price": 120.0,
        "orderItems": [
            {
                "orderItemId": 240,
                "skuId": 143,
                "purchasePrice": 30.0,
                "amount": 2
            },
            {
                "orderItemId": 241,
                "skuId": 1664,
                "purchasePrice": 60.0,
                "amount": 1
            }
        ]
    }
}
```

Неуспешное создание заказа:
```
POST https://kazanexpress.ru/api/partner/order

HTTP/1.1 200 OK
Server: nginx/1.15.6
Content-encoding: gzip
Content-Type: application/json;charset=UTF-8

{
    "status": {
        "code": 11,
        "message": "Some items from the order ran out of stock",
        "localizedMessage": "Некоторые позиции из заказа закончились"
    },
    "payload": {
        "orderItems": [
            {
                "skuId": 143,
                "availableAmount": 1
            }
        ]
    }
}
```

##### Поля в ответе

* **status** - содержит статус ответа или ошибки
    * **code** - статус/код ошибки, служит для быстрого определения типа ответа. Все **ненулевые статусы** считаются ошибкой
    * **message** - краткое описание статуса, на английском
* **payload** - содержит основную информацию в ответе
    * **orderId** - идентификатор успешно созданного заказа
    * **status** - статус заказа
    * **price** - общая сумма к оплате
    * **orderItems** - позиции заказа
        * **orderItemId** - идентификатор позиции заказа
        * **skuId** - идентификатор СКУ в системе
        * **purchasePrice** - сумма к оплате за одну штуку, в рублях
        * **amount** - количество
        * **availableAmount**: количество СКУ оставшееся на складе
        
Возможно следующие ошибки:

* Код *** - количество указанное в заказе больше, чем кол-во доступное для заказа, 
в таком слуае в ответе придет массив orderItems, содержащий максимально возможное кол-во товара для заказа.

* Код *** - цена не совпадает с ценой для покупки. В данном случае в ответе придет массив orderItems, 
содержащий все позиции в заказе с несовпадающей ценой.

<a name="payment"></a>
#### Изменение статуса оплаты для заказа
Пример запроса на подтверждение платежа: 
```
POST https://kazanexpress.ru/api/partner/payment
Cache-Control: no-cache
Content-Type: application/json
Authorization: Bearer %TOKEN%

{
    "paymentId": "bvkauqi214100092",
    "amount": 120.0,
    "orderId": 129709,
    "status": "confirmed"
}
```

Пример запроса на отмену платежа: 
```
POST https://kazanexpress.ru/api/partner/payment
Cache-Control: no-cache
Content-Type: application/json
Authorization: Bearer %TOKEN%

{
    "paymentId": "bvkauqi214100092",
    "orderId": 129709,
    "status": "canceled"
}
```

##### Поля

* **paymentId*** - идентификатор платежа в платежной системе
* **amount** - полная сумма оплаты
* **orderId*** - идентификатор заказа, который был оплачен
* **status*** - статус оплаты, возможные значение `confirmed`, `canceled`
    * `confirmed` - платеж успешно оплачен
    * `canceled` - платеж отменен со стороны платежной системы, клиента, партнера

##### Пример ответа
```
POST https://kazanexpress.ru/api/partner/payment

HTTP/1.1 200 OK
Server: nginx/1.15.6
Content-encoding: gzip
Content-Type: application/json;charset=UTF-8

{
    "status": {
        "code": 0,
        "message": "Success",
        "localizedMessage": "Успех"
    },
    "payload": null
}
```


<a name="order"></a>
#### Получение заказа
Пример запроса
```
GET https://kazanexpress.ru/api/partner/order/{{orderId}}
Cache-Control: no-cache
Content-Type: application/json
Authorization: Bearer %TOKEN%

```

* **orderId** - идентификатор заказа в системе KazanExpress

##### Пример ответа
```
GET https://kazanexpress.ru/api/partner/order/{{orderId}}

HTTP/1.1 200 OK
Server: nginx/1.15.6
Content-encoding: gzip
Content-Type: application/json;charset=UTF-8

{
    "status": {
        "code": 0,
        "message": "Success",
        "localizedMessage": "Успех"
    },
    "payload": {
        "orderId": 129709,
        "status": "CREATED",
        "price": 120.0,
        "orderItems": [
            {
                "orderItemId": 240,
                "skuId": 143,
                "purchasePrice": 30.0,
                "amount": 2
            },
            {
                "orderItemId": 241,
                "skuId": 1664,
                "purchasePrice": 60.0,
                "amount": 1
            }
        ]
    }
}
```

<a name="statuses"></a>
#### Возможные статусы заказа
* **CREATED** - создан, не оплачен
* **ACTIVE** - оплачен, принят в обработку
* **PACKED** - собран на складе, готовится к отправке
* **DELIVERING** - доставляется до клиента
* **DELIVERED** - доставлен до пункта выдачи KazanExpress
* **COMPLETED** - выдан покупателю

<a name="responses"></a>
## Коды ответов

code | причина 
--- | ---  
0 | Успешно выолненная операция
1 | Внутренняя ошибка сервера
2 | Неверно сформирован запрос
10 | Запрашиваемый СКУ не найден
11 | Несовпадение кол-ва СКУ
12 | Сумма заказа не совпадает
13 | Цена за СКУ не совпадает
20 | Заказ не найден
30 | Неправильная система оплаты для заказа
31 | Статус/состояние платежа не совпадает
32 | Ошибка обмена данными с банком
33 | Неверный статус заказа
34 | Невозможно отменить платеж

