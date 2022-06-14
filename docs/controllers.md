# Laravel 9 · Контроллеры

- [Введение](#introduction)
- [Написание контроллеров](#writing-controllers)
    - [Основные понятия о контроллерах](#basic-controllers)
    - [Контроллеры одиночного действия](#single-action-controllers)
- [Посредник контроллера](#controller-middleware)
- [Ресурсные контроллеры](#resource-controllers)
    - [Частичные ресурсные маршруты](#restful-partial-resource-routes)
    - [Вложенные ресурсы](#restful-nested-resources)
    - [Именование ресурсных маршрутов](#restful-naming-resource-routes)
    - [Именование параметров ресурсных маршрутов](#restful-naming-resource-route-parameters)
    - [Ограничение ресурсных маршрутов](#restful-scoping-resource-routes)
    - [Локализация URI ресурсов](#restful-localizing-resource-uris)
    - [Дополнение ресурсных контроллеров](#restful-supplementing-resource-controllers)
- [Внедрение зависимостей и контроллеры](#dependency-injection-and-controllers)

<a name="introduction"></a>
## Введение

Вместо того, чтобы определять всю логику обработки запросов как замыкания в файлах маршрутов, вы можете организовать это поведение с помощью классов «контроллеров». Контроллеры могут сгруппировать связанную логику обработки запросов в один класс. Например, класс `UserController` может обрабатывать все входящие запросы, относящиеся к пользователям, включая отображение, создание, обновление и удаление пользователей. По умолчанию контроллеры хранятся в каталоге `app/Http/Controllers`.

<a name="writing-controllers"></a>
## Написание контроллеров

<a name="basic-controllers"></a>
### Основные понятия о контроллерах

Давайте посмотрим на отвлеченный пример контроллера. Обратите внимание, что он расширяет базовый класс контроллера `App\Http\Controllers\Controller`, включенный в Laravel:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;

    class UserController extends Controller
    {
        /**
         * Показать профиль конкретного пользователя.
         *
         * @param  int  $id
         * @return \Illuminate\View\View
         */
        public function show($id)
        {
            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

Вы можете определить маршрут к этому методу контроллера следующим образом:

    use App\Http\Controllers\UserController;

    Route::get('/user/{id}', [UserController::class, 'show']);

Когда входящий запрос совпадает с указанным URI маршрута, будет вызван метод `show` класса `App\Http\Controllers\UserController`, и параметры маршрута будут переданы методу.

> {tip} Контроллеры **не требуют** расширения базового класса. Однако у вас не будет доступа к удобным функциям, таким как методы `middleware` и `authorize`.

<a name="single-action-controllers"></a>
### Контроллеры одиночного действия

Если действие контроллера является особенно сложным, вам может показаться удобным посвятить целый класс контроллера этому единственному действию. Для этого вы можете определить один метод `__invoke` в контроллере:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;

    class ProvisionServer extends Controller
    {
        /**
         * Подготовить новый веб-сервер.
         *
         * @return \Illuminate\Http\Response
         */
        public function __invoke()
        {
            // ...
        }
    }

При регистрации маршрутов для контроллеров одиночного действия вам не нужно указывать метод контроллера. Вместо этого вы можете просто передать маршрутизатору имя контроллера:

    use App\Http\Controllers\ProvisionServer;

    Route::post('/server', ProvisionServer::class);

Вы можете сгенерировать вызываемый контроллер, используя параметр `--invokable` команды `make:controller` Artisan:

```shell
php artisan make:controller ProvisionServer --invokable
```

> {tip} Заготовки контроллера можно настроить с помощью [публикации заготовок](artisan.md#stub-customization).

<a name="controller-middleware"></a>
## Посредник контроллера

[Посредник](middleware.md) может быть назначен маршрутам контроллера в ваших файлах маршрутизации:

    Route::get('profile', [UserController::class, 'show'])->middleware('auth');

Или вам может быть удобно указать посредника в конструкторе вашего контроллера. Используя метод `middleware` в конструкторе вашего контроллера, вы можете назначить посредника действиям контроллера:

    class UserController extends Controller
    {
        /**
         * Создать новый экземпляр контроллера.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');
            $this->middleware('log')->only('index');
            $this->middleware('subscribed')->except('store');
        }
    }

Контроллеры также позволяют регистрировать посредника с помощью замыкания. Это обеспечивает удобный способ определения встроенного посредника для одного контроллера без определения целого класса посредника:

    $this->middleware(function ($request, $next) {
        return $next($request);
    });

<a name="resource-controllers"></a>
## Ресурсные контроллеры

Если вы думаете о каждой модели Eloquent в вашем приложении как о «ресурсе», то для каждого ресурса в вашем приложении обычно выполняются одни и те же наборы действий. Например, представьте, что ваше приложение содержит модель `Photo` и модель `Movie`. Вполне вероятно, что пользователи могут создавать, читать, обновлять или удалять эти ресурсы.

Благодаря такому распространенному варианту использования, маршрутизация ресурсов Laravel присвоит типичные маршруты создания, чтения, обновления и удаления («CRUD») контроллеру с помощью одной строки кода. Для начала мы можем использовать параметр `--resource` команды `make:controller` Artisan, чтобы быстро создать контроллер для обработки этих действий:

```shell
php artisan make:controller PhotoController --resource
```

Эта команда поместит новый класс контроллера в каталог `app/Http/Controllers` вашего приложения. Контроллер будет содержать метод для каждого из доступных действий с ресурсами. Затем, вы можете зарегистрировать маршрут ресурса, который указывает на контроллер:

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class);

Это единое определение маршрута создаст несколько маршрутов для обработки множества действий с ресурсом. Сгенерированный контроллер уже будет иметь заготовки для каждого из этих действий. Помните, вы всегда можете получить быстрый обзор маршрутов своего приложения, выполнив команду `route:list` Artisan.

Вы даже можете зарегистрировать сразу несколько контроллеров ресурсов, передав массив методу `resources`:

    Route::resources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

<a name="actions-handled-by-resource-controller"></a>
#### Действия, выполняемые ресурсными контроллерами

Метод      | URI                    | Действие       | Имя маршрута
----------|------------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

<a name="customizing-missing-model-behavior"></a>
#### Настройка поведения при отсутствии модели

Обычно, если неявно связанная модель ресурса не найдена, то генерируется `404` HTTP-ответ. Однако вы можете изменить это поведение, вызвав метод `missing` при определении вашего ресурсного маршрута. Метод `missing` принимает замыкание, которое будет вызываться, если неявно связанная модель не может быть найдена для любого из маршрутов ресурса:

    use App\Http\Controllers\PhotoController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::resource('photos', PhotoController::class)
            ->missing(function (Request $request) {
                return Redirect::route('photos.index');
            });

<a name="specifying-the-resource-model"></a>
#### Указание модели ресурса

Если вы используете [привязку модели к маршруту](routing.md#route-model-binding) и хотите, чтобы методы контроллера ресурса содержали типизацию экземпляра модели, вы можете использовать параметр `--model` при создании контроллера:

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

<a name="generating-form-requests"></a>
#### Генерация запросов формы

Вы можете указать флаг `--requests` при создании ресурсного контроллера, чтобы указать Artisan о попутном создании [классов запросов](validation.md#form-request-validation) для методов `store` и `update` контроллера:

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

<a name="restful-partial-resource-routes"></a>
### Частичные ресурсные маршруты

При объявлении маршрута ресурса вы можете указать подмножество действий, которые должен обрабатывать контроллер, вместо полного набора действий по умолчанию:

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->only([
        'index', 'show'
    ]);

    Route::resource('photos', PhotoController::class)->except([
        'create', 'store', 'update', 'destroy'
    ]);

<a name="api-resource-routes"></a>
#### Ресурсные API-маршруты

При определении маршрутов ресурса, которые будут использоваться API, бывает необходимо исключить маршруты, содержащие ответы с HTML-шаблонами, такие как `create` и `edit`. Для удобства вы можете использовать метод `apiResource`, чтобы автоматически исключить эти два маршрута:

    use App\Http\Controllers\PhotoController;

    Route::apiResource('photos', PhotoController::class);

Вы можете зарегистрировать сразу несколько ресурсных API-контроллеров, передав массив методу `apiResources`:

    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\PostController;

    Route::apiResources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

Чтобы быстро сгенерировать ресурсный API-контроллер, который не включает методы `create` или `edit`, используйте переключатель `--api` при выполнении команды `make:controller`:

```shell
php artisan make:controller PhotoController --api
```

<a name="restful-nested-resources"></a>
### Вложенные ресурсы

Иногда требуется определить маршруты ко вложенному ресурсу. Например, фоторесурс может иметь несколько комментариев, которые могут быть прикреплены к фотографии. Чтобы вложить ресурсные контроллеры, используйте «точечную нотацию» в определении маршрута:

    use App\Http\Controllers\PhotoCommentController;

    Route::resource('photos.comments', PhotoCommentController::class);

Этот маршрут зарегистрирует вложенный ресурс, к которому можно получить доступ с помощью URI, подобных следующим:

    /photos/{photo}/comments/{comment}

<a name="scoping-nested-resources"></a>
#### Ограничение вложенных ресурсов

Функционал [неявной привязки модели](routing.md#implicit-model-binding-scoping) Laravel может автоматически ограничивать вложенные привязки для подтверждения принадлежности извлеченной дочерней модели по отношению к родительской модели. Используя метод `scoped` при определении вашего вложенного ресурса, вы можете включить автоматическое ограничение, а также указать Laravel, через какое поле дочерний ресурс должен быть получен. Для получения дополнительных сведений о том, как это сделать, смотрите документацию по [ограничению ресурсных маршрутов](#restful-scoping-resource-routes).

<a name="shallow-nesting"></a>
#### Упрощенное вложение

Часто нет необходимости иметь в URI и родительский, и дочерний идентификаторы, поскольку дочерний идентификатор уже является уникальным идентификатором. При использовании уникальных идентификаторов, таких как автоинкрементные первичные ключи, для идентификации ваших моделей в сегментах URI, вы можете использовать «упрощенное вложение»:

    use App\Http\Controllers\CommentController;

    Route::resource('photos.comments', CommentController::class)->shallow();

Это объявление маршрута будет определять следующие маршруты:

Метод      | URI                               | Действие       | Имя маршрута
----------|-----------------------------------|--------------|---------------------
GET       | `/photos/{photo}/comments`        | index        | photos.comments.index
GET       | `/photos/{photo}/comments/create` | create       | photos.comments.create
POST      | `/photos/{photo}/comments`        | store        | photos.comments.store
GET       | `/comments/{comment}`             | show         | comments.show
GET       | `/comments/{comment}/edit`        | edit         | comments.edit
PUT/PATCH | `/comments/{comment}`             | update       | comments.update
DELETE    | `/comments/{comment}`             | destroy      | comments.destroy

<a name="restful-naming-resource-routes"></a>
### Именование ресурсных маршрутов

По умолчанию все действия ресурсного контроллера имеют имя маршрута; однако, вы можете переопределить эти имена, передав массив имен с желаемыми именами маршрутов:

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->names([
        'create' => 'photos.build'
    ]);

<a name="restful-naming-resource-route-parameters"></a>
### Именование параметров ресурсных маршрутов

По умолчанию `Route::resource` создаст параметры маршрута для ваших ресурсных маршрутов на основе «сингулярной» версии имени ресурса. Вы можете легко переопределить это для каждого ресурса, используя метод `parameters`. Массив, передаваемый в метод `parameters`, должен быть ассоциативным массивом имен ресурсов и имен параметров:

    use App\Http\Controllers\AdminUserController;

    Route::resource('users', AdminUserController::class)->parameters([
        'users' => 'admin_user'
    ]);

В приведенном выше примере создается следующий URI для маршрута `show` ресурса:

    /users/{admin_user}

<a name="restful-scoping-resource-routes"></a>
### Ограничение ресурсных маршрутов

Функционал [ограничения неявной привязки модели](routing.md#implicit-model-binding-scoping) Laravel может автоматически ограничивать вложенные привязки для подтверждения принадлежности извлеченной дочерней модели по отношению к родительской модели. Используя метод `scoped` при определении вашего вложенного ресурса, вы можете включить автоматическое ограничение, а также указать Laravel, через какое поле дочерний ресурс должен быть получен:

    use App\Http\Controllers\PhotoCommentController;

    Route::resource('photos.comments', PhotoCommentController::class)->scoped([
        'comment' => 'slug',
    ]);

Этот маршрут зарегистрирует ограниченный вложенный ресурс, к которому можно получить доступ с помощью таких URI, как следующий:

    /photos/{photo}/comments/{comment:slug}

При использовании пользовательской неявной привязки с ключом в качестве параметра вложенного маршрута, Laravel автоматически задает ограничение для получения вложенной модели своим родителем, используя соглашения, чтобы угадать имя отношения родительского элемента. В этом случае предполагается, что модель `Photo` имеет отношение с именем `comments` (множественное число от имени параметра маршрута), которое можно использовать для получения модели `Comment`.

<a name="restful-localizing-resource-uris"></a>
### Локализация URI ресурсов

По умолчанию `Route::resource` создает URI ресурсов с использованием английских глаголов и языковых правил множественного числа. Если вам нужно локализовать команды действий `create` и `edit`, то вы можете использовать метод `Route::resourceVerbs`. Это можно сделать в начале метода `boot` поставщика `App\Providers\RouteServiceProvider`:

    /**
     * Определить связывание модели и маршрута, фильтры шаблонов и т.д.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);

        // ...
    }

Построитель слов во множественном числе Laravel поддерживает [несколько различных языков, которые вы можете изменить в зависимости от ваших потребностей](localization.md#pluralization-language). После того, как глаголы были скорректированы, а также изменены языковые правила множественного числа, регистрация маршрута ресурса, например, `Route::resource('publicacion', PublicacionController::class)`, создаст следующие URI:

    /publicacion/crear

    /publicacion/{publicaciones}/editar

<a name="restful-supplementing-resource-controllers"></a>
### Дополнение ресурсных контроллеров

Если вам нужно добавить дополнительные маршруты ресурсного контроллера помимо набора ресурсных маршрутов по умолчанию, вы должны определить эти маршруты перед вызовом метода `Route::resource`; в противном случае маршруты, определенные методом `resource`, могут непреднамеренно иметь приоритет над вашими дополнительными маршрутами:

    use App\Http\Controller\PhotoController;

    Route::get('/photos/popular', [PhotoController::class, 'popular']);
    Route::resource('photos', PhotoController::class);

> {tip} Помните, что ваши контроллеры должны быть сосредоточенными. Если вам постоянно требуются методы, выходящие за рамки типичного набора действий с ресурсами, рассмотрите возможность разделения вашего контроллера на два меньших контроллера.

<a name="dependency-injection-and-controllers"></a>
## Внедрение зависимостей и контроллеры

<a name="constructor-injection"></a>
#### Внедрение зависимостей в конструкторе контроллера

[Контейнер служб](container.md) Laravel используется для извлечения всех контроллеров. В результате вы можете объявить любые зависимости, которые могут понадобиться вашему контроллеру в его конструкторе. Объявленные зависимости будут автоматически извлечены и внедрены в экземпляр контроллера:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * Экземпляр репозитория Пользователь.
         */
        protected $users;

        /**
         * Создать новый экземпляр контроллера.
         *
         * @param  \App\Repositories\UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

<a name="method-injection"></a>
#### Внедрение зависимостей в методах контроллера

Помимо внедрения в конструкторе, вы также можете объявить тип зависимости в методах вашего контроллера. Распространенный вариант использования внедрения в методе – это внедрение экземпляра `Illuminate\Http\Request` в методы вашего контроллера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Сохранить нового пользователя.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

Если ваш метод контроллера также ожидает входные данные из параметра маршрута, укажите аргументы маршрута после других зависимостей. Например, если ваш маршрут определен так:

    use App\Http\Controllers\UserController;

    Route::put('/user/{id}', [UserController::class, 'update']);

Вы по-прежнему можете объявить тип зависимости `Illuminate\Http\Request` и получить доступ к вашему параметру `id`, определив свой метод контроллера следующим образом:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Обновить конкретного пользователя.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  string  $id
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }
