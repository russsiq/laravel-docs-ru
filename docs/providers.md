# Laravel 9 · Поставщики служб

- [Введение](#introduction)
- [Написание поставщиков служб](#writing-service-providers)
    - [Метод `register`](#the-register-method)
    - [Метод `boot`](#the-boot-method)
- [Регистрация поставщиков](#registering-providers)
- [Отложенные поставщики](#deferred-providers)

<a name="introduction"></a>
## Введение

Поставщики служб – это центральное место начальной загрузки всех приложений Laravel. Ваше собственное приложение, а также все основные службы Laravel загружаются через поставщиков.

Но, что мы подразумеваем под «начальной загрузкой»? В общем, мы имеем в виду **регистрацию** элементов, включая регистрацию связываний контейнера служб, слушателей событий, посредников и даже маршрутов. Поставщики служб являются центральным местом для конфигурирования приложения.

Если вы откроете файл `config/app.php`, включенный в Laravel, вы увидите массив поставщиков. Это все классы поставщиков служб, которые будут загружены вашим приложением. По умолчанию в этом массиве перечислены основные поставщики служб Laravel. Эти поставщики загружают основные компоненты Laravel, такие как почтовая программа, очередь, кеш и другие. Многие из этих поставщиков являются «отложенными», что означает, что они не будут загружаться при каждом запросе, а только тогда, когда предоставляемые ими службы действительно необходимы.

В этой документации вы узнаете, как писать собственных поставщиков служб и регистрировать их в приложении Laravel.

> {tip} Если вы хотите узнать больше о том, как Laravel обрабатывает запросы и работает изнутри, ознакомьтесь с нашей документацией по [жизненному циклу запроса](lifecycle.md) Laravel.

<a name="writing-service-providers"></a>
## Написание поставщиков служб

Все поставщики служб расширяют класс `Illuminate\Support\ServiceProvider`. Большинство поставщиков служб содержат метод `register` и `boot`. В рамках метода `register` следует **только связать сущности в [контейнере служб](container.md)**. Никогда не следует пытаться зарегистрировать каких-либо слушателей событий, маршруты или любые другие функциональные возможности в методе `register`.

Чтобы сгенерировать нового поставщика, используйте команду `make:provider` [Artisan](artisan.md):

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### Метод `register`

Как упоминалось ранее, в рамках метода `register` следует только связать сущности в [контейнере служб](container.md). Никогда не следует пытаться зарегистрировать слушателей событий, маршруты или любую другую функциональность в методе `register`. В противном случае вы можете случайно воспользоваться службой, которая еще не загружена.

Давайте взглянем на основного поставщика служб. В любом из методов поставщика служб у вас всегда есть доступ к свойству `$app`, которое обеспечивает доступ к контейнеру служб:

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Регистрация любых служб приложения.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

Этот поставщик службы определяет только метод `register` и использует этот метод для определения реализации `App\Services\Riak\Connection` в контейнере служб. Если вы еще не знакомы с контейнером служб Laravel, ознакомьтесь с [его документацией](container.md).

<a name="the-bindings-and-singletons-properties"></a>
#### Свойства `bindings` и `singletons`

Если ваш поставщик службы регистрирует много простых связываний, вы можете использовать свойства `bindings` и `singletons` вместо ручной регистрации каждого связывания контейнера. Когда поставщик службы загружается фреймворком, он автоматически проверяет эти свойства и регистрирует их связывания:

    <?php

    namespace App\Providers;

    use App\Contracts\DowntimeNotifier;
    use App\Contracts\ServerProvider;
    use App\Services\DigitalOceanServerProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\ServerToolsProvider;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Все связывания контейнера, которые должны быть зарегистрированы.
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * Все синглтоны (одиночки) контейнера, которые должны быть зарегистрированы.
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier::class,
            ServerProvider::class => ServerToolsProvider::class,
        ];
    }

<a name="the-boot-method"></a>
### Метод `boot`

Итак, что, если нам нужно зарегистрировать [компоновщик шаблонов](views.md#view-composers) в нашем поставщике службы? Это должно быть сделано в рамках метода `boot`. **Этот метод вызывается после регистрации всех других поставщиков служб**, что означает, что у вас есть доступ ко всем другим службам, которые были зарегистрированы фреймворком:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Загрузка любых служб приложения.
         *
         * @return void
         */
        public function boot()
        {
            View::composer('view', function () {
                //
            });
        }
    }

<a name="boot-method-dependency-injection"></a>
#### Внедрение зависимости в методе `boot`

Вы можете указывать тип зависимостей в методе `boot` поставщика службы. [Контейнер служб](container.md) автоматически внедрит любые необходимые зависимости:

    use Illuminate\Contracts\Routing\ResponseFactory;

    /**
     * Загрузка любых служб приложения.
     *
     * @param  \Illuminate\Contracts\Routing\ResponseFactory  $response
     * @return void
     */
    public function boot(ResponseFactory $response)
    {
        $response->macro('serialized', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Регистрация поставщиков

Все поставщики служб регистрируются в файле конфигурации `config/app.php`. Этот файл содержит массив `providers`, в котором можно перечислить имена классов ваших поставщиков служб. По умолчанию в этом массиве перечислены основные поставщики служб Laravel. Эти поставщики загружают основные компоненты Laravel, такие как почтовая программа, очереди, кеш и другие.

Чтобы зарегистрировать поставщика, добавьте его в массив:

    'providers' => [
        // Другие поставщики служб

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## Отложенные поставщики

Если ваш поставщик регистрирует **только** связывания в [контейнере служб](container.md), вы можете отложить его регистрацию до тех пор, пока одно из зарегистрированных связываний не понадобится. Отсрочка загрузки такого поставщика повысит производительность вашего приложения, так как он не загружается из файловой системы при каждом запросе.

Laravel составляет и сохраняет список всех служб, предоставляемых отложенными поставщиками служб, а также имя класса поставщика службы. Laravel загрузит поставщика службы только при необходимости в одной из этих служб.

Чтобы отложить загрузку поставщика, реализуйте интерфейс `\Illuminate\Contracts\Support\DeferrableProvider`, описав метод `provides`. Метод `provides` должен вернуть связывания контейнера службы, регистрируемые поставщиком:

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Contracts\Support\DeferrableProvider;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
    {
        /**
         * Регистрация любых служб приложения.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Получить службы, предоставляемые поставщиком.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }
    }
