# Laravel 9 · Посредники

- [Введение](#introduction)
- [Определение посредника](#defining-middleware)
- [Регистрация посредника](#registering-middleware)
    - [Глобальный стек HTTP-посредников](#global-middleware)
    - [Назначение посредников маршрутам](#assigning-middleware-to-routes)
    - [Группы посредников](#middleware-groups)
    - [Сортировка посредников](#sorting-middleware)
- [Параметры посредника](#middleware-parameters)
- [Завершающий посредник](#terminable-middleware)

<a name="introduction"></a>
## Введение

Посредник обеспечивает удобный механизм для проверки и фильтрации HTTP-запросов, поступающих в ваше приложение. Например, в Laravel уже содержится посредник, проверяющий аутентификацию пользователя вашего приложения. Если пользователь не аутентифицирован, то посредник перенаправит пользователя на экран входа в ваше приложение. Однако, если пользователь аутентифицирован, то посредник позволит запросу продолжить работу в приложении.

Посредник может быть написан для выполнения различных задач помимо аутентификации. Например, посредник для ведения журнала может регистрировать все входящие запросы вашего приложения. В состав фреймворка Laravel уже входят несколько посредников, включая посредник для аутентификации и посредник для защиты от CSRF. Все эти посредники находится в каталоге `app/Http/Middleware`.

<a name="defining-middleware"></a>
## Определение посредника

Чтобы создать нового посредника, используйте команду `make:middleware` [Artisan](artisan.md):

```shell
php artisan make:middleware EnsureTokenIsValid
```

Эта команда поместит новый класс посредника в каталог `app/Http/Middleware` вашего приложения. В этом посреднике мы будем разрешать доступ к маршруту только в том случае, если значение входящего `token` соответствует указанному. В противном случае мы перенаправим пользователя по маршруту `home`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class EnsureTokenIsValid
    {
        /**
         * Обработка входящего запроса.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->input('token') !== 'my-secret-token') {
                return redirect('home');
            }

            return $next($request);
        }
    }

Как видите, если переданный `token` не совпадает с нашим секретным токеном, то посредник вернет клиенту HTTP-перенаправление; в противном случае запрос будет передан в приложение. Чтобы передать запрос дальше в приложение (позволяя «пройти» посредника), вы должны вызвать замыкание `$next` с параметром `$request`.

Лучше всего представить себе посредников как серию «слоев» для HTTP-запроса, которые необходимо пройти, прежде чем запрос попадет в ваше приложение. Каждый слой может рассмотреть запрос и даже полностью отклонить его.

> {tip} Все посредники извлекаются из [контейнера служб](container.md), поэтому вы можете объявить необходимые вам зависимости в конструкторе посредника.

<a name="before-after-middleware"></a>
<a name="middleware-and-responses"></a>
#### Посредники и ответы

Конечно, посредник может выполнять задачи до или после передачи запроса в приложение. Например, следующий посредник будет выполнять некоторую задачу **до** того, как запрос будет обработан приложением:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Выполнить действие

            return $next($request);
        }
    }

Однако, этот посредник будет выполнять свою задачу **после** обработки входящего запроса приложением:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Выполнить действие

            return $response;
        }
    }

<a name="registering-middleware"></a>
## Регистрация посредника

<a name="global-middleware"></a>
### Глобальный стек HTTP-посредников

Если вы хотите, чтобы посредник запускался во время каждого HTTP-запроса к вашему приложению, то укажите класс посредника в свойстве `$middleware` вашего класса `App\Http\Kernel`.

<a name="assigning-middleware-to-routes"></a>
### Назначение посредников маршрутам

Если вы хотите назначить посредника определенным маршрутам, то вам следует сначала зарегистрировать ключ посредника в файле `app/Http/Kernel.php` вашего приложения. По умолчанию свойство `$routeMiddleware` этого класса содержит записи для посредников, уже включенных в состав Laravel. Вы можете добавить свой собственный посредник в этот список, и назначить ему ключ по вашему выбору:

    // Внутри класса App\Http\Kernel ...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];

После того, как посредник был определен в HTTP-ядре, вы можете использовать метод `middleware` для назначения посредника маршруту:

    Route::get('/profile', function () {
        //
    })->middleware('auth');

Вы можете назначить несколько посредников маршруту, передав массив имен посредников методу `middleware`:

    Route::get('/', function () {
        //
    })->middleware(['first', 'second']);

Вы можете назначить посредника, передав полное имя класса:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::get('/profile', function () {
        //
    })->middleware(EnsureTokenIsValid::class);

<a name="excluding-middleware"></a>
#### Исключение посредника

При назначении посредника группе маршрутов, иногда требуется запретить применение посредника к одному из маршрутов в группе. Вы можете сделать это с помощью метода `withoutMiddleware`:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::middleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/', function () {
            //
        });

        Route::get('/profile', function () {
            //
        })->withoutMiddleware([EnsureTokenIsValid::class]);
    });

Вы также можете исключить переданный набор посредников из определения [группы](routing.md#route-groups) маршрутов:

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/profile', function () {
            //
        });
    });

Метод `withoutMiddleware` удаляет только посредника маршрутизации и не применим к [глобальному посреднику](#global-middleware).

<a name="middleware-groups"></a>
### Группы посредников

По желанию можно сгруппировать несколько посредников под одним ключом, чтобы упростить их назначение маршрутам. Вы можете сделать это, используя свойство `$middlewareGroups` вашего HTTP-ядра.

По умолчанию Laravel поставляется с группами посредников `web` и `api`, которые содержат основных посредников, которые вы, возможно, захотите применить к своим веб- и  API-маршрутам. Помните, что эти группы посредников автоматически применяются поставщиком служб `App\Providers\RouteServiceProvider` вашего приложения к маршрутам, определенным в файлах маршрутов `web` и `api`, соответственно:

    /**
     * Группы посредников маршрутов приложения.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];

Группы посредников могут быть назначены маршрутам и действиям контроллера с использованием того же синтаксиса, что и для отдельных посредников. Опять же, группы посредников делают более удобным одновременное назначение нескольких посредников для маршрута:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::middleware(['web'])->group(function () {
        //
    });

> {tip} Из коробки группы посредников `web` и `api` автоматически применяются к соответствующим файлам вашего приложения `routes/web.php` и `routes/api.php` с помощью `App\Providers\RouteServiceProvider`.

<a name="sorting-middleware"></a>
### Сортировка посредников

В редких случаях, может понадобиться, чтобы посредники выполнялись в определенном порядке, но вы не можете контролировать их порядок, когда они назначены маршруту. В этом случае вы можете указать приоритет посредников, используя свойство `$middlewarePriority` вашего класса `App\Http\Kernel`. Это свойство может отсутствовать в вашем HTTP-ядре по умолчанию. Если оно не существует, то вы можете скопировать его определение по умолчанию:

    /**
     * Список посредников, отсортированный по приоритетности.
     *
     * Заставит неглобальных посредников всегда быть в заданном порядке.
     *
     * @var string[]
     */
    protected $middlewarePriority = [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Contracts\Session\Middleware\AuthenticatesSessions::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];

<a name="middleware-parameters"></a>
## Параметры посредника

Посредник может получать дополнительные параметры. Например, если вашему приложению необходимо проверить, что аутентифицированный пользователь имеет конкретную «роль» перед выполнением им конкретного действия, то вы можете создать посредника, например, `EnsureUserHasRole`, который получит имя роли в качестве дополнительного аргумента.

Дополнительные параметры посредника будут переданы после аргумента `$next`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class EnsureUserHasRole
    {
        /**
         * Обработка входящего запроса.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Перенаправление ...
            }

            return $next($request);
        }

    }

Параметры посредника можно указать при определении маршрута, разделив имя посредника и параметры символом `:`. Несколько параметров следует разделять запятыми:

    Route::put('/post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## Завершающий посредник

Иногда посреднику требуется выполнить некоторую работу после отправки HTTP-ответа в браузер. Если вы определите метод `terminate` в своем посреднике и при условии, что ваш веб-сервер использует FastCGI, то метод `terminate` будет автоматически вызван после отправки ответа в браузер:

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class TerminatingMiddleware
    {
        /**
         * Обработка входящего запроса.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        /**
         * Обработать задачи после отправки ответа в браузер.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Illuminate\Http\Response  $response
         * @return void
         */
        public function terminate($request, $response)
        {
            // ...
        }
    }

Метод `terminate` должен получать и запрос, и ответ. После того, как вы определили завершающий посредник, вы должны добавить его в список маршрутов или глобальный стек посредников в файле `app/Http/Kernel.php`.

При вызове метода `terminate` посредника, Laravel извлечет новый экземпляр посредника из [контейнера служб](container.md). Если вы хотите использовать один и тот же экземпляр посредника при вызове методов `handle` и `terminate`, то зарегистрируйте посредника в контейнере, используя метод контейнера `singleton`. Обычно это должно быть сделано в методе `register` вашего `AppServiceProvider`:

    use App\Http\Middleware\TerminatingMiddleware;

    /**
     * Регистрация любых служб приложения.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(TerminatingMiddleware::class);
    }
