# Laravel 8 · Стартовые комплекты

- [Введение](#introduction)
- [Laravel Breeze](#laravel-breeze)
    - [Установка](#laravel-breeze-installation)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## Введение

Чтобы дать вам фору при создании нового приложения Laravel, мы рады предложить стартовые комплекты приложения и, в частности, аутентификации. Эти комплекты автоматически дополнят ваше приложение маршрутами, контроллерами и шаблонами, необходимыми для регистрации и аутентификации пользователей вашего приложения.

Вы можете использовать эти стартовые комплекты, но они не требуются. Вы можете создать собственное приложение с нуля, просто установив новую копию Laravel. В любом случае, мы знаем, что вы создадите что-то отличное!

<a name="laravel-breeze"></a>
## Laravel Breeze

**Laravel Breeze** – это минимальная и простая реализация всего [функционала аутентификации](authentication.md) Laravel, включая вход в систему, регистрацию, сброс пароля, подтверждение адреса электронной почты и пароля. Слой «View» комплекта Laravel Breeze по умолчанию состоит из простых [шаблонов Blade](blade.md), стилизованных с помощью [Tailwind CSS](https://tailwindcss.com). Breeze является прекрасной отправной точкой для создания нового приложения Laravel.

<a name="laravel-breeze-installation"></a>
### Установка

Сначала вы должны [создать новое приложение Laravel](installation.md), настроить свою базу данных и запустить [миграции базы данных](migrations.md):

```bash
curl -s https://laravel.build/example-app | bash

cd example-app

php artisan migrate
```

Создав новое приложение Laravel, вы можете установить Laravel Breeze с помощью Composer:

```bash
composer require laravel/breeze --dev
```

После того, как Composer установит пакет Laravel Breeze, вы можете запустить команду `breeze:install` Artisan. Эта команда опубликует для вашего приложения шаблоны, маршруты, контроллеры и другие ресурсы аутентификации. Laravel Breeze опубликует весь свой код в вашем приложении, чтобы у вас был полный контроль, а также обзор всего функционала и его реализации. После установки Breeze вы также должны скомпилировать свои исходники, чтобы был доступен файл стилей вашего приложения:

```bash
php artisan breeze:install

npm install

npm run dev

php artisan migrate
```

Затем, вы можете перейти в своем веб-браузере по URL-адресам вашего приложения `/login` или `/register`. Все маршруты Breeze определены в файле `routes/auth.php`.

> {tip} Чтобы узнать больше о компиляции CSS и JavaScript вашего приложения, ознакомьтесь с [документацией Laravel Mix](mix.md#running-mix).

<a name="breeze-and-inertia"></a>
#### Breeze и Inertia

Laravel Breeze также предлагает реализацию внешнего интерфейса [Inertia.js](https://inertiajs.com) на базе Vue. Чтобы использовать стек Inertia, передайте параметр `--inertia` при выполнении команды `breeze:install` Artisan:

```bash
php artisan breeze:install --inertia

npm install

npm run dev

php artisan migrate
```

<a name="laravel-jetstream"></a>
## Laravel Jetstream

В то время как Laravel Breeze обеспечивает простую и минимальную отправную точку для создания приложения Laravel, Jetstream дополняет эту функциональность более надежными функциями и дополнительными стеками технологий клиентского интерфейса. **Для тех, кто новичок в Laravel, мы рекомендуем изучить основы работы с Laravel Breeze перед тем, как перейти на Laravel Jetstream.**

Jetstream предлагает красиво оформленный каркас приложений для Laravel и включает в себя вход в систему, регистрацию, подтверждение адреса электронной почты, двухфакторную аутентификацию, управление сессиями, поддержку API через Laravel Sanctum, и дополнительно, управление командой. Jetstream разработан с использованием [Tailwind CSS](https://tailwindcss.com) и предлагает на ваш выбор каркас клиентского интерфейса под управлением [Livewire](https://laravel-livewire.com) либо [Inertia.js](https://inertiajs.com).

Полное описание по установке Laravel Jetstream можно найти в [официальной документации Jetstream](https://jetstream.laravel.com).
