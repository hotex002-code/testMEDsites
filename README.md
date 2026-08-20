# Тестовая версия

Самостоятельный B2B/B2C-каталог лабораторного и медицинского оборудования с формированием заявок, личным кабинетом и CRM.

Это не интернет-магазин. В публичном интерфейсе нет цен, оплаты, checkout или автоматического расчёта доставки. Основной сценарий:

```text
каталог → товар → заявка → контакты → CRM менеджера
```

## Что реализовано

- главная, 8 категорий и 32 demo-товара;
- поиск по названию, артикулу, бренду и описанию;
- фильтры категории и статуса поставки;
- карточка товара с вариантами, характеристиками, документами и похожими моделями;
- локальный список заявки, гостевая отправка и быстрый запрос;
- вход клиентов через Яндекс ID, служебный email/password-вход для сотрудников и httpOnly-сессии;
- кабинет клиента с редактируемыми контактами, полной историей их изменений и собственными заявками;
- CRM для ADMIN/MANAGER: статусы, история, внутренние заметки и клиенты;
- отдельные карточки сотрудников: контакты, должность, подразделение, служебный логин и создание менеджеров администратором;
- безопасная установка и сброс временных паролей менеджеров с завершением их активных сессий;
- управление товарами, вариантами, характеристиками, категориями, ролями и настройками;
- audit log для чувствительных админ-операций;
- SMTP email и опциональный Telegram-провайдер;
- sitemap, robots, canonical, OpenGraph и schema.org Organization/WebSite/Product/BreadcrumbList;
- Yandex SmartCaptcha для гостевых заявок, rate limiting, honeypot, same-origin checks, security headers и серверная валидация;
- responsive UI и WCAG AA-проверки.

## Стек

Next.js 16 App Router, React 19, TypeScript, Tailwind CSS 4, PostgreSQL 16, Prisma, Zod, React Hook Form, Nodemailer, Vitest, Playwright и axe-core.

## Локальный запуск

Требования: Node.js 22+, npm 11+ и PostgreSQL 16+.

```bash
cp .env.example .env
npm ci
npm run db:generate
npm run db:migrate
npm run db:seed
npm run dev
```

Сайт будет доступен на `http://localhost:3000`.

Demo-аккаунты сотрудников создаются seed-скриптом. Пароли берутся из `.env`:

| Роль | Email | Переменная пароля |
|---|---|---|
| ADMIN | `admin@example.test` | `SEED_ADMIN_PASSWORD` |
| MANAGER | `manager@example.test` | `SEED_MANAGER_PASSWORD` |

Перед запуском в общем или production-окружении обязательно замените все demo-пароли.

Новые менеджеры создаются администратором в разделе «Сотрудники». Пароль показывается только в момент, когда администратор вводит его в форму: в базе хранится защищённый хеш, поэтому посмотреть действующий пароль в карточке нельзя. Если сотрудник забыл пароль, администратор задаёт новый временный пароль.

Для входа клиентов зарегистрируйте веб-приложение в Яндекс OAuth, добавьте Redirect URI `http://localhost:3000/api/auth/yandex/callback` и заполните `YANDEX_CLIENT_ID`, `YANDEX_CLIENT_SECRET`. В production используйте тот же путь на боевом домене и укажите его в `NEXT_PUBLIC_APP_URL`.

Для гостевых заявок создайте CAPTCHA в Yandex Cloud и заполните `NEXT_PUBLIC_YANDEX_SMARTCAPTCHA_CLIENT_KEY`, `YANDEX_SMARTCAPTCHA_SERVER_KEY`. Без ключей локальная разработка использует явно обозначенный тестовый режим; production работает с запретом отправки до успешной проверки.

## Docker

```bash
cp .env.example .env
docker compose up --build -d
docker compose run --rm migrate npx prisma db seed
```

`migrate` ждёт healthcheck PostgreSQL и применяет production-миграции до старта web-контейнера. Seed запускается отдельно, чтобы production-окружение не заполнялось demo-данными автоматически.

## Уведомления

Для email заполните `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `SMTP_FROM` и `MANAGER_EMAIL`.

Для Telegram заполните `TELEGRAM_BOT_TOKEN` и `TELEGRAM_CHAT_ID`.

Провайдеры запускаются после транзакции создания заявки. Их ошибка логируется, но не удаляет и не откатывает уже созданную заявку.

## Проверки

```bash
npm run lint
npm run typecheck
npm run test
npm run build
npm run test:e2e
```

Перед первым E2E-запуском установите Chromium:

```bash
npx playwright install chromium
```

E2E-набор проверяет гостевую заявку, CRM-сценарий, отсутствие транзакционной коммерции, axe WCAG-аудит и отсутствие горизонтального overflow на 360, 390, 430, 768, 1440 и 1920 px.

## Архитектура

```text
UI / Route Handlers / Server Actions
                ↓
      Application Services
                ↓
          Repositories
                ↓
         Prisma / PostgreSQL
```

Основные каталоги:

```text
src/app/                 маршруты, API и server actions
src/components/          UI по доменам
src/lib/                 auth, db, validation, email, security, SEO config
src/server/services/     прикладные сервисы
src/server/repositories/ вся работа с БД
prisma/                  схема, миграция и seed
tests/                   unit, integration и Playwright E2E
```

Подробные принятые решения и критерии находятся в [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md).

## Production checklist

- заменить demo-пароли, контакты и `NEXT_PUBLIC_APP_URL`;
- установить `AUTH_COOKIE_SECURE=true` за HTTPS reverse proxy;
- указать production `DATABASE_URL`;
- настроить SMTP и, при необходимости, Telegram;
- настроить приложение Яндекс OAuth и оба ключа Yandex SmartCaptcha;
- заменить demo-товары и правовые тексты;
- для горизонтального масштабирования заменить in-memory rate limiter на Redis-адаптер.
