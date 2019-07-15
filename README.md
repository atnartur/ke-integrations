# Документация к API KazanExpress для интеграции сторонних сервисов
Данный документ содержит в себе информацию о функционале и методах REST-API маркетплейса KazanExpress, которые необходимы для успешной интеграции товарной выгрузки и оформления заказов.

## Содержание
1. [ Объекты передачи данных ](#dto)
  * [ Category ](#category)
  * [ Characteristic ](#characteristic)
  * [ Photo ](#photo)
2. [ Товарные фиды ](#feeds)

<a name="dto"></a>
### Объекты передачи данных
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

<a name="feeds"></a>
### Товарные фиды для актуализации цен и остатков товаров на площадке
На стороне приложения **KazanExpress** будут генерироваться 2 JSON-файла по следующим адресам:
1. Ежедневный фид - https://hb.bizmrg.com/ke-archive/price-list/rshb-daily.xml обновляется в **00:20 ежедневно**
2. Ежечасный фид - https://hb.bizmrg.com/ke-archive/price-list/rshb-hourly.xml обновляется **каждую 10-ую минуту часа** и содержит в себе информацию об измененных товарах за прошедший час

##### Формат содержимого JSON-фида

```
{
  "products": [
    {
      "id": long,                           // ID товара в системе KazanExpress
      "title": string,                      // Название товара
      "category": Category,                 // Категория, в которой находится товар
      "totalAvailableAmount": int,          // Остаток товара, доступного к покупке
      "description": string,                // Описание товара
      "attributes": string[],               // Краткие характеристики товара
      "photos": Photo[],                    // Фото товаров
      "characteristics": Characteristic[],  // Список назавний и значений характеристик
      "skuList": 
    }
  ]
}
```


