# OTA API Comparison — Мониторинг конкурентов и управление промоакциями из PMS

Дата исследования: 01.04.2026

## Цель документа

Сравнение API-возможностей OTA-платформ с точки зрения:
1. **Мониторинг конкурентов** (READ) — можно ли получать данные о ценах конкурентов через API
2. **Управление промоакциями из PMS** (WRITE) — можно ли инициировать/настраивать скидки и акции через API

---

## 1. Общая сравнительная таблица

| OTA | Мониторинг (READ) | ПUSH промо из PMS (WRITE) | Базовые цены/доступность | Документация | NDA |
|-----|-------------------|--------------------------|-------------------------|--------------|-----|
| **Booking.com** | ✅ Demand API | ✅ **ПОЛНЫЙ** (Promotions API) | ✅ | developers.booking.com | Да (Connectivity Partner) |
| **Ostrovok (ETG)** | ✅ ETG API v3 | ✅ **Частичный** (rate rules) | ✅ | docs.emergingtravel.com | Да (бесплатно) |
| **Yandex Travel** | ✅ Partner API | ❌ **Ничего** | ✅ (только через CM) | yandex.ru/dev/travel-partners-api | Нет (Partner) / Да (CM) |
| **Bronevik** | ✅ XML/SOAP | ⚠️ Ограниченный | ✅ | bronevik.com/api-solutions | Да |
| **Sutochno.ru** | ⚠️ iCal + парсинг | ❌ Нет | ⚠️ iCal | Нет | — |
| **Tvil.ru** | ❌ Нет | ❌ Нет | ❌ | Нет | — |
| **Avito** | ⚠️ Listings API | ⚠️ Только цена | ⚠️ | developers.avito.ru | Бизнес-аккаунт |
| **101hotels** | ❌ Нет | ❌ Только CM | ⚠️ Только CM | Нет | — |
| **Otello** | ❌ Нет | ❌ Только CM | ⚠️ Только CM | Нет | — |

---

## 2. Детальный разбор по платформам

### 2.1 Yandex Travel

#### Partner API (публичный, чтение)
- **URL**: `https://whitelabel.travel.yandex-net.ru/`
- **Доступ**: OAuth-токен, без NDA
- **Основные endpoints**:
  - `GET /hotels/search/` — поиск отелей с ценами
  - `GET /hotels/hotel/offers/` — предложения конкретного отеля
  - `GET /hotels/top-offers/` — лучшие предложения

#### Пример запроса мониторинга конкурентов
```
GET /hotels/search/
  ? geo_id=213
  & checkin_date=2026-05-01
  & checkout_date=2026-05-03
  & adults=2
  & bbox=37.5,55.7~37.7,55.8
  & order_by=price-asc
```

#### Что возвращает (ключевые поля)
```json
{
  "hotel_snippets": [{
    "hotel_id": "1019057204",
    "name": "Конкурентный отель",
    "top_offers": [{
      "price": { "value": 5000, "currency": "RUB" },
      "discount": {
        "strikethrough_price": 6250,
        "percent": 20,
        "reason": "Горящее предложение"
      },
      "cancellation": { "refund_type": "FULLY_REFUNDABLE" },
      "is_corporate": false
    }]
  }]
}
```

#### Channel Manager API (закрытый, запись)
- **Статус**: Под NDA, только для сертифицированных CM
- **Доступ**: Через `partner-tours@yandex-team.ru`
- **Пушит**: цены, доступность, ограничения, категории, фото
- **НЕ пушит**: промоакции, бустеры, B2B-тариф

#### Промоакции — ТОЛЬКО ЭКСТРАНЕТ (UI)
| Промоакция | API | Описание |
|-----------|-----|----------|
| Скидка 20% на первые 3 бронирования | ❌ | Только для новых объектов |
| Горящее предложение | ❌ | Скидка при бронировании за ≤7 дней |
| Раннее бронирование | ❌ | Скидка при бронировании за ≥14 дней |
| Длительное проживание | ❌ | Скидка при проживании ≥X ночей |
| Скидки лояльным путешественникам | ❌ | Для пользователей программы лояльности |
| Мобильные цены | ❌ | Только через мобильное приложение |
| Бесплатный завтрак | ❌ | Бонус к бронированию |
| Сезонные промоакции | ❌ | Календарные акции |
| B2B-тариф | ❌ | Корпоративные скидки (мин. 5%) |
| Бустеры | ❌ | Платное продвижение (+1-10% комиссия) |

---

