1. Я хочу с помощью API добавить новые товары на продажу в Магазине на Kaspi.kz. Как получить схему JSON?

Пример запроса
GET
https://kaspi.kz/shop/api/products/import/schema?HTTP/1.1=&Host= kaspi.kz
Accept: application/json
X-Auth-Token: token

пример ответа

 {
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Product attributes import scheme",
    "description": "Used to validate JSON documents with product attribute values and images.",
    "type": "array",
    "minItems": 1,
    "maxItems": 10000,
    "items": {
        "title": "Array of products",
        "description": "Contains individual products with their attributes and images.",
        "type": "object",
        "properties": {
            "sku": {
                "title": "SKU",
                "description": "Unique identifier used to find products in catalog.",
                "type": "string",
                "maxLength": 64,
                "minLength": 1,
                "example": "00167PVS"
            },
            "title": {
                "title": "Product title",
                "description": "Contains product title that would be shown on storefront, without category prefix (eg. Smartphone, Fridge etc.).",
                "type": "string",
                "maxLength": 1024,
                "minLength": 1,
                "example": "Sony Alpha A6100 Kit 16-50 мм OSS черный"
            },
            "brand": {
                "title": "Product brand",
                "description": "Contains product brand name that would be used in 'Brand' facet on product listing pages of storefront.",
                "type": "string",
                "maxLength": 256,
                "minLength": 1,
                "example": "Sony"
            },
            "category": {
                "title": "Category code",
                "description": "This code would be used to assign the product to a category. It also defines the scope of available attributes. Only leaf categories are accepted",
                "type": "string",
                "maxLength": 512,
                "minLength": 1,
                "example": "Master - Photo cameras",
                "pattern": "^Master ?-[a-zA-Z_\\s\\d-]+$"
            },
            "description": {
                "title": "Short product definition",
                "description": "Contains small product definition used on item page. Usually defaults to some major specs.",
                "type": "string",
                "maxLength": 1024,
                "minLength": 1,
                "example": "- тип: беззеркальная со сменной оптикой\n- число эффективных пикселов: 24 Мпикс\n- объектив в комплекте: да\n- диагональ жк-экрана: 3 дюйм\n- ручная настройка выдержки и диафрагмы: да"
            },
            "weight": {
                "title": "Product's weight (optional)",
                "description": "Contains double value, but type is string",
                "type": "string",
                "maxLength": 256,
                "minLength": 1
            },
            "attributes": {
                "title": "List of attributes",
                "description": "Array of attributes available for the assigned category.",
                "type": "array",
                "minItems": 1,
                "maxItems": 256,
                "items": {
                    "title": "Individual attribute",
                    "description": "Contains data for individual attribute.",
                    "type": "object",
                    "properties": {
                        "code": {
                            "title": "Attribute code",
                            "description": "Identifies attribute within category. Unique inside category.",
                            "type": "string",
                            "maxLength": 256,
                            "minLength": 1,
                            "example": "Photo cameras*Camera matrix.photo cameras*type",
                            "pattern": "^([a-zA-Z_\\s\\d&.\\-#%\/,]+\\*)+[a-zA-Z_\\s\\d&.\\-#%\/,]+$"
                        },
                        "value": {
                            "title": "Attribute value",
                            "description": "Array could be used for multivalued attributes only.",
                            "example": [
                                "[\"зеркальный фотоаппарат (TTL)\", \"фотоаппарат с оптическим видоискателем\"]",
                                "\"6000x4000\"",
                                "24"
                            ],
                            "oneOf": [
                                {
                                    "type": "array",
                                    "minItems": 1,
                                    "maxItems": 32,
                                    "uniqueItems": true,
                                    "items": {
                                        "type": "string",
                                        "maxLength": 256,
                                        "minLength": 1
                                    }
                                },
                                {
                                    "type": "string",
                                    "maxLength": 256,
                                    "minLength": 1
                                },
                                {
                                    "type": "number"
                                },
                                {
                                    "type": "boolean"
                                }
                            ]
                        }
                    },
                    "required": [
                        "code",
                        "value"
                    ],
                    "additionalProperties": false
                }
            },
            "images": {
                "title": "Product images",
                "description": "Must contain at least one valid image url. Valid means downloadable and having valid picture as content. Redirects are supported.",
                "type": "array",
                "minItems": 1,
                "uniqueItems": true,
                "items": {
                    "type": "object",
                    "properties": {
                        "url": {
                            "title": "Image url",
                            "description": "Individual image url.",
                            "type": "string",
                            "maxLength": 1024,
                            "minLength": 1,
                            "format": "uri",
                            "pattern": "^https?://",
                            "example": "https://cdn-kaspi.kz/shop/medias/sys_master/images/images/h5d/h37/16008607399966/sony-alpha-a6600-kit-16-50mm-cernyj-100692376-1.jpg"
                        }
                    },
                    "required": [
                        "url"
                    ],
                    "additionalProperties": false
                }
            }
        },
        "required": [
            "sku",
            "title",
            "brand",
            "category",
            "attributes",
            "images"
        ],
        "additionalProperties": false
    }
}


2. Как с помощью API в Магазине на Kaspi.kz узнать код категории для добавления товара

пример запроса

GET /shop/api/products/classification/categories HTTP/1.1
Host: kaspi.kz
Accept: application/json
X-Auth-Token: token

