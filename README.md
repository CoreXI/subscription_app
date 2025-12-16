# subscription_app
Описание ios приложения с подпиской 

## Оглавление

- [Архитектура проекта](#архитектура-проекта)
  - [Общая структура](#общая-структура)
  - [Инфраструктура](#инфраструктура)
  - [База данных](#база-данных-postgresql)
  - [Backend API](#backend-api-rest--websocket-для-real-time)
  - [Frontend (iOS)](#frontend-ios---экраны)
  - [Логика работы](#логика-работы)
  - [Дополнительные сервисы](#дополнительные-сервисыинтеграции)
- [Оценка времени реализации](#реализация-в-рамках-команды-без-использования-ии)
  - [Разбивка по этапам](#разбивка-по-этапам)
  - [Итоговые оценки](#итого)
  - [MVP версия](#mvp-версия-упрощенная)

## Экраны со стороны пользовтеля 

<img width="200" alt="IMG_9398" src="https://github.com/user-attachments/assets/78a6b5a8-c727-4209-90fc-2283b1a459c0" />
<img width="200" alt="IMG_9397" src="https://github.com/user-attachments/assets/9a8a91d2-67ba-46f9-b4d9-cd5474920ce5" />
<img width="200" alt="IMG_9386" src="https://github.com/user-attachments/assets/0d6f9a35-c71a-4588-b13b-06ba2ff6ecdb" />
<img width="200" alt="IMG_9388" src="https://github.com/user-attachments/assets/63d7dda1-dbaf-4eb6-8e25-7875af7f3f75" />
<img width="200" alt="IMG_9389" src="https://github.com/user-attachments/assets/a68e3a0a-4c94-42de-a215-5417ed3a5023" />
<img width="200" alt="IMG_9390" src="https://github.com/user-attachments/assets/d68ad796-4f91-444c-9cef-ffd34c933c8c" />
<img width="200" alt="IMG_9391" src="https://github.com/user-attachments/assets/7aa86427-30f2-4232-84be-5c8ec3d45452" />
<img width="200" alt="IMG_9392" src="https://github.com/user-attachments/assets/e1c43590-796f-4564-8661-2793f693d920" />
<img width="200" alt="IMG_9393" src="https://github.com/user-attachments/assets/b29ea3b5-8d5f-4f88-af15-798f04d9a58b" />
<img width="200" alt="IMG_9394" src="https://github.com/user-attachments/assets/20b09f3e-7af2-4406-89c4-fde2aeeca8a3" />
<img width="200" alt="IMG_9395" src="https://github.com/user-attachments/assets/d0d853d3-813a-4582-83fc-411cee25ec01" />
<img width="200" alt="IMG_9396" src="https://github.com/user-attachments/assets/3857aa2f-5f44-4c86-b046-b9cf67c589f2" />




## Архитектура проекта

### Общая структура
- iOS приложение (Swift/SwiftUI)
- Backend API (Node.js/NestJS или Python/FastAPI)
- База данных (PostgreSQL)
- Файловое хранилище (S3/MinIO для фото)
- Push-уведомления (Firebase Cloud Messaging)
- Платежная система (Stripe/YooKassa)

---

## Инфраструктура

### Для iOS:
- **CI/CD**: GitHub Actions / GitLab CI
- **Distribution**: TestFlight (beta) + App Store Connect
- **Analytics**: Firebase Analytics / Amplitude
- **Crash reporting**: Sentry / Firebase Crashlytics
- **Maps**: Apple Maps / Yandex Maps SDK
- **Push**: Firebase Cloud Messaging
- **Keychain**: для хранения токенов

### Backend инфраструктура:
- **Hosting**: AWS / DigitalOcean / Yandex Cloud
- **Container**: Docker + Docker Compose
- **Reverse proxy**: Nginx
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Backup**: Автоматический бэкап БД (pg_dump) + S3

---

## База данных (PostgreSQL)

### Таблицы (примерно 15-18):

1. **users** — пользователи
   - id, phone, name, email, created_at, updated_at, is_active, fcm_token

2. **addresses** — адреса пользователей
   - id, user_id, city, street, house, apartment, entrance, floor, coordinates (lat, lng), is_default, created_at

3. **subscriptions** — подписки
   - id, user_id, address_id, plan_id, status (active/paused/cancelled), start_date, end_date, next_pickup_date, created_at

4. **subscription_plans** — тарифы подписок
   - id, name, description, price, frequency (weekly/biweekly/monthly), pickup_count, is_active

5. **orders** — разовые заказы
   - id, user_id, address_id, type (regular/construction/cleaning), status (pending/in_progress/completed/cancelled), scheduled_date, completed_at, created_at

6. **pickups** — выносы мусора (для подписок)
   - id, subscription_id, scheduled_date, status (scheduled/completed/skipped), completed_at, worker_id, notes

7. **payments** — платежи
   - id, user_id, subscription_id/order_id, amount, currency, status (pending/success/failed), payment_method, transaction_id, created_at

8. **news** — новости
   - id, title, description, image_url, link_url, is_active, published_at, created_at

9. **promo_codes** — промокоды
   - id, code, discount_type (percent/fixed), discount_value, max_uses, used_count, valid_from, valid_until, is_active

10. **user_promo_codes** — использованные промокоды
    - id, user_id, promo_code_id, used_at

11. **workers** — работники
    - id, name, phone, email, is_active, created_at

12. **worker_assignments** — назначения работников
    - id, worker_id, pickup_id/order_id, assigned_at, completed_at

13. **notifications** — уведомления
    - id, user_id, type, title, message, is_read, created_at

14. **support_tickets** — обращения в поддержку
    - id, user_id, subject, message, status (open/in_progress/resolved), created_at, updated_at

15. **app_settings** — настройки приложения
    - id, key, value, description

16. **audit_logs** — логи действий (опционально)
    - id, user_id, action, entity_type, entity_id, details, created_at

---

## Backend API (REST + WebSocket для real-time)

### Роуты (примерно 40-50 эндпоинтов):

#### Auth (5):
- `POST /api/auth/send-code` — отправка SMS кода
- `POST /api/auth/verify-code` — верификация кода
- `POST /api/auth/refresh` — обновление токена
- `POST /api/auth/logout` — выход
- `DELETE /api/auth/account` — удаление аккаунта

#### User (5):
- `GET /api/user/profile` — профиль
- `PUT /api/user/profile` — обновление профиля
- `GET /api/user/addresses` — список адресов
- `POST /api/user/addresses` — добавление адреса
- `PUT /api/user/addresses/:id` — обновление адреса
- `DELETE /api/user/addresses/:id` — удаление адреса

#### Subscriptions (8):
- `GET /api/subscriptions/plans` — список тарифов
- `GET /api/subscriptions/my` — мои подписки
- `POST /api/subscriptions` — создание подписки
- `PUT /api/subscriptions/:id/pause` — пауза
- `PUT /api/subscriptions/:id/resume` — возобновление
- `DELETE /api/subscriptions/:id` — отмена
- `GET /api/subscriptions/:id/pickups` — история выносов
- `PUT /api/subscriptions/:id/address` — смена адреса

#### Orders (6):
- `POST /api/orders` — создание разового заказа
- `GET /api/orders/my` — мои заказы
- `GET /api/orders/:id` — детали заказа
- `PUT /api/orders/:id/cancel` — отмена заказа
- `GET /api/orders/active` — активные заказы
- `GET /api/orders/history` — история

#### Payments (4):
- `POST /api/payments/create` — создание платежа
- `POST /api/payments/webhook` — webhook от платежной системы
- `GET /api/payments/history` — история платежей
- `GET /api/payments/:id` — детали платежа

#### Promo Codes (3):
- `POST /api/promo-codes/validate` — проверка промокода
- `POST /api/promo-codes/apply` — применение промокода
- `GET /api/promo-codes/my` — использованные промокоды

#### News (2):
- `GET /api/news` — список новостей
- `GET /api/news/:id` — детали новости

#### Notifications (3):
- `GET /api/notifications` — список уведомлений
- `PUT /api/notifications/:id/read` — отметить прочитанным
- `PUT /api/notifications/read-all` — отметить все прочитанными

#### Maps (2):
- `POST /api/maps/geocode` — геокодирование адреса
- `POST /api/maps/reverse-geocode` — обратное геокодирование

#### Support (3):
- `POST /api/support/tickets` — создание обращения
- `GET /api/support/tickets` — список обращений
- `GET /api/support/tickets/:id` — детали обращения

---

### Admin API (для продавца) — примерно 20 эндпоинтов:

#### Dashboard (3):
- `GET /api/admin/dashboard/stats` — статистика
- `GET /api/admin/dashboard/revenue` — доходы
- `GET /api/admin/dashboard/active-subscriptions` — активные подписки

#### Users (4):
- `GET /api/admin/users` — список пользователей
- `GET /api/admin/users/:id` — детали пользователя
- `GET /api/admin/users/:id/subscriptions` — подписки пользователя
- `GET /api/admin/users/:id/orders` — заказы пользователя

#### Subscriptions Management (5):
- `GET /api/admin/subscriptions` — все подписки
- `GET /api/admin/subscriptions/:id` — детали подписки
- `PUT /api/admin/subscriptions/:id/status` — изменение статуса
- `POST /api/admin/subscriptions/:id/pickup` — отметить вынос
- `PUT /api/admin/subscriptions/:id/assign-worker` — назначить работника

#### Orders Management (4):
- `GET /api/admin/orders` — все заказы
- `GET /api/admin/orders/:id` — детали заказа
- `PUT /api/admin/orders/:id/status` — изменение статуса
- `PUT /api/admin/orders/:id/assign-worker` — назначить работника

#### Workers (4):
- `GET /api/admin/workers` — список работников
- `POST /api/admin/workers` — создание работника
- `PUT /api/admin/workers/:id` — обновление работника
- `GET /api/admin/workers/:id/assignments` — назначения работника

---

## Frontend (iOS) — экраны

### Экраны с логикой (15-18):

1. **AuthFlow:**
   - `PhoneInputScreen` — ввод телефона
   - `CodeVerificationScreen` — ввод кода

2. **MainFlow:**
   - `HomeScreen` — главная (новости, кнопки услуг, часы работы)
   - `NewsDetailScreen` — детали новости
   - `ServiceSelectionScreen` — выбор услуги (разовый/подписка)

3. **AddressFlow:**
   - `AddressListScreen` — список адресов
   - `AddressMapScreen` — карта выбора адреса
   - `AddressFormScreen` — форма адреса

4. **SubscriptionFlow:**
   - `SubscriptionPlansScreen` — выбор тарифа
   - `SubscriptionDetailsScreen` — детали подписки
   - `SubscriptionPaymentScreen` — оплата подписки
   - `MySubscriptionsScreen` — мои подписки

5. **OrdersFlow:**
   - `OrdersListScreen` — список заказов (активные/история)
   - `OrderDetailsScreen` — детали заказа
   - `OrderCreationScreen` — создание разового заказа

6. **ProfileFlow:**
   - `ProfileScreen` — профиль
   - `EditProfileScreen` — редактирование профиля
   - `PromoCodesScreen` — промокоды
   - `SupportScreen` — поддержка

### Экраны без сложной логики (5-7):

1. `SplashScreen` — загрузка
2. `OnboardingScreen` — онбординг (опционально)
3. `PaymentSuccessScreen` — успешная оплата
4. `PaymentFailedScreen` — ошибка оплаты
5. `SettingsScreen` — настройки
6. `AboutScreen` — о приложении
7. `TermsScreen` — условия использования

---

## Логика работы

### 1. Регистрация/Авторизация:
```
- Пользователь вводит телефон
- Отправка SMS кода через SMS-провайдер (Twilio/SMS.ru)
- Верификация кода (6 цифр, валидность 5 минут)
- Создание/обновление пользователя в БД
- Генерация JWT токена (access + refresh)
- Сохранение FCM токена для push-уведомлений
```

### 2. Выбор адреса:
```
- Открытие карты (Apple Maps / Yandex Maps)
- Поиск адреса через геокодинг API
- Выбор точки на карте → обратное геокодирование
- Сохранение адреса (город, улица, дом, квартира, координаты)
- Валидация адреса (проверка зоны обслуживания)
```

### 3. Создание подписки:
```
- Выбор тарифа (неделя/2 недели/месяц, количество выносов)
- Выбор адреса (или создание нового)
- Применение промокода (если есть)
- Расчет стоимости с учетом скидки
- Создание платежа через платежный шлюз
- После успешной оплаты:
  - Создание записи в subscriptions
  - Расчет next_pickup_date (по расписанию тарифа)
  - Создание записей в pickups на период подписки
  - Отправка push-уведомления
```

### 4. Разовый заказ:
```
- Выбор типа услуги (вынос/строительный/уборка)
- Выбор адреса
- Выбор даты/времени (в пределах рабочих часов)
- Расчет стоимости (базовая цена + тип услуги)
- Создание заказа в БД (status: pending)
- Оплата
- После оплаты: статус → in_progress
- Назначение работника (автоматически или вручную через админку)
- Уведомление пользователя о назначении
```

### 5. Выполнение выноса (подписка):
```
- Работник отмечает выполнение в админке
- Обновление статуса pickup → completed
- Обновление next_pickup_date подписки
- Создание следующего pickup (если подписка активна)
- Уведомление пользователя
- Обновление счетчика выполненных выносов
```

### 6. Платежи:
```
- Интеграция с платежным шлюзом (Stripe/YooKassa)
- Создание платежной сессии
- Редирект на страницу оплаты
- Webhook от платежной системы:
  - Валидация подписи
  - Обновление статуса платежа
  - Активация подписки/заказа
  - Отправка уведомления
```

### 7. Промокоды:
```
- Валидация промокода:
  - Проверка существования и активности
  - Проверка срока действия
  - Проверка лимита использований
  - Проверка на повторное использование (user_promo_codes)
- Применение промокода:
  - Расчет скидки (процент или фиксированная сумма)
  - Сохранение связи user_promo_code
  - Обновление стоимости заказа/подписки
```

### 8. Уведомления:
```
- Push-уведомления через FCM:
  - Новый заказ создан
  - Заказ назначен работнику
  - Заказ выполнен
  - Подписка активирована
  - Ближайший вынос (за день до)
  - Платеж успешен/неуспешен
- In-app уведомления (сохранение в БД)
```

### 9. Админка (продавец):
```
- Dashboard:
  - Статистика (активные подписки, заказы за день/неделю/месяц)
  - Выручка
  - Графики
- Управление подписками:
  - Просмотр всех подписок
  - Фильтрация по статусу/дате
  - Отметка выполнения выноса
  - Пауза/возобновление подписки
  - Назначение работника
- Управление заказами:
  - Просмотр всех заказов
  - Изменение статуса
  - Назначение работника
  - Просмотр деталей (адрес, контакты пользователя)
- Управление пользователями:
  - Поиск по телефону/имени
  - Просмотр истории подписок/заказов
  - Блокировка/разблокировка
```

### 10. Расписание выносов:
```
- Автоматический расчет next_pickup_date:
  - Weekly: +7 дней от последнего выноса
  - Biweekly: +14 дней
  - Monthly: +30 дней
- Учет выходных/праздников (опционально)
- Создание pickup записей на месяц вперед при создании подписки
- Автоматическое создание следующего pickup после выполнения
```

### 11. Бэкапирование:
```
- Ежедневный бэкап БД (pg_dump):
  - Полный бэкап раз в день (3:00 AM)
  - Хранение последних 30 дней
  - Загрузка в S3/облачное хранилище
- Инкрементальные бэкапы (WAL архивирование)
- Бэкап файлов (фото новостей, документы):
  - Синхронизация с S3
  - Версионирование
- Мониторинг бэкапов:
  - Проверка успешности бэкапа
  - Алерты при ошибках
  - Тестовое восстановление раз в месяц
```

### 12. Безопасность:
```
- JWT токены (access: 15 мин, refresh: 7 дней)
- Хеширование паролей (bcrypt) - если будет пароль
- Rate limiting на API (100 req/min на пользователя)
- Валидация всех входных данных
- HTTPS только
- Защита от SQL injection (ORM/параметризованные запросы)
- CORS настройки
- Защита админки (отдельная авторизация, 2FA опционально)
```

---

## Дополнительные сервисы/интеграции:

1. SMS-провайдер (Twilio/SMS.ru) — коды авторизации
2. Платежный шлюз (Stripe/YooKassa) — оплата
3. Карты (Apple Maps/Yandex Maps) — выбор адреса
4. Push (Firebase Cloud Messaging) — уведомления
5. Analytics (Firebase Analytics) — аналитика
6. Crash reporting (Sentry) — отслеживание ошибок

---

Итог: ~18 таблиц БД, ~70 API эндпоинтов (50 клиент + 20 админ), ~25 экранов iOS (18 с логикой + 7 простых), автоматическое бэкапирование БД и файлов.



## Реализация в рамках команды без использования ИИ

Оценка времени реализации (команда: 1 iOS, 1 Backend, 1 Fullstack для админки):

## Разбивка по этапам:

### 1. Backend (БД + API) — 6-8 недель

**База данных (1 неделя):**
- Схема БД (18 таблиц) — 2 дня
- Миграции, индексы, связи — 2 дня
- Seed данные, тестовые данные — 1 день

**API клиентская часть (3-4 недели):**
- Auth модуль (5 эндпоинтов) — 3 дня
- User модуль (6 эндпоинтов) — 3 дня
- Subscriptions модуль (8 эндпоинтов) — 5 дней
- Orders модуль (6 эндпоинтов) — 4 дня
- Payments модуль (4 эндпоинта) — 3 дня
- Promo codes (3 эндпоинта) — 2 дня
- News (2 эндпоинта) — 1 день
- Notifications (3 эндпоинта) — 2 дня
- Maps/Geocoding (2 эндпоинта) — 2 дня
- Support (3 эндпоинта) — 2 дня
- Тестирование, рефакторинг — 3 дня

**Admin API (2 недели):**
- Dashboard (3 эндпоинта) — 2 дня
- Users management (4 эндпоинта) — 2 дня
- Subscriptions management (5 эндпоинтов) — 3 дня
- Orders management (4 эндпоинта) — 2 дня
- Workers management (4 эндпоинта) — 2 дня
- Тестирование — 2 дня

**Бизнес-логика (1 неделя):**
- Расписание выносов, автоматизация — 3 дня
- Уведомления (push, in-app) — 2 дня

---

### 2. iOS приложение — 8-10 недель

**Настройка проекта (3 дня):**
- Архитектура (MVVM/Coordinator), навигация — 2 дня
- UI Kit, компоненты — 1 день

**Auth Flow (1 неделя):**
- PhoneInputScreen — 2 дня
- CodeVerificationScreen — 2 дня
- Интеграция SMS API — 1 день

**Main Flow (1.5 недели):**
- HomeScreen (новости, кнопки) — 3 дня
- NewsDetailScreen — 1 день
- ServiceSelectionScreen — 2 дня

**Address Flow (1.5 недели):**
- AddressListScreen — 2 дня
- AddressMapScreen (карты) — 3 дня
- AddressFormScreen — 2 дня

**Subscription Flow (2 недели):**
- SubscriptionPlansScreen — 2 дня
- SubscriptionDetailsScreen — 2 дня
- SubscriptionPaymentScreen (интеграция платежей) — 3 дня
- MySubscriptionsScreen — 2 дня

**Orders Flow (1.5 недели):**
- OrdersListScreen (табы) — 2 дня
- OrderDetailsScreen — 2 дня
- OrderCreationScreen — 2 дня

**Profile Flow (1 неделя):**
- ProfileScreen — 2 дня
- EditProfileScreen — 2 дня
- PromoCodesScreen — 1 день
- SupportScreen — 1 день

**Прочие экраны (3 дня):**
- SplashScreen, PaymentSuccess/Failed — 1 день
- Settings, About, Terms — 2 дня

**Интеграции (1 неделя):**
- Push-уведомления (FCM) — 2 дня
- Analytics — 1 день
- Crash reporting — 1 день
- Карты (Apple Maps/Yandex) — 2 дня

**Тестирование и полировка (1 неделя):**
- Тестирование на устройствах — 3 дня
- Багфиксы, оптимизация — 2 дня

---

### 3. Админка (Web) — 4-5 недель

**Настройка (2 дня):**
- React/Vue проект, роутинг — 1 день
- UI библиотека (Ant Design/Material UI) — 1 день

**Dashboard (1 неделя):**
- Статистика, графики — 3 дня
- Выручка, метрики — 2 дня

**Subscriptions Management (1.5 недели):**
- Список, фильтры — 3 дня
- Отметка выполнения, пауза/возобновление — 2 дня
- Назначение работника — 2 дня

**Orders Management (1 неделя):**
- Список, фильтры — 2 дня
- Изменение статуса — 2 дня
- Назначение работника — 1 день

**Users Management (1 неделя):**
- Список, поиск — 2 дня
- Детали пользователя — 2 дня
- История подписок/заказов — 1 день

**Workers Management (3 дня):**
- CRUD работников — 2 дня
- Назначения — 1 день

**Тестирование (3 дня)**

---

### 4. Интеграции — 2 недели

**Платежная система (1 неделя):**
- Интеграция Stripe/YooKassa — 3 дня
- Webhook обработка — 2 дня
- Тестирование платежей — 2 дня

**SMS провайдер (2 дня):**
- Интеграция SMS.ru/Twilio — 1 день
- Тестирование отправки — 1 день

**Push-уведомления (3 дня):**
- Настройка FCM — 1 день
- Интеграция в backend — 1 день
- Тестирование — 1 день

---

### 5. Инфраструктура и деплой — 2 недели

**DevOps (1 неделя):**
- Docker, docker-compose — 2 дня
- CI/CD настройка — 2 дня
- Мониторинг, логирование — 2 дня
- Nginx конфигурация — 1 день

**Бэкапирование (3 дня):**
- Автоматические бэкапы БД — 1 день
- Бэкап файлов в S3 — 1 день
- Мониторинг бэкапов — 1 день

**Деплой и настройка (4 дня):**
- Деплой backend — 1 день
- Деплой админки — 1 день
- Настройка окружений (dev/staging/prod) — 1 день
- SSL сертификаты, домены — 1 день

---

### 6. Тестирование и багфиксы — 2-3 недели

**Интеграционное тестирование (1 неделя):**
- Тестирование всех сценариев — 3 дня
- Тестирование платежей — 2 дня
- Тестирование уведомлений — 2 дня

**Багфиксы (1-2 недели):**
- Исправление критичных багов — 3-5 дней
- Исправление минорных багов — 2-3 дня
- Оптимизация производительности — 2 дня

---

## Итого:

| Этап | Время |
|------|-------|
| Backend | 6-8 недель |
| iOS приложение | 8-10 недель |
| Админка | 4-5 недель |
| Интеграции | 2 недели |
| Инфраструктура | 2 недели |
| Тестирование | 2-3 недели |
| **ИТОГО** | **24-30 недель** |

---

## С параллельной работой (команда 3-4 человека):

- Backend и iOS параллельно: 8-10 недель
- Админка параллельно с финальными этапами iOS: +4-5 недель
- Интеграции параллельно: +2 недели
- Инфраструктура параллельно: +2 недели
- Тестирование: +2-3 недели

**Итого с параллельной работой: 12-16 недель (3-4 месяца)**

---

## MVP версия (упрощенная):

Если нужен быстрый запуск MVP:

- Базовый Backend (без админки): 4 недели
- iOS MVP (основные экраны): 5 недель
- Простая админка (только просмотр): 2 недели
- Базовые интеграции: 1 неделя
- Простая инфраструктура: 1 неделя

**MVP: 8-10 недель (2-2.5 месяца)**

---

## Риски, увеличивающие срок:

- Изменения требований: +20-30%
- Проблемы с интеграциями: +1-2 недели
- Сложности с платежами: +1 неделя
- Проблемы с картами/геокодингом: +3-5 дней
- Масштабирование/производительность: +1-2 недели

**Реалистичная оценка с учетом рисков: 14-18 недель (3.5-4.5 месяца)**