### 2.2 Ostrovok (Emerging Travel Group — ETG)

#### ETG API v3 (публичный, чтение)
- **URL**: `https://api.worldota.net/api/b2b/v3/` (продакшен)
- **Sandbox**: `https://api-sandbox.worldota.net/api/b2b/v3/` (без NDA)
- **Документация**: docs.emergingtravel.com
- **Доступ**: API key + secret, sandbox — сразу, продакшен — NDA (бесплатно)

#### Два типа API
| Тип | Назначение | Цены |
|-----|-----------|------|
| **B2B API** | Оптовые цены (для турагентств) | Ниже розничных |
| **Affiliate API** | Розничные цены (для аффилейтов) | Как на сайте |

#### Основные endpoints для мониторинга
- `POST /search/serp/geo/` — поиск по координатам и радиусу
- `POST /search/serp/region/` — поиск по региону
- `POST /search/serp/hotels/` — поиск по ID отелей
- `GET /search/hotelpage/` — страница отеля с тарифами

#### Пример запроса мониторинга
```json
POST https://api-sandbox.worldota.net/api/b2b/v3/search/serp/geo/
{
  "checkin": "2026-05-01",
  "checkout": "2026-05-03",
  "residency": "ru",
  "language": "ru",
  "guests": [{"adults": 2, "children": []}],
  "latitude": 55.7558,
  "longitude": 37.6173,
  "radius": 5000,
  "currency": "RUB"
}
```

#### Что возвращает (ключевые поля)
```json
{
  "hotels": [{
    "hid": 12345,
    "name": "Апарт-отель Рядом",
    "rates": [{
      "show_amount": "4500",
      "show_currency_code": "RUB",
      "daily_prices": ["4500", "4500"],
      "meal_data": { "value": "nomeal" },
      "allotment": 2,
      "bar_rate_price_data": {
        "amount": "5200",
        "currency_code": "RUB"
      },
      "cancellation_penalties": { ... }
    }]
  }]
}
```

#### Push из PMS (Channel Manager)
- Доступен через отдельный Channel Manager протокол
- Поддерживает: цены, доступность, ограничения, категории
- Промоакции: через rate rules (частичная автоматизация)

#### Промоакции через API
| Функция | Доступность | Как |
|---------|-------------|-----|
| Базовые цены | ✅ | Через inventory API |
| Скидки (% от базы) | ✅ | Через rate rules |
| Last-minute скидки | ✅ | Через rate rules с окном |
| Early bird скидки | ✅ | Через rate rules с окном |
| Min/max stay | ✅ | Через restrictions |
| Close to arrival | ✅ | Через restrictions |
| Мобильная скидка | ⚠️ | Ограничено |
| Кампании (Black Friday) | ❌ | Только UI |

---

### 2.3 Booking.com

#### Demand API (чтение, мониторинг)
- **URL**: `https://demand-api.booking.com/`
- **Доступ**: Требуется регистрация партнёра
- **Для мониторинга**: поиск отелей, цены, скидки

#### Connectivity API (запись, push из PMS)
- **URL**: `https://supply-xml.booking.com/` (не-PCI)
- **URL**: `https://secure-supply-xml.booking.com/` (PCI, бронирования)
- **Форматы**: OTA XML, B.XML, JSON
- **Статус**: Требуется стать Connectivity Partner

#### Промоакции — ПОЛНЫЙ КОНТРОЛЬ ЧЕРЕЗ API

**6 типов промоакций, создаваемых через API:**

| Тип | Код | Описание | API параметры |
|-----|-----|----------|---------------|
| Базовая скидка | `basic` | Полная кастомизация | book_date, stay_date, discount, rooms, rates |
| Last-minute | `last_minute` | Скидка за X дней/часов до заезда | last_minute.unit (day/hour), last_minute.value, discount |
| Early bird | `early_booker` | Скидка при бронировании за X дней | early_booker.value, discount |
| Кампания | `campaign_*` | Сезонные акции (Black Friday и др.) | campaign_black_friday, campaign_early_year, discount (min 15-30%) |
| Гео-тариф | `geo_rate` | Скидка для гостей из определённой страны | target_channel (india_pos и др.), discount |
| Мобильная скидка | `mobile_rate` | Только через приложение/мобильный | target_channel (app/all), discount |

