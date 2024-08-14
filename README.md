# Тестовое задание для Junior/Стажер

Ожидаемое время выполнения: 3 - 10 дней

## Задача 
Необходимо создать сервис для создания/хранения/получения зон доставки ресторана. Сервис должен предоставлять API, работающее поверх HTTP в формате JSON.

## Требования
- Язык программирования – Typescript
- Среда выполнения – NodeJS LTS, можно использовать любые библиотеки/фреймворки
- База данных – PostgreSQL с расширением PostGIS
- Интеграционные тесты, можно использовать любой тестовый фреймворк
- Код нужно выложить на github. Предоставить инструкцию по запуску приложения и запуску тестов в файле README.md в корне репозитория

## Детали

### База данных

Используем PostgreSQL с расширением [PostGIS](https://postgis.net/). Схема таблицы для хранения зон доставки

```
create table "delivery_zone" (
  "id" serial primary key,
  "title" varchar(30) not null unique,
  "polygon" GEOGRAPHY(POLYGON,4326) not null
);

```

### Методы API

#### Метод создания зоны доставки

Создает новую зону доставки

###### Запрос
POST /delivery-zones\
body: [CreateDeliveryZoneDTO](#CreateDeliveryZoneDTO)

###### Ответы
- 200, body: [DeliveryZoneDTO](#DeliveryZoneDTO) если зона создалась
- 400, если запрос не прошел валидацию
- 422, если зона с таким title уже существует

#### Метод получения зон по координатам

Передаем координаты точки и возвращаем зоны доставки которые пересекаются с точкой. Для вычисления пересечения нужно использовать метод [ST_Intersects](https://postgis.net/docs/ST_Intersects.html) из PostGIS

###### Запрос
GET /delivery-zones\
querystring: [PointDTO](#PointDTO)

###### Ответы
- 200, body: массив из [DeliveryZoneDTO](#DeliveryZoneDTO) если все ок
- 400, если запрос не прошел валидацию

#### DTO методов

Все DTO описаны в формате [JSON Schema](https://json-schema.org/). Эти схемы можно использовать для валидации запросов и вывода типов для Typescript

###### DeliveryZoneDTO
```
{
  "additionalProperties": false,
  "type": "object",
  "properties": {
    "id": {
      "type": "integer"
    },
    "title": {
      "minLength": 1,
      "maxLength": 30,
      "type": "string"
    },
    "polygon": {
      "additionalProperties": false,
      "description": "Полигон зоны доставки в формате GeoJSON",
      "type": "object",
      "properties": {
        "type": {
          "const": "Polygon",
          "type": "string"
        },
        "coordinates": {
          "minItems": 1,
          "maxItems": 3,
          "description": "Массив колец полигона. Первое кольцо в массиве - внешнее кольцо. Остальные кольца - внутренние кольца",
          "type": "array",
          "items": {
            "minItems": 3,
            "maxItems": 150,
            "description": "Кольцо полигона, состоит из координат",
            "type": "array",
            "items": {
              "minItems": 2,
              "maxItems": 2,
              "description": "Координата. Первое значение в массиве - долгота, второе - широта",
              "type": "array",
              "items": {
                "type": "number"
              }
            }
          }
        }
      },
      "required": [
        "type",
        "coordinates"
      ]
    }
  },
  "required": [
    "id",
    "title",
    "polygon"
  ]
}
```
###### CreateDeliveryZoneDTO
```
{
  "additionalProperties": false,
  "type": "object",
  "properties": {
    "title": {
      "minLength": 1,
      "maxLength": 30,
      "type": "string"
    },
    "polygon": {
      "additionalProperties": false,
      "description": "Полигон зоны доставки в формате GeoJSON",
      "type": "object",
      "properties": {
        "type": {
          "const": "Polygon",
          "type": "string"
        },
        "coordinates": {
          "minItems": 1,
          "maxItems": 3,
          "description": "Массив колец полигона. Первое кольцо в массиве - внешнее кольцо. Остальные кольца - внутренние кольца",
          "type": "array",
          "items": {
            "minItems": 3,
            "maxItems": 150,
            "description": "Кольцо полигона, состоит из координат",
            "type": "array",
            "items": {
              "minItems": 2,
              "maxItems": 2,
              "description": "Координата. Первое значение в массиве - долгота, второе - широта",
              "type": "array",
              "items": {
                "type": "number"
              }
            }
          }
        }
      },
      "required": [
        "type",
        "coordinates"
      ]
    }
  },
  "required": [
    "title",
    "polygon"
  ]
}
```
###### PointDTO
```
{
  "additionalProperties": false,
  "type": "object",
  "properties": {
    "longitude": {
      "minimum": -180,
      "maximum": 180,
      "description": "Долгота",
      "type": "number"
    },
    "latitude": {
      "minimum": -90,
      "maximum": 90,
      "description": "Широта",
      "type": "number"
    }
  },
  "required": [
    "longitude",
    "latitude"
  ]
}
```
