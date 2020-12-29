# HTTP-сессия

- [Введение](#introduction)
    - [Конфигурирование](#configuration)
    - [Предварительная подготовка драйверов](#driver-prerequisites)
- [Взаимодействие с сессией](#interacting-with-the-session)
    - [Получение данных](#retrieving-data)
    - [Сохранение данных](#storing-data)
    - [Кратковременные данные](#flash-data)
    - [Удаление данных](#deleting-data)
    - [Пересоздание идентификатора сессии](#regenerating-the-session-id)
- [Блокирование сессии](#session-blocking)
- [Добавление пользовательских драйверов сессии](#adding-custom-session-drivers)
    - [Реализация драйвера](#implementing-the-driver)
    - [Регистрация драйвера](#registering-the-driver)

<a name="introduction"></a>
## Введение

Since HTTP driven applications are stateless, sessions provide a way to store information about the user across multiple requests. That user information is typically placed in a persistent store / backend that can be accessed from subsequent requests.

Laravel ships with a variety of session backends that are accessed through an expressive, unified API. Support for popular backends such as [Memcached](https://memcached.org), [Redis](https://redis.io), and databases is included.

<a name="configuration"></a>
### Конфигурирование

Your application's session configuration file is stored at `config/session.php`. Be sure to review the options available to you in this file. By default, Laravel is configured to use the `file` session driver, which will work well for many applications. If your application will be load balanced across multiple web servers, you should choose a centralized store that all servers can access, such as Redis or a database.

The session `driver` configuration option defines where session data will be stored for each request. Laravel ships with several great drivers out of the box:

<!-- <div class="content-list" markdown="1"> -->
- `file` - sessions are stored in `storage/framework/sessions`.
- `cookie` - sessions are stored in secure, encrypted cookies.
- `database` - sessions are stored in a relational database.
- `memcached` / `redis` - sessions are stored in one of these fast, cache based stores.
- `array` - sessions are stored in a PHP array and will not be persisted.
<!-- </div> -->

> {tip} The array driver is primarily used during [testing](testing.md) and prevents the data stored in the session from being persisted.

<a name="driver-prerequisites"></a>
### Предварительная подготовка драйверов

<a name="database"></a>
#### Database

When using the `database` session driver, you will need to create a table to contain the session records. An example `Schema` declaration for the table may be found below:

    Schema::create('sessions', function ($table) {
        $table->string('id')->primary();
        $table->foreignId('user_id')->nullable()->index();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity')->index();
    });

You may use the `session:table` Artisan command to generate this migration. To learn more about database migrations, you may consult the complete [migration documentation](migrations.md):

    php artisan session:table

    php artisan migrate

<a name="redis"></a>
#### Redis

Before using Redis sessions with Laravel, you will need to either install the PhpRedis PHP extension via PECL or install the `predis/predis` package (~1.0) via Composer. For more information on configuring Redis, consult Laravel's [Redis documentation](redis.md#configuration).

> {tip} In the `session` configuration file, the `connection` option may be used to specify which Redis connection is used by the session.

<a name="interacting-with-the-session"></a>
## Взаимодействие с сессией

<a name="retrieving-data"></a>
### Получение данных

There are two primary ways of working with session data in Laravel: the global `session` helper and via a `Request` instance. First, let's look at accessing the session via a `Request` instance, which can be type-hinted on a route closure or controller method. Remember, controller method dependencies are automatically injected via the Laravel [service container](container.md):

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Показать профиль конкретного пользователя.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

When you retrieve an item from the session, you may also pass a default value as the second argument to the `get` method. This default value will be returned if the specified key does not exist in the session. If you pass a closure as the default value to the `get` method and the requested key does not exist, the closure will be executed and its result returned:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

<a name="the-global-session-helper"></a>
#### Глобальный помощник `session`

You may also use the global `session` PHP function to retrieve and store data in the session. When the `session` helper is called with a single, string argument, it will return the value of that session key. When the helper is called with an array of key / value pairs, those values will be stored in the session:

    Route::get('/home', function () {
        // Получить часть данных из сессии ...
        $value = session('key');

        // Получить с указанием значения по умолчанию ...
        $value = session('key', 'default');

        // Сохранить часть данных в сессию ...
        session(['key' => 'value']);
    });

> {tip} There is little practical difference between using the session via an HTTP request instance versus using the global `session` helper. Both methods are [testable](testing.md) via the `assertSessionHas` method which is available in all of your test cases.

<a name="retrieving-all-session-data"></a>
#### Получение всех данных сессии

If you would like to retrieve all the data in the session, you may use the `all` method:

    $data = $request->session()->all();

<a name="determining-if-an-item-exists-in-the-session"></a>
#### Determining If An Item Exists In The Session

To determine if an item is present in the session, you may use the `has` method. The `has` method returns `true` if the item is present and is not `null`:

    if ($request->session()->has('users')) {
        //
    }

To determine if an item is present in the session, even if its value is `null`, you may use the `exists` method:

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### Сохранение данных

To store data in the session, you will typically use the request instance's `put` method or the `session` helper:

    // Через экземпляр запроса ...
    $request->session()->put('key', 'value');

    // Через глобальный помощник «session» ...
    session(['key' => 'value']);

<a name="pushing-to-array-session-values"></a>
#### Добавление в массив значений сессии

The `push` method may be used to push a new value onto a session value that is an array. For example, if the `user.teams` key contains an array of team names, you may push a new value onto the array like so:

    $request->session()->push('user.teams', 'developers');

<a name="retrieving-deleting-an-item"></a>
#### Получение с последующим удалением элемента

The `pull` method will retrieve and delete an item from the session in a single statement:

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### Кратковременные данные

Sometimes you may wish to store items in the session for the next request. You may do so using the `flash` method. Data stored in the session using this method will be available immediately and during the subsequent HTTP request. After the subsequent HTTP request, the flashed data will be deleted. Flash data is primarily useful for short-lived status messages:

    $request->session()->flash('status', 'Task was successful!');

If you need to persist your flash data for several requests, you may use the `reflash` method, which will keep all of the flash data for an additional request. If you only need to keep specific flash data, you may use the `keep` method:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### Удаление данных

The `forget` method will remove a piece of data from the session. If you would like to remove all data from the session, you may use the `flush` method:

    // Удалить единственный ключ ...
    $request->session()->forget('name');

    // Удалить несколько ключей ...
    $request->session()->forget(['name', 'status']);

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### Пересоздание идентификатора сессии

Regenerating the session ID is often done in order to prevent malicious users from exploiting a [session fixation](https://owasp.org/www-community/attacks/Session_fixation) attack on your application.

Laravel automatically regenerates the session ID during authentication if you are using one of the Laravel [application starter kits](starter-kits.md) or [Laravel Fortify](fortify.md); however, if you need to manually regenerate the session ID, you may use the `regenerate` method.

    $request->session()->regenerate();

<a name="session-blocking"></a>
## Блокирование сессии

> {note} To utilize session blocking, your application must be using a cache driver that supports [atomic locks](cache.md#atomic-locks). Currently, those cache drivers include the `memcached`, `dynamodb`, `redis`, and `database` drivers. In addition, you may not use the `cookie` session driver.

By default, Laravel allows requests using the same session to execute concurrently. So, for example, if you use a JavaScript HTTP library to make two HTTP requests to your application, they will both execute at the same time. For many applications, this is not a problem; however, session data loss can occur in a small subset of applications that make concurrent requests to two different application endpoints which both write data to the session.

To mitigate this, Laravel provides functionality that allows you to limit concurrent requests for a given session. To get started, you may simply chain the `block` method onto your route definition. In this example, an incoming request to the `/profile` endpoint would acquire a session lock. While this lock is being held, any incoming requests to the `/profile` or `/order` endpoints which share the same session ID will wait for the first request to finish executing before continuing their execution:

    Route::post('/profile', function () {
        //
    })->block($lockSeconds = 10, $waitSeconds = 10)

    Route::post('/order', function () {
        //
    })->block($lockSeconds = 10, $waitSeconds = 10)

The `block` method accepts two optional arguments. The first argument accepted by the `block` method is the maximum number of seconds the session lock should be held for before it is released. Of course, if the request finishes executing before this time the lock will be released earlier.

The second argument accepted by the `block` method is the number of seconds a request should wait while attempting to obtain a session lock. An `Illuminate\Contracts\Cache\LockTimeoutException` will be thrown if the request is unable to obtain a session lock within the given number of seconds.

If neither of these arguments are passed, the lock will be obtained for a maximum of 10 seconds and requests will wait a maximum of 10 seconds while attempting to obtain a lock:

    Route::post('/profile', function () {
        //
    })->block()

<a name="adding-custom-session-drivers"></a>
## Добавление пользовательских драйверов сессии

<a name="implementing-the-driver"></a>
#### Реализация драйвера

If none of the existing session drivers fit your application's needs, Laravel makes it possible to write your own session handler. Your custom session driver should implement PHP's built-in `SessionHandlerInterface`. This interface contains just a few simple methods. A stubbed MongoDB implementation looks like the following:

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> {tip} Laravel does not ship with a directory to contain your extensions. You are free to place them anywhere you like. In this example, we have created an `Extensions` directory to house the `MongoSessionHandler`.

Since the purpose of these methods is not readily understandable, let's quickly cover what each of the methods do:

<!-- <div class="content-list" markdown="1"> -->
- The `open` method would typically be used in file based session store systems. Since Laravel ships with a `file` session driver, you will almost never need to put anything in this method. You can simply leave this method empty.
- The `close` method, like the `open` method, can also usually be disregarded. For most drivers, it is not needed.
- The `read` method should return the string version of the session data associated with the given `$sessionId`. There is no need to do any serialization or other encoding when retrieving or storing session data in your driver, as Laravel will perform the serialization for you.
- The `write` method should write the given `$data` string associated with the `$sessionId` to some persistent storage system, such as MongoDB or another storage system of your choice.  Again, you should not perform any serialization - Laravel will have already handled that for you.
- The `destroy` method should remove the data associated with the `$sessionId` from persistent storage.
- The `gc` method should destroy all session data that is older than the given `$lifetime`, which is a UNIX timestamp. For self-expiring systems like Memcached and Redis, this method may be left empty.
<!-- </div> -->

<a name="registering-the-driver"></a>
#### Регистрация драйвера

Once your driver has been implemented, you are ready to register it with Laravel. To add additional drivers to Laravel's session backend, you may use the `extend` method provided by the `Session` [facade](facades.md). You should call the `extend` method from the `boot` method of a [service provider](providers.md). You may do this from the existing `App\Providers\AppServiceProvider` or create an entirely new provider:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Регистрация любых служб приложения.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Загрузка любых служб приложения.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function ($app) {
                // Return an implementation of SessionHandlerInterface...
                return new MongoSessionHandler;
            });
        }
    }

Once the session driver has been registered, you may use the `mongo` driver in your `config/session.php` configuration file.