#### Пример создания last-minute скидки
```xml
POST https://supply-xml.booking.com/hotels/xml/promotions
<request>
  <hotel_id>12312</hotel_id>
  <promotion name="Горящее предложение" type="last_minute" target_channel="public">
    <last_minute unit="day" value="3" />
    <stay_date start="2026-05-01" end="2026-12-31" />
    <rooms><room id="1423432"/></rooms>
    <parent_rates><parent_rate id="756878"/></parent_rates>
    <discount value="20" />
  </promotion>
</request>
```

#### Пример создания early bird скидки
```xml
<request>
  <hotel_id>12312</hotel_id>
  <promotion name="Ранняя бронь" type="early_booker">
    <early_booker value="30" />
    <stay_date start="2026-06-01" end="2026-09-30" />
    <rooms><room id="1423432"/></rooms>
    <parent_rates><parent_rate id="756878"/></parent_rates>
    <discount value="15" />
  </promotion>
</request>
```

#### Параметры, управляемые через API
| Параметр | API | Описание |
|----------|-----|----------|
| discount value | ✅ | Процент скидки (1-99%) |
| book_date (start/end) | ✅ | Период бронирования |
| stay_date (start/end) | ✅ | Период проживания |
| last_minute (unit, value) | ✅ | Окно last-minute (дни/часы) |
| early_booker (value) | ✅ | Окно early bird (дни) |
| min_stay_through | ✅ | Мин. срок проживания (0-7) |
| non_refundable | ✅ | Без возврата / с возвратом |
| active_weekdays | ✅ | Только определённые дни недели |
| excluded_dates | ✅ | Исключить конкретные даты |
| additional_dates | ✅ | Добавить конкретные даты |
| rooms | ✅ | К каким типам номеров |
| parent_rates | ✅ | К каким тарифам |
| target_channel | ✅ | Аудитория (public/subscribers/app) |
| Activate/Deactivate | ✅ | Включить/выключить промо |
| Update | ✅ | Изменить существующее промо |

---

### 2.4 Bronevik

#### API (чтение)
- **Формат**: XML/SOAP
- **Доступ**: NDA
- **Документация**: bronevik.com/en/api-solutions
- **Поддерживает**: 21+ channel managers

#### Push из PMS
- Channel Manager API (закрытый)
- Базовые функции: цены, доступность, ограничения
- Промоакции: ограниченная поддержка

---

### 2.5 Sutochno.ru

#### API
- **Публичный API**: Нет (только Travelpayouts для аффилейтов)
- **Интеграция CM**: iCal sync + прямые интеграции с некоторыми PMS
- **Промоакции**: Нет API

---

### 2.6 Avito

#### Developer API
- **URL**: developers.avito.ru
- **Доступ**: Бизнес-аккаунт
- **Для недвижимости**: Listings management, realty reports
- **Цена**: Можно управлять
- **Промоакции**: Нет

---

## 3. Сводная таблица: что можно делать из PMS

### 3.1 Мониторинг конкурентов (READ)

| Действие | Yandex | Ostrovok | Booking | Bronevik |
|----------|--------|----------|---------|----------|
| Поиск отелей по координатам | ✅ bbox | ✅ geo | ✅ lat/lon | ✅ |
| Поиск отелей по региону | ✅ geo_id | ✅ region | ✅ | ✅ |
| Получение цен | ✅ top_offers | ✅ rates | ✅ | ✅ |
| Получение скидок | ✅ discount.percent | ✅ | ✅ | ✅ |
| Причина скидки | ✅ discount.reason | ⚠️ | ✅ | ⚠️ |
| Остаток номеров | ❌ | ✅ allotment | ⚠️ | ⚠️ |
| Тип питания | ✅ meal_type | ✅ meal_data | ✅ | ✅ |
| Правила отмены | ✅ cancellation | ✅ | ✅ | ✅ |
| B2B цены | ❌ | ✅ B2B API | ❌ | ⚠️ |
| BAR (Best Available Rate) | ❌ | ✅ bar_rate_price_data | ❌ | ❌ |

### 3.2 Управление промоакциями из PMS (WRITE)