пример ответа

 [
    {
        "code": "Master - Exercise notebooks",
        "title": "Тетради"
    },
    {
        "code": "Master - Head units",
        "title": "Автомагнитолы"
    },
    {
        "code": "Master - Car alarms",
        "title": "Автосигнализации"
    },
    {
        "code": "Master - Blenders",
        "title": "Блендеры"
    },
    {
        "code": "Master - Cooktops",
        "title": "Варочные поверхности"
    },
    {
        "code": "Master - Video cameras",
        "title": "Видеокамеры"
    },
    {
        "code": "Master - Videocards",
        "title": "Видеокарты"
    },
    {
        "code": "Master - DVRs",
        "title": "Видеорегистраторы"
    }
]

3. Как с помощью API в Магазине на Kaspi.kz узнать список характеристик товара?

пример запроса

GET
https://kaspi.kz/shop/api/products/classification/attributes?c=Master - Exercise notebooks&HTTP/1.1=&Host= kaspi.kz
Accept: application/json
X-Auth-Token: token

пример ответа

 [
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*type",
        "type": "enum",
        "multiValued": false,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*format",
        "type": "enum",
        "multiValued": true,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*orientation",
        "type": "enum",
        "multiValued": false,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*ruling",
        "type": "enum",
        "multiValued": false,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*fields",
        "type": "boolean",
        "multiValued": false,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*number of sheets",
        "type": "number",
        "multiValued": false,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*number of notebooks",
        "type": "number",
        "multiValued": false,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*paper whiteness",
        "type": "number",
        "multiValued": false,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*framing",
        "type": "enum",
        "multiValued": true,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*paper",
        "type": "enum",
        "multiValued": true,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*base weight",
        "type": "number",
        "multiValued": false,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*cover",
        "type": "enum",
        "multiValued": false,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*decor",
        "type": "enum",
        "multiValued": true,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*design",
        "type": "enum",
        "multiValued": true,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*color",
        "type": "enum",
        "multiValued": true,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*length",
        "type": "number",
        "multiValued": false,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*width",
        "type": "number",
        "multiValued": false,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*weight",
        "type": "number",
        "multiValued": false,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*features",
        "type": "enum",
        "multiValued": true,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*components",
        "type": "string",
        "multiValued": false,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*additional",
        "type": "string",
        "multiValued": false,
        "mandatory": false
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*notice",
        "type": "string",
        "multiValued": false,
        "mandatory": true
    },
    {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*range",
        "type": "enum",
        "multiValued": false,
        "mandatory": true
    }
]

4. Как с помощью API добавить товар для продажи в Магазине на Kaspi.kz?

пример запроса

POST /shop/api/products/import HTTP/1.1
Host: kaspi.kz
Accept: application/json
X-Auth-Token: token
Content-Type: text/plain
 
//body
 
[
 
  {
 
    "sku": "Testsku»,
    "title": "test notebook",
    "brand": "test",
    "category": "Master - Exercise notebooks",
    "description": "Some description",
    "attributes": [
            {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*type",
        "value": "тетрадь-блокнот"
      },
      {
 
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*format",
        "value": "А5"
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*orientation",
        "value": "вертикальная"
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*ruling",
        "value": "крупная клетка"
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*fields",
        "value": true
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*number of sheets",
        "value": 48
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*number of notebooks",
        "value": 2
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*framing",
        "value": "резинка"
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*paper",
        "value": "офсетная"
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*cover",
        "value": "мягкая"
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*color",
        "value": "желтый"
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*notice",
        "value": "тестест"
      },
      {
        "code": "Exercise notebooks*Obsie harakteristiki.exercise notebooks*range",
        "value": "расцветка в зависимости от наличия на складе"
      }
         ],
    "images": [
      {
        "url": "https://resources.cdn-kaspi.kz/img/m/p/h44/h88/63854412398622.jpg"
      }
    ]
  }
]

пример ответа

{
    "code": "testproduct",
    "status": "UPLOADED"
}

5. Я с помощью API добавил товары для продажи в Магазине на Kaspi.kz. Как узнать результат?

пример запроса

GET
https://kaspi.kz/shop/api/products/import/result?i=testproduct&HTTP/1.1=&Host=kaspi.kz
Accept: application/json
X-Auth-Token: token

пример ответа

 {
    "errors": 0,
    "warnings": 0,
    "skipped": 0,
    "total": 1,
    "result": {
        " Testsku": {
            "state": "FINISHED"
        }
    }
}

6. Как с помощью API узнать, что товар добавлен для продажи в Магазине на Kaspi.kz?

пример запроса

GET
https://kaspi.kz/shop/api/products/import?i=testproduct&HTTP/1.1=&Host=kaspi.kz
Accept: application/json
X-Auth-Token: token

пример ответа

 {
    "code": "testproduct ",
    "status": "FINISHED"
}

7. Как с помощью API в Магазине на Kaspi.kz получить возможные значения для характеристик товара?

пример запроса

GET
https://kaspi.kz/shop/api/products/classification/attribute/values?c=Master - Exercise notebooks&HTTP/1.1=&Host= kaspi.kz&a=Exercise notebooks*Obsie harakteristiki.exercise notebooks*cover
Accept: application/json
X-Auth-Token: token

пример ответа

 [
    {
        "code": "мягкая",
        "name": "мягкая"
    },
    {
        "code": "твердая",
        "name": "твердая"
    }
]


