## Документация Laravel 9.x

Вольный перевод репозитория документации [**laravel/docs**](https://github.com/laravel/docs/tree/9.x) ветка 9.x на русский язык. Актуализация с основным репозиторием документации Laravel осуществляется не реже одного раза в месяц. Орфографические, пунктуационные и грубые смысловые ошибки исправляются по мере их выявления.

> Перевод документации [Laravel ветка 8.x](https://github.com/russsiq/laravel-docs-ru/tree/8.x).

<a name="navigation"></a>
### Содержание документации

> Перед чтением документации ознакомьтесь с [соглашением по переводу](https://github.com/russsiq/translation-agreements).

Разделы, помеченные галочками, уже имеют актуализированные переводы **полностью на русском языке**.

- #### Пролог
    - [x] [Примечания к релизу](./docs/releases.md)
    - [x] [Руководство по обновлению](./docs/upgrade.md)
    - [x] [Рекомендации по участию](./docs/contributions.md)
- #### Начало
    - [x] [Установка](./docs/installation.md)
    - [x] [Конфигурирование](./docs/configuration.md)
    - [x] [Структура каталогов](./docs/structure.md)
    - [ ] [Внешний интерфейс приложения](./docs/frontend.md)
    - [x] [Стартовые комплекты](./docs/starter-kits.md)
    - [x] [Развертывание](./docs/deployment.md)
- #### Архитектурные концепции
    - [x] [Жизненный цикл запроса](./docs/lifecycle.md)
    - [x] [Контейнер служб](./docs/container.md)
    - [x] [Поставщики служб](./docs/providers.md)
    - [x] [Фасады](./docs/facades.md)
- #### Основы
    - [x] [Маршрутизация](./docs/routing.md)
    - [x] [Посредники](./docs/middleware.md)
    - [x] [Предотвращение атак CSRF](./docs/csrf.md)
    - [x] [Контроллеры](./docs/controllers.md)
    - [x] [HTTP-запросы](./docs/requests.md)
    - [x] [HTTP-ответы](./docs/responses.md)
    - [x] [HTML-шаблоны](./docs/views.md)
    - [x] [Шаблонизатор Blade](./docs/blade.md)
    - [ ] [Объединение веб-активов](./docs/vite.md)
    - [x] [Генерация URL-адресов](./docs/urls.md)
    - [x] [Сессия HTTP](./docs/session.md)
    - [x] [Валидация](./docs/validation.md)
    - [x] [Обработка ошибок](./docs/errors.md)
    - [x] [Логирование](./docs/logging.md)
- #### Продвинутое руководство
    - [x] [Консоль Artisan](./docs/artisan.md)
    - [x] [Трансляция событий](./docs/broadcasting.md)
    - [x] [Кеш приложения](./docs/cache.md)
    - [x] [Коллекции](./docs/collections.md)
    - [x] [Контракты](./docs/contracts.md)
    - [x] [События](./docs/events.md)
    - [x] [Файловое хранилище](./docs/filesystem.md)
    - [x] [Глобальные помощники](./docs/helpers.md)
    - [x] [HTTP-клиент](./docs/http-client.md)
    - [x] [Локализация интерфейса](./docs/localization.md)
    - [x] [Почтовые отправления](./docs/mail.md)
    - [x] [Уведомления](./docs/notifications.md)
    - [x] [Разработка пакетов](./docs/packages.md)
    - [x] [Очереди](./docs/queues.md)
    - [x] [Ограничители частоты](./docs/rate-limiting.md)
    - [x] [Планирование задач](./docs/scheduling.md)
- #### Безопасность
    - [x] [Аутентификация](./docs/authentication.md)
    - [x] [Авторизация](./docs/authorization.md)
    - [x] [Подтверждение адреса электронной почты](./docs/verification.md)
    - [x] [Шифрование](./docs/encryption.md)
    - [x] [Хеширование](./docs/hashing.md)
    - [x] [Сброс пароля](./docs/passwords.md)
- #### База данных
    - [x] [Начало работы](./docs/database.md)
    - [x] [Построитель запросов](./docs/queries.md)
    - [x] [Постраничная навигация](./docs/pagination.md)
    - [x] [Миграции](./docs/migrations.md)
    - [x] [Наполнение фиктивными данными](./docs/seeding.md)
    - [x] [Использование Redis](./docs/redis.md)
- #### Eloquent ORM
    - [x] [Начало работы](./docs/eloquent.md)
    - [x] [Отношения](./docs/eloquent-relationships.md)
    - [x] [Коллекции](./docs/eloquent-collections.md)
    - [x] [Мутаторы и типизация](./docs/eloquent-mutators.md)
    - [x] [Ресурсы API](./docs/eloquent-resources.md)
    - [x] [Сериализация](./docs/eloquent-serialization.md)
    - [ ] [Фабрики](./docs/eloquent-factories.md)
- #### Тестирование
    - [x] [Начало работы](./docs/testing.md)
    - [x] [Тесты HTTP](./docs/http-tests.md)
    - [x] [Тесты консольных команд](./docs/console-tests.md)
    - [x] [Браузерные тесты](./docs/dusk.md)
    - [x] [База данных](./docs/database-testing.md)
    - [x] [Имитация](./docs/mocking.md)
- #### Пакеты
    - [x] [*Breeze*](./docs/starter-kits.md#laravel-breeze) – легковесная реализация аутентификации Laravel для ознакомления с функционалом. Включает простые шаблоны Blade, стилизованные с помощью Tailwind CSS. Содержит маршруты для публикации.
    <!-- - [ ] [*Cashier (Stripe)*](./docs/billing.md) -->
    <!-- - [ ] [*Cashier (Paddle)*](./docs/cashier-paddle.md) -->
    - [x] [*Dusk*](./docs/dusk.md) – автоматизация поведения браузера и тестирование с использованием ChromeDriver.
    - [x] [*Envoy*](./docs/envoy.md) – инструмент для запуска задач, выполняемых на удаленных серверах. Задачи определяются в файле `Envoy.blade.php` в корне приложения с использованием директив шаблонизатора Blade.
    - [x] [*Fortify*](./docs/fortify.md) – серверная реализация аутентификации Laravel. Не содержит никаких шаблонов. Используется в Laravel Jetstream.
    - [x] [*Homestead*](./docs/homestead.md) – официальный образ Vagrant для приложений Laravel.
    - [x] [*Horizon*](./docs/horizon.md) – панель управления и конфигурация очередей, использующих Redis.
    - [ ] [*Jetstream*](https://jetstream.laravel.com) – красиво оформленный каркас приложений. Включает в себя Fortify и Sanctum.
    - [x] [*Mix*](./docs/mix.md) – гибкий API для определения шагов сборки Webpack; упрощает компиляцию и минимизацию файлов CSS и JavaScript.
    - [ ] [*Octane*](./docs/octane.md) – повышает производительность вашего приложения с использованием мощных серверов [Swoole](https://swoole.co.uk) и [RoadRunner](https://roadrunner.dev)
    - [ ] [*Passport*](./docs/passport.md) – реализация сервера OAuth2 для вашего приложения Laravel на основе [League OAuth2](https://github.com/thephpleague/oauth2-server).
    - [ ] [*Pint*](./docs/pint.md) – is an opinionated PHP code style fixer for minimalists.
    - [x] [*Sail*](./docs/sail.md) – CLI для взаимодействия со средой разработки Docker.
    - [x] [*Sanctum*](./docs/sanctum.md) – легковесная система аутентификации для SPA (одностраничных приложений), мобильных приложений и простых API на основе токенов. Управление токенами API, аутентификация сессии. Не содержит никаких шаблонов. Используется в Laravel Jetstream.
    - [x] [*Scout*](./docs/scout.md) – «простое» решение на основе драйверов для добавления полнотекстового поиска моделям Eloquent.
    - [x] [*Socialite*](./docs/socialite.md) – аутентификация через провайдеров OAuth: Facebook, Twitter, LinkedIn, Google, GitHub, GitLab и Bitbucket.
    - [x] [*Telescope*](./docs/telescope.md) – панель управления, отображающая записи о произошедших в приложении событиях.
    - [ ] [*Valet*](./docs/valet.md) – окружение разработки приложений Laravel для пользователей macOS.
- [Документация API](https://laravel.com/api/9.x/)

<a name="contributing"></a>
### Содействие переводу

Официальная документация Laravel доступна только на английском языке. Для получения дополнительной информации о том, как вы можете внести свой вклад в перевод документации Laravel на русский язык, пожалуйста, прочтите [Содействие в переводе документации](CONTRIBUTING.md).

<a name="license"></a>
### Лицензия

Ссылка на [лицензию](https://github.com/laravel/docs/blob/9.x/license.md) оригинала документации **laravel/docs**.

Текущий перевод документации распространяется по лицензии [MIT](LICENSE).
