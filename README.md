[README.md](https://github.com/user-attachments/files/35097588/README.md)
# Car Audio Shop — Backend интернет-магазина автозвука

Серверное приложение (REST API) на **Java 17 + Spring Boot 3**.
Магазин продаёт автомагнитолы, акустику, сабвуферы, усилители и аксессуары.

## Проблема и аудитория
Небольшому магазину автозвука нужна система, где покупатели сами оформляют заказы,
менеджеры их обрабатывают, а склад обновляется автоматически (остатки списываются при заказе
и возвращаются при отмене).

## Роли
| Роль | Возможности |
|------|-------------|
| Гость | Просмотр каталога и категорий, регистрация |
| CUSTOMER | Оформление заказа, просмотр своих заказов, отмена своего нового заказа |
| MANAGER | Управление товарами и категориями, просмотр всех заказов, смена статусов |
| ADMIN | Всё то же + управление пользователями и ролями |

## Технологии
Java 17, Spring Boot 3.3, Spring Web, Spring Data JPA/Hibernate, Spring Security (HTTP Basic, BCrypt),
Bean Validation, H2 (по умолчанию) / PostgreSQL, Maven, Lombok.

## Сайт (frontend)
Одностраничный сайт `src/main/resources/static/index.html` отдаётся самим Spring Boot.
После запуска откройте http://localhost:8080 — каталог, корзина, заказы, панель менеджера и админа.

## Запуск
1. Установить JDK 17+ (в IntelliJ IDEA Maven встроен).
2. `mvn spring-boot:run` (или запустить класс `CarAudioApplication` в IDE).
3. Приложение: http://localhost:8080

По умолчанию используется **H2 в памяти** — настраивать ничего не нужно,
тестовые данные создаются автоматически при старте.

### PostgreSQL (по желанию)
1. Создать БД: `CREATE DATABASE caraudio;`
2. Поправить логин/пароль в `src/main/resources/application-postgres.properties`.
3. Запуск: `mvn spring-boot:run -Dspring-boot.run.profiles=postgres`

## Тестовые пользователи
| Логин | Пароль | Роль |
|-------|--------|------|
| admin | admin123 | ADMIN |
| manager | manager123 | MANAGER |
| customer | customer123 | CUSTOMER |

## API (авторизация — HTTP Basic)
| Метод | URL | Доступ |
|-------|-----|--------|
| POST | /api/auth/register | все |
| GET | /api/auth/me | авторизованные |
| GET | /api/categories | все |
| POST, DELETE | /api/categories, /api/categories/{id} | MANAGER, ADMIN |
| GET | /api/products, /api/products/{id}, ?categoryId=, ?q= | все |
| POST, PUT, DELETE | /api/products, /api/products/{id} | MANAGER, ADMIN |
| POST | /api/orders | авторизованные |
| GET | /api/orders/my | авторизованные |
| POST | /api/orders/{id}/cancel | владелец заказа |
| GET | /api/orders | MANAGER, ADMIN |
| PATCH | /api/orders/{id}/status?status=PAID | MANAGER, ADMIN |
| GET | /api/admin/users | ADMIN |
| PATCH | /api/admin/users/{id}/role?role=MANAGER | ADMIN |

Примеры тел запросов:
```json
POST /api/orders
{ "items": [ { "productId": 1, "quantity": 2 }, { "productId": 3, "quantity": 1 } ] }

POST /api/products
{ "name": "Новая колонка", "brand": "Alpine", "description": "16 см", "price": 5000, "stock": 10, "categoryId": 2 }
```

## Дополнительно
- **Избранное:** `GET /api/favorites`, `POST|DELETE /api/favorites/{productId}`
- **Отзывы:** `GET /api/reviews/product/{id}`, `POST /api/reviews/product/{id}` (1 отзыв на товар), `GET /api/reviews/summary`
- **Доставка:** при заказе можно передать `deliveryType` (`PICKUP`/`COURIER`), `address`, `phone`, `comment`. Курьер — 500 ₽, бесплатно от 30 000 ₽.
- **Скидки:** поле `discountPercent` у товара, итоговая цена — `finalPrice`.
- **Фото товаров:** загрузка `POST /api/products/upload` (файлы в папке `uploads/`) или ссылка/путь `/images/...` (папка `src/main/resources/static/images`).
- **Промокоды:** `GET /api/promo/check?code=&total=` (покупатель), управление `/api/promo` (MANAGER, ADMIN). Тестовые: `AUDIO10` (−10%), `WELCOME5` (−5%), `BASS15` (−15% от 10 000 ₽). В заказ передаётся `promoCode`.
- **Оплата (имитация):** `POST /api/orders/{id}/pay` — тестовая карта `4242 4242 4242 4242`, срок `12/30`, CVV `123`; карта на `0000` отклоняется.
- **Уведомления:** `GET /api/notifications`, `POST /api/notifications/read-all` — создаются при оформлении, оплате, отмене и смене статуса заказа.
- **Смена пароля:** `PATCH /api/auth/password`.
- **Статистика:** `GET /api/stats` (MANAGER, ADMIN) — заказы, выручка, топ товаров, остатки.
- База H2 хранится в папке `data/`. Чтобы сбросить данные — остановить приложение и удалить `data/`.

## База данных
- `users` (id, username, password, full_name, role)
- `category` (id, name)
- `product` (id, name, brand, description, price, stock, category_id → category)
- `orders` (id, user_id → users, status, created_at, total)
- `favorites` (id, user_id, product_id)
- `reviews` (id, user_id, product_id, rating, text, created_at)
- `promo_code` (id, code, percent, min_total, active)
- `notification` (id, user_id, text, created_at, seen)
- `order_item` (id, order_id → orders, product_id → product, quantity, price)

Связи: Category 1—N Product; User 1—N Order; Order 1—N OrderItem; Product 1—N OrderItem.

## Презентация
Будет в папке `docs/`.
