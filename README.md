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

---
<a name="dto"></a>
## Объекты передачи данных
<a name="category"></a>
##### Category - модель объекта товарной категории
```
{
  "id": long,          // ID категории в системе KazanExpress
  "title": string,     // Название категории
  "parent": Category   // Родительская категория 
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
1. Ежедневный фид - https://hb.bizmrg.com/ke-archive/price-list/rshb-daily.json обновляется в **00:20 ежедневно**
2. Ежечасный фид - https://hb.bizmrg.com/ke-archive/price-list/rshb-hourly.json обновляется **каждую 10-ую минуту часа** и содержит в себе информацию об измененных товарах за прошедший час

<a name="format"></a>
##### Формат содержимого JSON-фида
```
{
  "products": [
    {
      "id": long,                           // ID товара в системе KazanExpress
      "title": string,                      // Название товара
      "category": Category,                 // Категория, в которой находится товар
      "totalAvailableAmount": int,          // Остаток товара, доступного к покупке
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
            "category": {
                "id": 1438,
                "title": "Сыворотки и Элексиры",
                "parent": {
                    "id": 1411,
                    "title": "Для волос",
                    "parent": {
                        "id": 1140,
                        "title": "Красота и Здоровье",
                        "parent": null
                    }
                }
            },
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

