# Laravel 8 · Руководство по обновлению

- [Обновление с 7.x версии до 8.0](#upgrade-8.0)

<a name="high-impact-changes"></a>
## Изменения, оказывающие большое влияние

<!-- <div class="content-list" markdown="1"> -->
- [Фабрики модели](#model-factories)
- [Метод очереди `retryAfter`](#queue-retry-after-method)
- [Свойство очереди `timeoutAt`](#queue-timeout-at-property)
- [Методы очереди `allOnQueue` и `allOnConnection`](#queue-allOnQueue-allOnConnection)
- [Пагинация по умолчанию](#pagination-defaults)
- [Пространства имен наполнителей и фабрик](#seeder-factory-namespaces)
<!-- </div> -->

<a name="medium-impact-changes"></a>
## Изменения со средней степенью воздействия

<!-- <div class="content-list" markdown="1"> -->
- [Требование PHP 7.3.0](#php-7.3.0-required)
- [Поддержка пакетной обработки и таблица невыполненных заданий](#failed-jobs-table-batch-support)
- [Обновления режима обслуживания](#maintenance-mode-updates)
- [Параметр `php artisan down --message`](#artisan-down-message)
- [Метод `assertExactJson`](#assert-exact-json-method)
<!-- </div> -->

<a name="upgrade-8.0"></a>
## Обновление с 7.x версии до 8.0

<a name="estimated-upgrade-time-15-minutes"></a>
#### Приблизительное время обновления: 15 минут

> {note} Мы стараемся задокументировать все возможные критические изменения. Поскольку некоторые из этих критических изменений находятся в малоизвестных частях фреймворка, только часть этих изменений может повлиять на ваше приложение.

<a name="php-7.3.0-required"></a>
### Требование PHP 7.3.0

**Вероятность воздействия: средняя**

Новая минимальная версия PHP теперь 7.3.0.

<a name="updating-dependencies"></a>
### Обновление зависимостей

Обновите следующие зависимости в вашем файле `composer.json`:

<!-- <div class="content-list" markdown="1"> -->
- `guzzlehttp/guzzle` до `^7.0.1`
- `facade/ignition` до `^2.3.6`
- `laravel/framework` до `^8.0`
- `laravel/ui` до `^3.0`
- `nunomaduro/collision` до `^5.0`
- `phpunit/phpunit` до `^9.0`
<!-- </div> -->

Следующие сторонние пакеты имеют новые основные выпуски для поддержки Laravel 8. Если возможно, вы должны прочитать соответствующие руководства перед обновлением:

<!-- <div class="content-list" markdown="1"> -->
- [Horizon v5.0](https://github.com/laravel/horizon/blob/master/UPGRADE.md)
- [Passport v10.0](https://github.com/laravel/passport/blob/master/UPGRADE.md)
- [Socialite v5.0](https://github.com/laravel/socialite/blob/master/UPGRADE.md)
- [Telescope v4.0](https://github.com/laravel/telescope/blob/master/UPGRADE.md)
<!-- </div> -->

Кроме того, установщик Laravel был обновлен для поддержки `composer create-project` и Laravel Jetstream. Любой установщик старше 4.0 перестанет работать после октября 2020 года. Вам следует как можно скорее обновить глобальный установщик до `^4.0`.

Наконец, изучите любые другие сторонние пакеты, используемые вашим приложением, и убедитесь, что вы используете корректную версию с поддержкой Laravel 8.

<a name="collections"></a>
### Коллекции

<a name="the-isset-method"></a>
#### Метод `isset`

**Вероятность воздействия: низкая**

Чтобы соответствовать типичному поведению PHP, метод `offsetExists` в `Illuminate\Support\Collection` был обновлен и теперь использует `isset` вместо `array_key_exists`. Это может привести к изменению поведения при работе с элементами коллекции, имеющими значение `null`:

    $collection = collect([null]);

    // Laravel 7.x - true
    isset($collection[0]);

    // Laravel 8.x - false
    isset($collection[0]);

<a name="database"></a>
### База данных

<a name="seeder-factory-namespaces"></a>
#### Пространства имен наполнителей и фабрик

**Вероятность воздействия: высокая**

Наполнители и фабрики теперь имеют пространство имен. Чтобы учесть эти изменения, добавьте пространство имен `Database\Seeders` в ваши классы наполнителей. Кроме того, имеющийся каталог `database/seeds` должен быть переименован в `database/seeders`:

    <?php

    namespace Database\Seeders;

    use App\Models\User;
    use Illuminate\Database\Seeder;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Заполнить базу данных приложения.
         *
         * @return void
         */
        public function run()
        {
            ...
        }
    }

Если вы решите использовать пакет `laravel/legacy-factories`, то никаких изменений в классах ваших фабрик не требуется. Однако, если вы обновляете свои фабрики, вы должны добавить к этим классам пространство имен `Database\Factories`.

Затем в вашем файле `composer.json` удалите блок `classmap` из раздела `autoload` и добавьте новые сопоставления каталогов классов с пространством имен:

    "autoload": {
        "psr-4": {
            "App\\": "app/",
            "Database\\Factories\\": "database/factories/",
            "Database\\Seeders\\": "database/seeders/"
        }
    },

<a name="eloquent"></a>
### Eloquent

<a name="model-factories"></a>
#### Фабрики модели

**Вероятность воздействия: высокая**

Функция Laravel [фабрики модели](/docs/database-testing.md#defining-model-factories) была полностью переписана для поддержки классов и несовместима с фабриками стиля Laravel 7.x. Однако, чтобы упростить процесс обновления, был создан новый пакет `laravel/legacy-factories`, чтобы продолжать использовать ваши существующие фабрики с Laravel 8.x. Вы можете установить этот пакет через Composer:

    composer require laravel/legacy-factories

<a name="the-castable-interface"></a>
#### Интерфейс `Castable`

**Вероятность воздействия: низкая**

Метод `castUsing` интерфейса `Castable` обновлен и теперь принимает массив аргументов. Если вы реализуете этот интерфейс, вам, соответственно, следует обновить реализацию:

    public static function castUsing(array $arguments);

<a name="increment-decrement-events"></a>
#### События Increment / Decrement

**Вероятность воздействия: низкая**

События модели, связанные с «обновлением» и «сохранением», теперь будут вызываться при выполнении методов `increment` или `decrement` экземпляров модели Eloquent.

<a name="events"></a>
### События

<a name="the-event-service-provider-class"></a>
#### Класс `EventServiceProvider`

**Вероятность воздействия: низкая**

Если ваш класс `App\Providers\EventServiceProvider` содержит метод `register`, то вы должны убедиться, что вы вызываете `parent::register` в начале этого метода. В противном случае события вашего приложения не будут зарегистрированы.

<a name="the-dispatcher-contract"></a>
#### Контракт `Dispatcher`

**Вероятность воздействия: низкая**

Метод `listen` контракта `Illuminate\Contracts\Events\Dispatcher` был обновлен, чтобы сделать свойство `$listener` необязательным. Это изменение было внесено для поддержки автоматического определения обрабатываемых типов событий через рефлексию. Если вы реализуете этот интерфейс, вам, соответственно, следует обновить реализацию:

    public function listen($events, $listener = null);

<a name="framework"></a>
### Фреймворк

<a name="maintenance-mode-updates"></a>
#### Обновления режима обслуживания

**Вероятность воздействия: необязательно**

[Режим обслуживания](/docs/configuration.md#maintenance-mode) был улучшен в Laravel 8.x. Теперь поддерживается предварительный рендеринг шаблона режима обслуживания, что исключает вероятность того, что конечные пользователи столкнутся с ошибками в режиме обслуживания. Однако для поддержки этого в ваш файл `public/index.php` необходимо добавить следующие строки. Эти строки следует разместить непосредственно под существующим определением константы `LARAVEL_START`:

    define('LARAVEL_START', microtime(true));

    if (file_exists(__DIR__.'/../storage/framework/maintenance.php')) {
        require __DIR__.'/../storage/framework/maintenance.php';
    }

<a name="artisan-down-message"></a>
#### Параметр `php artisan down --message`

**Вероятность воздействия: средняя**

Параметр `--message` команды `php artisan down` была удалена. В качестве альтернативы рассмотрите возможность [предварительного рендеринга шаблонов в режиме обслуживания](/docs/configuration.md#maintenance-mode) с желаемым сообщением.

<a name="php-artisan-serve-no-reload-option"></a>
#### Параметр `php artisan serve --no-reload`

**Вероятность воздействия: низкая**

В команде `php artisan serve` добавлен параметр `--no-reload`. Это даст указание встроенному серверу не перезагружаться при обнаружении изменений файла окружения. Эта опция в первую очередь полезна при запуске тестов Laravel Dusk в среде CI (непрерывной интеграции).

<a name="manager-app-property"></a>
#### Свойство `$app` Менеджера

**Вероятность воздействия: низкая**

Ранее устаревшее свойство `$app` класса `Illuminate\Support\Manager` было удалено. Если вы полагались на это свойство, вам следует использовать вместо него свойство `$container`.

<a name="the-elixir-helper"></a>
#### Помощник `elixir`

**Вероятность воздействия: низкая**

Ранее устаревший помощник `elixir` был удален. Приложениям, все еще использующим данный метод сборки, рекомендуется перейти на [Laravel Mix](https://github.com/JeffreyWay/laravel-mix).

<a name="mail"></a>
### Почта

<a name="the-sendnow-method"></a>
#### Метод `sendNow`

**Вероятность воздействия: низкая**

Ранее устаревший метод `sendNow` был удален. Вместо этого используйте метод `send`.

<a name="pagination"></a>
### Постраничная навигация

<a name="pagination-defaults"></a>
#### Пагинация по умолчанию

**Вероятность воздействия: высокая**

Пагинатор теперь использует [CSS-фреймворк Tailwind](https://tailwindcss.com) для стилизации по умолчанию. Чтобы продолжить использование Bootstrap, вы должны добавить следующий вызов метода в методе `boot` поставщика служб приложения `AppServiceProvider`:

    use Illuminate\Pagination\Paginator;

    Paginator::useBootstrap();

<a name="queue"></a>
### Очереди

<a name="queue-retry-after-method"></a>
#### Метод `retryAfter`

**Вероятность воздействия: высокая**

Для согласованности с другой функциональностью Laravel, метод `retryAfter` и свойство `retryAfter` заданий в очереди, почтовых программ, уведомлений и слушателей были переименованы в `backoff`. Вам следует обновить имя этого метода / свойства в соответствующих классах вашего приложения.

<a name="queue-timeout-at-property"></a>
#### Свойство `timeoutAt`

**Вероятность воздействия: высокая**

Свойство `timeoutAt` заданий в очереди, уведомлений и слушателей переименовано в `retryUntil`. Вам следует обновить имя этого свойства в соответствующих классах вашего приложения.

<a name="queue-allOnQueue-allOnConnection"></a>
#### Методы `allOnQueue()` / `allOnConnection()`

**Вероятность воздействия: высокая**

Для согласованности с другими методами диспетчеризации были удалены методы `allOnQueue()` и `allOnConnection()`, используемые с цепочкой заданий. Вместо этого вы можете использовать методы `onQueue()` и `onConnection()`. Эти методы следует вызывать перед вызовом метода `dispatch`:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->onConnection('redis')->onQueue('podcasts')->dispatch();

Обратите внимание, что это изменение влияет только на код, использующий метод `withChain`. В то время как методы `allOnQueue()` и `allOnConnection()` по-прежнему доступны при использовании глобального помощника `dispatch()`.

<a name="failed-jobs-table-batch-support"></a>
#### Поддержка пакетной обработки и таблица невыполненных заданий

**Вероятность воздействия: необязательно**

Если вы планируете использовать функционал [пакетной обработки заданий](/docs/queues.md#job-batching) Laravel 8.x, то таблица `failed_jobs` БД должна быть обновлена. Во-первых, в эту таблицу должен быть добавлен новый столбец `uuid`:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('failed_jobs', function (Blueprint $table) {
        $table->string('uuid')->after('id')->nullable()->unique();
    });

Затем, параметр конфигурации `failed.driver` в конфигурационном файле `config/queue.php` должен быть изменен на `database-uuids`.

Кроме того, вы можете сгенерировать UUID для существующих невыполненных заданий:

    DB::table('failed_jobs')->whereNull('uuid')->cursor()->each(function ($job) {
        DB::table('failed_jobs')
            ->where('id', $job->id)
            ->update(['uuid' => (string) Illuminate\Support\Str::uuid()]);
    });

<a name="routing"></a>
### Маршрутизация

<a name="automatic-controller-namespace-prefixing"></a>
#### Автоматическое префикс пространства имен контроллера

**Вероятность воздействия: необязательно**

В предыдущих выпусках Laravel класс `RouteServiceProvider` содержал свойство `$namespace` со значением `App\Http\Controllers`. Значение этого свойства использовалось для объявлений автоматического префикса маршрута контроллера и генерации URL маршрута контроллера, например, при вызове помощника `action`.

В Laravel 8 для этого свойства по умолчанию установлено значение `null`. Это позволяет объявлениям маршрута вашего контроллера использовать стандартный вызываемый синтаксис PHP, который обеспечивает лучшую поддержку перехода к классу контроллера во многих IDE:

    use App\Http\Controllers\UserController;

    // Использование вызываемого синтаксиса PHP ...
    Route::get('/users', [UserController::class, 'index']);

    // Использование строкового синтаксиса ...
    Route::get('/users', 'App\Http\Controllers\UserController@index');

В большинстве случаев это не повлияет на обновляемые приложения, потому что ваш `RouteServiceProvider` по-прежнему будет содержать свойство `$namespace` с его предыдущим значением. Однако, если вы обновите свое приложение, создав новый проект Laravel, то это изменение может стать критическим.

Если вы хотите по-прежнему использовать исходную маршрутизацию контроллера с автоматическим префиксом, вы можете просто установить значение свойства `$namespace` в` RouteServiceProvider` и обновить регистрации маршрута в методе `boot`, чтобы использовать свойство `$namespace`:

    class RouteServiceProvider extends ServiceProvider
    {
        /**
         * Путь к «домашнему» маршруту вашего приложения.
         *
         * Используется аутентификацией Laravel для перенаправления пользователей после входа в систему.
         *
         * @var string
         */
        public const HOME = '/home';

        /**
         * Если указано, это пространство имен автоматически применяется к маршрутам вашего контроллера.
         *
         * Кроме того, оно устанавливается как корневое пространство имен генератора URL.
         *
         * @var string
         */
        protected $namespace = 'App\Http\Controllers';

        /**
         * Определить связывание модели и маршрута, фильтры шаблонов и т.д.
         *
         * @return void
         */
        public function boot()
        {
            $this->configureRateLimiting();

            $this->routes(function () {
                Route::middleware('web')
                    ->namespace($this->namespace)
                    ->group(base_path('routes/web.php'));

                Route::prefix('api')
                    ->middleware('api')
                    ->namespace($this->namespace)
                    ->group(base_path('routes/api.php'));
            });
        }

        /**
         * Настроить ограничения запросов для приложения.
         *
         * @return void
         */
        protected function configureRateLimiting()
        {
            RateLimiter::for('api', function (Request $request) {
                return Limit::perMinute(60)->by(optional($request->user())->id ?: $request->ip());
            });
        }
    }

<a name="scheduling"></a>
### Планирование задач

<a name="the-cron-expression-library"></a>
#### Библиотека `cron-expression`

**Вероятность воздействия: низкая**

Зависимость Laravel от `dragonmantank/cron-expression` была обновлена с `2.x` до `3.x`. Это не должно вызывать каких-либо критических изменений в вашем приложении, если вы не взаимодействуете напрямую с библиотекой `cron-expression`. Если вы напрямую взаимодействуете с этой библиотекой, просмотрите ее [журнал изменений](https://github.com/dragonmantank/cron-expression/blob/master/CHANGELOG.md).

<a name="session"></a>
### Сессия

<a name="the-session-contract"></a>
#### Контракт `Session`

**Вероятность воздействия: низкая**

Контракт `Illuminate\Contracts\Session\Session` получил новый метод `pull`. Если вы реализуете этот контракт самостоятельно, то вам следует соответствующим образом обновить его реализацию:

    /**
     * Получите значение переданного ключа и удалить его.
     *
     * @param  string  $key
     * @param  mixed  $default
     * @return mixed
     */
    public function pull($key, $default = null);

<a name="testing"></a>
### Тестирование

<a name="decode-response-json-method"></a>
#### Метод `decodeResponseJson`

**Вероятность воздействия: низкая**

Метод `decodeResponseJson`, принадлежащий классу `Illuminate\Testing\TestResponse`, больше не принимает никаких аргументов. Пожалуйста, подумайте об использовании вместо этого метода `json`.

<a name="assert-exact-json-method"></a>
#### Метод `assertExactJson`

**Вероятность воздействия: средняя**

Метод `assertExactJson` теперь требует, чтобы числовые ключи сравниваемых массивов совпадали и располагались в том же порядке. Если вы хотите сравнить JSON с массивом, не требуя, чтобы массивы с числовыми ключами имели одинаковый порядок, вы можете вместо этого использовать метод `assertSimilarJson`.

<a name="validation"></a>
### Валидация

<a name="database-rule-connections"></a>
### Соединения для правил, использующих БД

**Вероятность воздействия: низкая**

Правила `unique` и `exists` теперь будут учитывать указанное имя соединения моделей Eloquent при выполнении запросов. Это имя соединения доступно через метод `getConnectionName` модели.

<a name="miscellaneous"></a>
### Разное

Мы также рекомендуем вам просматривать изменения в GitHub-репозитории [`laravel/laravel`](https://github.com/laravel/laravel). Хотя многие из этих изменений могут быть неважны, но вы можете синхронизировать эти файлы с вашим приложением. Некоторые из этих изменений будут рассмотрены в данном руководстве по обновлению, но другие, такие как изменения файлов конфигурации или комментарии, не будут. Вы можете легко просмотреть изменения с помощью [инструмента сравнения GitHub](https://github.com/laravel/laravel/compare/7.x...8.x) и выбрать, какие обновления важны для вас.