| Действие | Yandex | Ostrovok | Booking |
|----------|--------|----------|---------|
| Установить базовую цену | ✅ (через CM) | ✅ | ✅ |
| Изменить доступность | ✅ (через CM) | ✅ | ✅ |
| Установить min/max stay | ✅ (через CM) | ✅ | ✅ |
| Close to arrival | ✅ (через CM) | ✅ | ✅ |
| Close to departure | ✅ (через CM) | ✅ | ✅ |
| Базовая скидка (%) | ❌ | ✅ | ✅ |
| Last-minute скидка | ❌ | ✅ | ✅ |
| Early bird скидка | ❌ | ✅ | ✅ |
| Скидка на первые бронирования | ❌ | ❌ | ✅ |
| Мобильная скидка | ❌ | ⚠️ | ✅ |
| Гео-тариф | ❌ | ❌ | ✅ |
| Сезонная кампания | ❌ | ❌ | ✅ |
| B2B-тариф | ❌ | ✅ (B2B API) | ❌ |
| Включить/выключить промо | ❌ | ✅ | ✅ |
| Бустер (повышение в выдаче) | ❌ | ❌ | ❌ |

---

## 4. Стратегия для PMS: рекомендации

### 4.1 Мониторинг конкурентов

```
Приоритет API для мониторинга:

1. Ostrovok ETG API (sandbox — без NDA, сразу)
   ├── Поиск по координатам + радиус
   ├── B2B цены (ниже розничных)
   ├── BAR (минимальная цена на рынке)
   └── Остаток номеров (allotment)

2. Yandex Partner API (без NDA, сразу)
   ├── Поиск по geo_id или bbox
   ├── Цены и скидки конкурентов
   └── Причина скидки (reason)

3. Booking.com Demand API (требует аккаунта)
   ├── Поиск по координатам
   ├── Цены и промо-отметки
   └── Охват международного рынка
```

### 4.2 Push промоакций в OTA из PMS

```
Возможности автоматизации:

Booking.com — ПОЛНАЯ АВТОМАТИЗАЦИЯ
├── Все 6 типов промо через API
├── Создание, обновление, активация, деактивация
├── Полный контроль условий (даты, дни недели, комнаты, тарифы)
└── Автоматизация: Пользователь нажал кнопку → промо создано

Ostrovok — ЧАСТИЧНАЯ АВТОМАТИЗАЦИЯ
├── Rate rules для скидок
├── Окна (last-minute, early bird)
├── Restrictions (min stay, CTA, CTD)
└── Промоакции (кампании) — только через UI

Yandex Travel — РУЧНОЕ УПРАВЛЕНИЕ
├── Push только: цены, доступность, ограничения
├── НЕТ доступа: промоакции, бустеры, B2B
├── Требуется: открыть Экстранет вручную
└── Обходные пути: нет (API не предоставлен)
```

### 4.3 UX-рекомендации для PMS

```
Когда пользователь хочет включить промоакцию:

Если OTA = Booking или Ostrovok:
  → PMS автоматически создаёт промо через API
  → "Промоакция создана ✓"

Если OTA = Yandex:
  → PMS показывает: "Для Яндекса нужно настроить в Экстранете"
  → PMS даёт прямую ссылку: https://travel.yandex.ru/extranet
  → PMS показывает подсказку: "Включите промоакцию в разделе Продвижение"
  → (Опционально) Открывает браузер через deep link
```

---

## 5. Контакты и ссылки

| OTA | API Документация | Контакт для доступа |
|-----|-----------------|---------------------|
| Yandex Travel | yandex.ru/dev/travel-partners-api | partner-tours@yandex-team.ru |
| Yandex CM | Под NDA | hotel.partners@support.yandex.ru |
| Ostrovok ETG | docs.emergingtravel.com | api-support@ostrovok.ru |
| Booking.com | developers.booking.com/connectivity | portal.connectivity.booking.com |
| Bronevik | bronevik.com/en/api-solutions | api@bronevik.com |
| Avito | developers.avito.ru | Через бизнес-кабинет |

---

## 6. Ключевые выводы

1. **Для мониторинга конкурентов** — Yandex Partner API и Ostrovok ETG API доступны сразу (sandbox), без NDA.

2. **Для push промоакций** — только Booking.com имеет полноценный Promotions API. Ostrovok поддерживает частичную автоматизацию через rate rules.

3. **Yandex Travel** — полная "чёрная дыра" для промоавтоматизации. Все 9+ промоакций доступны ТОЛЬКО через ручное управление в Экстранете. Channel Manager API не включает промофункции.

4. **Стратегический вывод**: Если ваша PMS хочет конкурировать с Bnovo/TravelLine, ключевое отличие — это **автоматизация промоакций через API**. Booking.com это позволяет, Яндекс — нет.

5. **Гибридный подход**: PMS мониторит конкурентов через Yandex + Ostrovok API, автоматически включает промо на Booking.com и Ostrovok через API, а для Яндекса — отправляет пользователя в Экстранет с подсказкой.
