# Laravel 9 · HTTP-ответы

- [Создание ответов](#creating-responses)
    - [Добавление заголовков к ответам](#attaching-headers-to-responses)
    - [Добавление файлов Cookies к ответам](#attaching-cookies-to-responses)
    - [Файлы Cookies и шифрование](#cookies-and-encryption)
- [Перенаправления](#redirects)
    - [Перенаправление на именованные маршруты](#redirecting-named-routes)
    - [Перенаправление к действиям контроллера](#redirecting-controller-actions)
    - [Перенаправление на внешние домены](#redirecting-external-domains)
    - [Перенаправление с кратковременным сохранением данных в сессии](#redirecting-with-flashed-session-data)
- [Другие типы ответов](#other-response-types)
    - [Ответы с HTML-шаблонами](#view-responses)
    - [Ответы JSON](#json-responses)
    - [Ответы для загрузки файлов](#file-downloads)
    - [Ответы, отображающие содержимое файлов](#file-responses)
- [Макрокоманды ответа](#response-macros)

<a name="creating-responses"></a>
## Создание ответов

<a name="strings-arrays"></a>
#### Строки и массивы

Все маршруты и контроллеры должны возвращать ответ, который будет отправлен обратно в браузер пользователя. Laravel предлагает несколько разных способов вернуть ответы. Самый простой ответ – это возврат строки из маршрута или контроллера. Фреймворк автоматически преобразует строку в полный HTTP-ответ:

    Route::get('/', function () {
        return 'Hello World';
    });

Помимо возврата строк из ваших маршрутов и контроллеров, вы также можете возвращать массивы. Фреймворк автоматически преобразует массив в ответ JSON:

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} Знаете ли вы, что можете возвращать [коллекции Eloquent](eloquent-collections.md) из ваших маршрутов или контроллеров? Они будут автоматически преобразованы в JSON.

<a name="response-objects"></a>
#### Объекты ответа

Как правило, вы не просто будете возвращать строки или массивы из действий маршрута. Вместо этого вы вернете полные экземпляры `Illuminate\Http\Response` или [шаблоны](views.md).

Возврат полного экземпляра `Response` позволяет вам настроить код состояния и заголовки HTTP ответа. Экземпляр `Response` наследуется от класса `Symfony\Component\HttpFoundation\Response`, который содержит множество методов для построения ответов HTTP:

    Route::get('/home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="eloquent-models-and-collections"></a>
#### Модели и коллекции Eloquent

По желанию можно вернуть модели и коллекции [Eloquent ORM](eloquent.md) прямо из ваших маршрутов и контроллеров. Когда вы это сделаете, Laravel автоматически преобразует модели и коллекции в ответы JSON, учитывая [скрытие атрибутов](eloquent-serialization.md#hiding-attributes-from-json) модели:

    use App\Models\User;

    Route::get('/user/{user}', function (User $user) {
        return $user;
    });

<a name="attaching-headers-to-responses"></a>
### Добавление заголовков к ответам

Имейте в виду, что большинство методов ответа можно объединять в цепочку вызовов для гибкого создания экземпляров ответа. Например, вы можете использовать метод `header` для добавления серии заголовков к ответу перед его отправкой обратно пользователю:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

Или вы можете использовать метод `withHeaders`, чтобы указать массив заголовков, которые будут добавлены к ответу:

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="cache-control-middleware"></a>
#### Посредник управления кешем

Laravel содержит посредник `cache.headers`, используемый для быстрой установки заголовка `Cache-Control` для группы маршрутов. Директивы управления кешем должны быть переданы с использованием «змеиной нотации» и точки с запятой в качестве разделителя. Если в списке директив указан `etag`, то MD5-хеш содержимого ответа будет автоматически установлен как идентификатор ETag:

    Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
        Route::get('/privacy', function () {
            // ...
        });

        Route::get('/terms', function () {
            // ...
        });
    });

<a name="attaching-cookies-to-responses"></a>
### Добавление файлов Cookies к ответам

Вы можете добавить Cookies к исходящему экземпляру `Illuminate\Http\Response`, используя метод `cookie`. Вы должны передать этому методу имя, значение и количество минут, в течение которых куки должен считаться действительным:

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

Метод `cookie` также принимает еще несколько аргументов, которые используются реже. Как правило, эти аргументы имеют то же назначение и значение, что и аргументы, передаваемые встроенному в PHP методу [`setcookie`](https://www.php.net/manual/ru/function.setcookie.php) method:

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

Если вы хотите, чтобы куки отправлялся вместе с исходящим ответом, но у вас еще нет экземпляра этого ответа, вы можете использовать фасад `Cookie`, чтобы «поставить в очередь» файлы Cookies для добавления их к ответу при его отправке. Метод `queue` принимает аргументы, необходимые для создания экземпляра `Cookie`. Эти файлы Cookies будут добавлены к исходящему ответу перед его отправкой в браузер:

    use Illuminate\Support\Facades\Cookie;

    Cookie::queue('name', 'value', $minutes);

<a name="generating-cookie-instances"></a>
#### Создание экземпляров `Cookie`

Если вы хотите сгенерировать экземпляр `Symfony\Component\HttpFoundation\Cookie`, который может быть добавлен к экземпляру ответа позже, вы можете использовать глобальный помощник `cookie`. Этот файл Cookies не будет отправлен обратно клиенту, если он не прикреплен к экземпляру ответа:

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="expiring-cookies-early"></a>
#### Досрочное окончание срока действия файлов Cookies

Вы можете удалить куки, обнулив срок его действия с помощью метода `withoutCookie` исходящего ответа:

    return response('Hello World')->withoutCookie('name');

Если у вас еще нет экземпляра исходящего ответа, вы можете использовать метод `expire` фасада `Cookie` для обнуления срока действия кук:

    Cookie::expire('name');

<a name="cookies-and-encryption"></a>
### Файлы Cookies и шифрование

По умолчанию все файлы Cookies, генерируемые Laravel, зашифрованы и подписаны, поэтому клиент не может их изменить или прочитать. Если вы хотите отключить шифрование для некоторого подмножества файлов Cookies, создаваемых вашим приложением, вы можете использовать свойство `$except` посредника `App\Http\Middleware\EncryptCookies`, находящегося в каталоге `app/Http/Middleware`:

    /**
     * Имена файлов Cookies, которые не должны быть зашифрованы.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## Перенаправления

Ответы с перенаправлением являются экземплярами класса `Illuminate\Http\RedirectResponse` и содержат корректные заголовки, необходимые для перенаправления пользователя на другой URL. Есть несколько способов сгенерировать экземпляр `RedirectResponse`. Самый простой способ – использовать глобальный помощник `redirect`:

    Route::get('/dashboard', function () {
        return redirect('home/dashboard');
    });

По желанию можно перенаправить пользователя в его предыдущее местоположение, например, когда отправленная форма является недействительной. Вы можете сделать это с помощью глобального помощника `back`. Поскольку эта функция использует [сессии](session.md), убедитесь, что маршрут, вызывающий функцию `back`, использует группу посредников `web`:

    Route::post('/user/profile', function () {
        // Валидация запроса ...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### Перенаправление на именованные маршруты

Когда вы вызываете помощник `redirect` без параметров, возвращается экземпляр `Illuminate\Routing\Redirector`, что позволяет вам вызывать любой метод экземпляра `Redirector`. Например, чтобы сгенерировать `RedirectResponse` на именованный маршрут, вы можете использовать метод `route`:

    return redirect()->route('login');

Если ваш маршрут имеет параметры, вы можете передать их в качестве второго аргумента методу `route`:

    // Для маршрута со следующим URI: /profile/{id}

    return redirect()->route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>
#### Заполнение параметров с моделей Eloquent

Если вы перенаправляете на маршрут с параметром `ID`, который извлекается из модели Eloquent, то вы можете просто передать саму модель. ID будет извлечен автоматически:

    // Для маршрута со следующим URI: /profile/{id}

    return redirect()->route('profile', [$user]);

Если вы хотите настроить значение, которое соответствует параметру маршрута, то вы можете указать столбец при определении параметра маршрута (`/profile/{id:slug}`) или переопределить метод `getRouteKey` в вашей модели Eloquent:

    /**
     * Получить значение ключа маршрута модели.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### Перенаправление к действиям контроллера

Вы также можете генерировать перенаправления на [действия контроллера](controllers.md). Для этого передайте имя контроллера и действия методу `action`:

    use App\Http\Controllers\UserController;

    return redirect()->action([UserController::class, 'index']);

Если ваш маршрут контроллера требует параметров, вы можете передать их в качестве второго аргумента методу `action`:

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### Перенаправление на внешние домены

Иногда требуется перенаправление на домен за пределами вашего приложения. Вы можете сделать это, вызвав метод `away`, который создает `RedirectResponse` без какой-либо дополнительной кодировки URL, валидации или проверки:

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>
### Перенаправление с кратковременным сохранением данных в сессии

Перенаправление на новый URL-адрес и [краткосрочная запись данных в сессию](session.md#flash-data) обычно выполняются одновременно. Обычно это делается после успешного выполнения действия, когда вы отправляете сообщение об успешном завершении в сессию. Для удобства вы можете создать экземпляр `RedirectResponse` и передать данные в сессию в единой текучей цепочке методов:

    Route::post('/user/profile', function () {
        // ...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

После перенаправления пользователя, вы можете отобразить сохраненное из [сессии](session.md) сообщение. Например, используя [синтаксис Blade](blade.md):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="redirecting-with-input"></a>
#### Перенаправление с кратковременным сохранением входных данных

Вы можете использовать метод `withInput` экземпляра `RedirectResponse`, для передачи входных данных текущего запроса в сессию перед перенаправлением пользователя в новое место. Обычно это делается, если пользователь спровоцировал ошибку валидации. После того, как входные данные были переданы в сессию, вы можете легко [получить их](requests.md#retrieving-old-input) во время следующего запроса для повторного автозаполнения формы:

    return back()->withInput();

<a name="other-response-types"></a>
## Другие типы ответов

Помощник `response` используется для генерации других типов экземпляров ответа. Когда помощник `response` вызывается без аргументов, возвращается реализация [контракта](contracts.md) `Illuminate\Contracts\Routing\ResponseFactory`. Этот контракт содержит несколько полезных методов для генерации ответов.

<a name="view-responses"></a>
### Ответы с HTML-шаблонами

Если вам нужен контроль над статусом и заголовками ответа, но также необходимо вернуть [HTML-шаблон](views.md) в качестве содержимого ответа, то вы должны использовать метод `view`:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

Конечно, вы можете использовать глобальный помощник `view`, даже если вам не нужно передавать собственные код состояния или заголовки HTTP.

<a name="json-responses"></a>
### Ответы JSON

Метод `json` автоматически установит заголовок `Content-Type` в `application/json`, а также преобразует переданный массив в JSON с помощью функции `json_encode` PHP:

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA',
    ]);

Если вы хотите создать ответ JSONP, вы можете использовать метод `json` в сочетании с методом `withCallback`:

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### Ответы для загрузки файлов

Метод `download` используется для генерации ответа, который заставляет браузер пользователя загружать файл по указанному пути. Метод `download` принимает имя файла в качестве второго аргумента метода, который будет определять имя файла, которое видит пользователь, загружающий файл. Наконец, вы можете передать массив заголовков HTTP в качестве третьего аргумента метода:

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> {note} Symfony HttpFoundation, управляющий загрузкой файлов, требует, чтобы имя загружаемого файла было в кодировке ASCII.

<a name="streamed-downloads"></a>
#### Потоковые загрузки

По желанию можно превратить строковый ответ переданной функции в загружаемый ответ без необходимости записывать результирующее содержимое на диск. В этом сценарии вы можете использовать метод `streamDownload`. Этот метод принимает в качестве аргументов замыкание, имя файла и необязательный массив заголовков:

    use App\Services\GitHub;

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

<a name="file-responses"></a>
### Ответы, отображающие содержимое файлов

Метод `file` используется для отображения файла, такого как изображение или PDF, непосредственно в браузере пользователя вместо того, чтобы инициировать загрузку. Этот метод принимает путь к файлу в качестве первого аргумента и массив заголовков в качестве второго аргумента:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## Макрокоманды ответа

Если вы хотите определить собственный ответ, который вы можете повторно использовать в различных маршрутах и контроллерах, то вы можете использовать метод `macro` фасада `Response`. Как правило, вызов этого метода осуществляется в методе `boot` одного из [поставщиков служб](providers.md) вашего приложения, например, `App\Providers\AppServiceProvider`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Загрузка любых служб приложения.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

Метод `macro` принимает имя макрокоманды как свой первый аргумент и замыкание – как второй аргумент. Замыкание макрокоманды будет выполнено при вызове имени макрокоманды из реализации `ResponseFactory` или глобального помощника `response`:

    return response()->caps('foo');
