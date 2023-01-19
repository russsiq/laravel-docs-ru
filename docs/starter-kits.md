# Laravel 9 · Стартовые комплекты

- [Введение](#introduction)
- [Laravel Breeze](#laravel-breeze)
    - [Установка](#laravel-breeze-installation)
    - [Breeze и Blade](#breeze-and-blade)
    - [Breeze и React / Vue](#breeze-and-inertia)
    - [Breeze и Next.js / API](#breeze-and-next)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## Введение

Чтобы дать вам фору при создании нового приложения, Laravel предлагает стартовые комплекты приложения и, в частности, аутентификации. Эти комплекты автоматически дополнят ваше приложение маршрутами, контроллерами и шаблонами, необходимыми для регистрации и аутентификации пользователей вашего приложения.

Вы можете использовать эти стартовые комплекты, но они не требуются. Вы можете создать собственное приложение с нуля, просто установив новую копию Laravel. В любом случае, мы знаем, что вы создадите что-то отличное!

<a name="laravel-breeze"></a>
## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze) – это минимальная и простая реализация всего [функционала аутентификации](authentication.md) Laravel, включая вход в систему, регистрацию, сброс пароля, подтверждение адреса электронной почты и пароля. Кроме того, Breeze включает простую страницу «профиля», где пользователь может обновить свое имя, адрес электронной почты и пароль.

Слой «View» комплекта Laravel Breeze по умолчанию состоит из простых [шаблонов Blade](blade.md), стилизованных с помощью [Tailwind CSS](https://tailwindcss.com). Или Breeze может создать каркас вашего приложения с помощью Vue или React и [Inertia] (https://inertiajs.com).

Breeze является прекрасной отправной точкой для создания нового приложения Laravel, а также отличный выбор для проектов, которые планируют вывести использование шаблонов Blade на новый уровень с помощью [Laravel Livewire](https://laravel-livewire.com).

<img src="./img/breeze-register.png">

#### Laravel Bootcamp

Если вы новичок в Laravel, не стесняйтесь перейти в [Laravel Bootcamp] (https://bootcamp.laravel.com). Laravel Bootcamp проведет вас через создание вашего первого приложения Laravel с использованием Breeze. Это отличный способ познакомиться со всем, что могут предложить Laravel и Breeze.

<a name="laravel-breeze-installation"></a>
### Установка

Сначала вы должны [создать новое приложение Laravel](installation.md), настроить свою базу данных и запустить [миграции базы данных](migrations.md). После того, как вы создали новое приложение Laravel, вы можете установить Laravel Breeze с помощью Composer:

```shell
composer require laravel/breeze --dev
```

После установки Breeze вы можете создать шаблон для своего приложения, используя один из «стеков» Breeze, описанных в документации ниже.

<a name="breeze-and-blade"></a>
### Breeze и Blade

После того, как Composer установит пакет Laravel Breeze, вы можете запустить команду `breeze:install` Artisan. Эта команда опубликует для вашего приложения шаблоны, маршруты, контроллеры и другие ресурсы аутентификации. Laravel Breeze опубликует весь свой код в вашем приложении, чтобы у вас был полный контроль, а также обзор всего функционала и его реализации. 

Стек Breeze по умолчанию — это стек Blade, который использует простые [шаблоны Blade](blade.md) для отрисовки внешнего интерфейса вашего приложения. Стек Blade можно установить, вызвав команду `breeze:install` без дополнительных аргументов. После установки каркаса Breeze вы также должны скомпилировать ресурсы внешнего интерфейсные вашего приложения:

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

Затем, вы можете перейти в своем веб-браузере по URL-адресам вашего приложения `/login` или `/register`. Все маршруты Breeze определены в файле `routes/auth.php`.

<a name="dark-mode"></a>
#### Темная тема

Если вы хотите, чтобы Breeze включил поддержку «темной темы» при создании внешнего интерфейса вашего приложения, просто укажите директиву `--dark` при выполнении команды `breeze:install`:

```shell
php artisan breeze:install --dark
```

> **Примечание**\
> Чтобы узнать больше о компиляции CSS и JavaScript вашего приложения, ознакомьтесь с [документацией Laravel Vite](vite.md#running-vite).

<a name="breeze-and-inertia"></a>
### Breeze и React / Vue

Laravel Breeze также предлагает каркасы React и Vue через реализацию внешнего интерфейса [Inertia](https://inertiajs.com). Inertia позволяет создавать современные одностраничные приложения React и Vue, используя классическую маршрутизацию и контроллеры на стороне сервера.

Inertia позволяет вам наслаждаться мощью внешнего интерфейса React и Vue в сочетании с невероятной производительностью Laravel и молниеносной компиляцией [Vite](https://vitejs.dev). Чтобы использовать стек Inertia, укажите `vue` или `react` в качестве желаемого стека при выполнении команды `breeze:install` Artisan. После установки каркаса Breeze вы также должны скомпилировать ресурсы внешнего интерфейсные вашего приложения:

```shell
php artisan breeze:install vue

# Или ...

php artisan breeze:install react

php artisan migrate
npm install
npm run dev
```

Затем, вы можете перейти в своем веб-браузере по URL-адресам вашего приложения `/login` или `/register`. Все маршруты Breeze определены в файле `routes/auth.php`.

<a name="server-side-rendering"></a>
#### Серверный рендеринг

Если вы хотите, чтобы Breeze поддерживал [Inertia SSR](https://inertiajs.com/server-side-rendering), то вы можете указать параметр `ssr` при вызове команды `breeze:install`:

```shell
php artisan breeze:install vue --ssr
php artisan breeze:install react --ssr
```

<a name="breeze-and-next"></a>
### Breeze и Next.js / API

Laravel Breeze также может создать API-интерфейс аутентификации, готовый для аутентификации современных приложений JavaScript, например, на базе [Next](https://nextjs.org), [Nuxt](https://nuxtjs.org) и других. Для начала укажите стек `api` в качестве желаемого при выполнении команды `breeze:install` Artisan:

```shell
php artisan breeze:install api

php artisan migrate
```

Во время установки Breeze добавит переменную среды `FRONTEND_URL` в файл `.env` вашего приложения. Этот URL-адрес должен быть URL-адресом вашего приложения JavaScript. Обычно во время локальной разработки этим адресом будет `http://localhost:3000`. Кроме того, вы должны убедиться, что для вашего `APP_URL` установлено значение `http://localhost:8000`, которое является URL-адресом по умолчанию, используемым командой `serve` Artisan.

<a name="next-reference-implementation"></a>
#### Доступная реализация Next.js

Теперь вы будете готовы совместить указанный выше бэкэнд с выбранным вами интерфейсом. Реализация Next внешнего интерфейса Breeze доступна на [GitHub](https://github.com/laravel/breeze-next). Этот интерфейс поддерживается Laravel и содержит тот же пользовательский интерфейс, что и традиционные стеки Blade и Inertia, предоставляемые Breeze.

<a name="laravel-jetstream"></a>
## Laravel Jetstream

В то время как Laravel Breeze обеспечивает простую и минимальную отправную точку для создания приложения Laravel, Jetstream дополняет эту функциональность более надежными функциями и дополнительными стеками технологий клиентского интерфейса. **Для тех, кто новичок в Laravel, мы рекомендуем изучить основы работы с Laravel Breeze перед тем, как перейти на Laravel Jetstream.**

Jetstream предлагает красиво оформленный каркас приложений для Laravel и включает в себя вход в систему, регистрацию, подтверждение адреса электронной почты, двухфакторную аутентификацию, управление сессиями, поддержку API через Laravel Sanctum, и дополнительно, управление командой. Jetstream разработан с использованием [Tailwind CSS](https://tailwindcss.com) и предлагает на ваш выбор каркас клиентского интерфейса под управлением [Livewire](https://laravel-livewire.com) либо [Inertia](https://inertiajs.com).

Полное описание по установке Laravel Jetstream можно найти в [официальной документации Jetstream](https://jetstream.laravel.com).
