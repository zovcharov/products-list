# Product Data Format

Схема описывает одну карточку товара в каталоге. Один объект = один SKU.

## Пример

```json
{
  "id": 1,
  "sku": "SM-A54-128-BLK",
  "title": "Смартфон Galaxy A54 128GB",
  "description": "Яркий AMOLED-экран, тройная камера 50 Мп…",
  "category": "Электроника",
  "brand": "Samsung",

  "price": 32990,
  "oldPrice": 36660,
  "costPrice": 19790,
  "currency": "RUB",
  "vat": 20,
  "discount": 10,
  "bonusPoints": 330,
  "promoLabel": null,
  "promoEndsAt": null,

  "rating": 4.6,
  "reviewsCount": 1842,
  "ratingBreakdown": { "5": 1216, "4": 424, "3": 110, "2": 55, "1": 37 },
  "photosInReviews": 276,
  "recommendRate": 92,

  "inStock": true,
  "image": "https://loremflickr.com/600/600/smartphone,android?lock=1",
  "createdAt": "2024-03-12"
}
```

## Поля

### Идентификация
| Поле | Тип | Описание |
|---|---|---|
| `id` | number | Уникальный идентификатор товара |
| `sku` | string | Артикул (Stock Keeping Unit), уникален |
| `title` | string | Название товара |
| `description` | string | Краткое описание (1–2 предложения) |
| `category` | string | Категория каталога |
| `brand` | string | Бренд-производитель |

### Цены и деньги
| Поле | Тип | Описание |
|---|---|---|
| `price` | number | Текущая цена в минорных единицах (рубли) |
| `oldPrice` | number | Цена до скидки. Если скидки нет — равна `price` |
| `costPrice` | number | Закупочная цена (для расчёта маржи) |
| `currency` | string | Код валюты по ISO 4217 (`RUB`, `USD`, `EUR`) |
| `vat` | number | Ставка НДС в % (10 или 20) |
| `discount` | number | Размер скидки в % (0–99) |
| `bonusPoints` | number | Кэшбэк баллами, которые получит покупатель |
| `promoLabel` | string \| null | Бейдж на карточке: `"Хит продаж"`, `"Скидка"`, `"Новинка"` или `null` |
| `promoEndsAt` | string (ISO date) \| null | Дата окончания акции |

**Правило расчёта:** `price ≈ oldPrice × (1 − discount/100)`. `oldPrice = price`, если `discount = 0`.

### Отзывы и рейтинг 
| Поле | Тип | Описание |
|---|---|---|
| `rating` | number | Средний рейтинг (0.0–5.0), округлён до 0.1 |
| `reviewsCount` | number | Общее количество отзывов |
| `ratingBreakdown` | object | Распределение оценок: `{"5": N, "4": N, "3": N, "2": N, "1": N}` |
| `photosInReviews` | number | Сколько отзывов с фотографиями |
| `recommendRate` | number | % покупателей, рекомендующих товар (0–100) |

**Инварианты:**
- `ratingBreakdown[5] + [4] + [3] + [2] + [1] = reviewsCount`
- `rating ≈ (5×N5 + 4×N4 + 3×N3 + 2×N2 + 1×N1) / reviewsCount` (с округлением до 0.1)
- `photosInReviews ≤ reviewsCount`
- `recommendRate ≈ round(rating × 20)` для согласованности

### Наличие
| Поле | Тип | Описание |
|---|---|---|
| `inStock` | boolean | Есть ли товар в наличии |

### Медиа
| Поле | Тип | Описание |
|---|---|---|
| `image` | string (URL) | Ссылка на главное изображение (600×600) |

Для моков используется `https://loremflickr.com/600/600/{keywords}?lock={id}` — стабильная тематическая картинка (ключ `lock` фиксирует результат запроса).

### Служебное
| Поле | Тип | Описание |
|---|---|---|
| `createdAt` | string (ISO date) | Дата добавления товара в каталог |

## Соглашения по генерации моков

- `oldPrice = discount > 0 ? round(price / (1 − discount/100)) : price`
- `costPrice = round(price × 0.6)` — условная маржа 40 %
- `bonusPoints = round(price × 0.01)` — 1 % кэшбэка
- `vat = 10` для категорий «Продукты» и «Медицина», иначе `20`
- `promoLabel = rating ≥ 4.8 ? "Хит продаж" : discount ≥ 15 ? "Скидка" : null`
- `promoEndsAt = promoLabel ? "2024-06-30" : null`
- `photosInReviews = round(reviewsCount × 0.15)`
- `recommendRate = round(rating × 20)`
- `ratingBreakdown` распределяется по весам, зависящим от `rating`:

  | rating | 5★  | 4★  | 3★  | 2★  | 1★  |
  |---|---|---|---|---|---|
  | 4.9 | 82% | 13% | 3%  | 1%  | 1%  |
  | 4.8 | 78% | 15% | 4%  | 2%  | 1%  |
  | 4.7 | 72% | 19% | 5%  | 2%  | 2%  |
  | 4.6 | 66% | 23% | 6%  | 3%  | 2%  |
  | 4.5 | 60% | 25% | 8%  | 4%  | 3%  |
  | 4.4 | 53% | 27% | 11% | 5%  | 4%  |
  | 4.3 | 48% | 28% | 13% | 6%  | 5%  |

  Затем доли округляются до целых, а недостающая единица компенсируется в колонке `1★`.