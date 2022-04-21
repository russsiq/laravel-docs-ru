# Laravel 9 · Контейнер служб

- [Введение](#introduction)
    - [Неконфигурируемое внедрение](#zero-configuration-resolution)
    - [Когда использовать контейнер](#when-to-use-the-container)
- [Связывание](#binding)
    - [Основы связываний](#binding-basics)
    - [Связывание интерфейсов и реализаций](#binding-interfaces-to-implementations)
    - [Контекстная привязка](#contextual-binding)
    - [Связывание примитивов](#binding-primitives)
    - [Связывание типизированных вариаций](#binding-typed-variadics)
    - [Добавление меток](#tagging)
    - [Расширяемость связываний](#extending-bindings)
- [Извлечение](#resolving)
    - [Метод `make`](#the-make-method)
    - [Автоматическое внедрение зависимостей](#automatic-injection)
- [Вызов и внедрение метода](#method-invocation-and-injection)
- [События контейнера](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## Введение

Контейнер служб Laravel – это мощный инструмент для управления зависимостями классов и выполнения внедрения зависимостей. Внедрение зависимостей – это причудливая фраза, которая по существу означает следующее: зависимости классов «вводятся» в класс через конструктор или, в некоторых случаях, через методы «сеттеры».

Давайте посмотрим на простой пример:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Repositories\UserRepository;
    use App\Models\User;

    class UserController extends Controller
    {
        /**
         * Реализация репозитория User.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Создать новый экземпляр контроллера.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Показать профиль конкретного пользователя.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

В этом примере `UserController` необходимо получить пользователей из источника данных. Итак, мы **внедрим** службу, которая может получать пользователей. В этом контексте наш `UserRepository`, скорее всего, использует [Eloquent](eloquent.md) для получения информации о пользователе из базы данных. Однако, поскольку репозиторий внедрен, мы можем легко заменить его другой реализацией. Мы также можем легко «имитировать» или создать фиктивную реализацию `UserRepository` при тестировании нашего приложения.

Глубокое понимание контейнера служб Laravel необходимо для создания большого, мощного приложения, а также для внесения вклада в само ядро Laravel.

<a name="zero-configuration-resolution"></a>
### Неконфигурируемое внедрение

Если класс не имеет зависимостей или зависит только от других конкретных классов (не интерфейсов), то контейнеру не нужно указывать как создавать этот класс. Например, вы можете поместить следующий код в свой файл `routes/web.php`:

    <?php

    class Service
    {
        //
    }

    Route::get('/', function (Service $service) {
        die(get_class($service));
    });

В этом примере, при посещении `/` вашего приложения, маршрут автоматически получит класс `Service` и внедрит его в обработчик вашего маршрута. Это меняет правила игры. Это означает, что вы можете разработать свое приложение и воспользоваться преимуществами внедрения зависимостей, не беспокоясь о раздутых файлах конфигурации.

К счастью, многие классы, которые вы будете писать при создании приложения Laravel, автоматически получают свои зависимости через контейнер, включая [контроллеры](controllers.md), [слушатели событий](events.md), [посредники](middleware.md ) и т.д. Кроме того, вы можете объявить зависимости в методе `handle` [заданиям в очередях](queues.md). Как только вы почувствуете всю мощь автоматического неконфигурируемого внедрения зависимостей, вы почувствуете невозможность разработки без нее.

<a name="when-to-use-the-container"></a>
### Когда использовать контейнер

Благодаря неконфигурируемому внедрению, вы часто будете объявлять типы зависимостей в маршрутах, контроллерах, слушателях событий и других местах, не взаимодействуя с контейнером напрямую. Например, вы можете указать объект `Illuminate\Http\Request` в определении вашего маршрута, чтобы вы могли легко получить доступ к текущему запросу. Несмотря на то, что нам никогда не нужно взаимодействовать с контейнером для написания этого кода, он управляет внедрением этих зависимостей за кулисами:

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        // ...
    });

Во многих случаях, благодаря автоматическому внедрению зависимостей и [фасадам](facades.md), вы можете строить приложения Laravel без необходимости **когда-либо** вручную связывать или извлекать что-либо из контейнера. **Итак, когда бы вы могли вручную взаимодействовать с контейнером?**. Давайте рассмотрим две ситуации.

Во-первых, если вы пишете класс, реализующий интерфейс, и хотите объявить тип этого интерфейса в конструкторе маршрута или класса, то вы должны [сообщить контейнеру, как получить этот интерфейс](#binding-interfaces-to-implementations). Во-вторых, если вы [пишете пакет Laravel](packages.md), которым планируете поделиться с другими разработчиками Laravel, то вам может потребоваться связать в контейнере службы вашего пакета.

<a name="binding"></a>
## Связывание

<a name="binding-basics"></a>
### Основы связываний

<a name="simple-bindings"></a>
#### Простое связывание

Почти все ваши связывания в контейнере служб будут зарегистрированы в [поставщиках служб](providers.md), поэтому в большинстве этих примеров будет продемонстрировано использование контейнера в этом контексте.

Внутри поставщика служб у вас всегда есть доступ к контейнеру через свойство `$this->app`. Мы можем зарегистрировать связывание, используя метод `bind`, передав имя класса или интерфейса, которые мы хотим зарегистрировать, вместе с замыканием, возвращающем экземпляр класса:

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $this->app->bind(Transistor::class, function ($app) {
        return new Transistor($app->make(PodcastParser::class));
    });

Обратите внимание, что мы получаем сам контейнер в качестве аргумента. Затем мы можем использовать контейнер для извлечения под-зависимостей объекта, который мы создаем.

Как уже упоминалось, вы обычно будете взаимодействовать с контейнером внутри поставщиков служб; однако, если вы хотите взаимодействовать с контейнером вне поставщика услуг, вы можете сделать это через [фасад](facades.md) `App`:

    use App\Services\Transistor;
    use Illuminate\Support\Facades\App;

    App::bind(Transistor::class, function ($app) {
        // ...
    });

> {tip} Нет необходимости привязывать классы в контейнере, если они не зависят от каких-либо интерфейсов. Контейнеру не нужно указывать, как создавать эти объекты, поскольку он может автоматически извлекать эти объекты с помощью рефлексии.

<a name="binding-a-singleton"></a>
#### Связывание одиночек

Метод `singleton` связывает класс или интерфейс в контейнере, который должен быть извлечен только один раз. После получения одиночки, тот же экземпляр объекта будет возвращен из контейнера и при последующих вызовах:

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $this->app->singleton(Transistor::class, function ($app) {
        return new Transistor($app->make(PodcastParser::class));
    });

<a name="binding-scoped"></a>
#### Контекстное связывание одиночек

Метод `scoped` связывает класс или интерфейс в контейнере, который должен быть извлечен только один раз в течение текущего жизненного цикла запроса / задания Laravel. Хотя этот метод похож на метод `singleton`, экземпляры, зарегистрированные с помощью метода `scoped`, будут обновлены всякий раз, когда приложение Laravel запускает новый «жизненный цикл», например, когда обработчик [Laravel Octane](octane.md) обрабатывает новый запрос или когда [обработчик очереди](queues.md) Laravel обрабатывает новое задание:

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $this->app->scoped(Transistor::class, function ($app) {
        return new Transistor($app->make(PodcastParser::class));
    });

<a name="binding-instances"></a>
#### Связывание экземпляров

Вы также можете привязать существующий экземпляр объекта в контейнере, используя метод `instance`. Переданный экземпляр всегда будет возвращен из контейнера при последующих вызовах:

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $service = new Transistor(new PodcastParser);

    $this->app->instance(Transistor::class, $service);

<a name="binding-interfaces-to-implementations"></a>
### Связывание интерфейсов и реализаций

Очень мощная функция контейнера служб – это его способность связывать интерфейс с конкретной реализацией. Например, предположим, что у нас есть интерфейс `EventPusher` и реализация `RedisEventPusher`. После того, как мы написали нашу реализацию `RedisEventPusher` этого интерфейса, мы можем зарегистрировать его в контейнере следующим образом:

    use App\Contracts\EventPusher;
    use App\Services\RedisEventPusher;

    $this->app->bind(EventPusher::class, RedisEventPusher::class);

Эта запись сообщает контейнеру, что он должен внедрить `RedisEventPusher`, когда классу требуется реализация `EventPusher`. Теперь мы можем указать интерфейс `EventPusher` в конструкторе класса, который будет извлечен контейнером. Помните, что контроллеры, слушатели событий, посредники и различные другие типы классов в приложениях Laravel всегда выполняются с помощью контейнера:

    use App\Contracts\EventPusher;

    /**
     * Создать новый экземпляр класса.
     *
     * @param  \App\Contracts\EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Контекстная привязка

Иногда у вас может быть два класса, которые используют один и тот же интерфейс, но вы хотите внедрить разные реализации в каждый класс. Например, два контроллера могут зависеть от разных реализаций [контракта](contracts.md) `Illuminate\Contracts\Filesystem\Filesystem`. Laravel предлагает простой и понятный интерфейс для определения этого поведения:

    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\UploadController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;
    use Illuminate\Support\Facades\Storage;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when([VideoController::class, UploadController::class])
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="binding-primitives"></a>
### Связывание примитивов

Иногда у вас может быть класс, который получает некоторые внедренные классы, но также нуждается в примитиве, таком как целое число. Вы можете легко использовать контекстную привязку, чтобы внедрить любое значение, которое может понадобиться вашему классу:

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

Иногда класс может зависеть от массива экземпляров, объединенных [меткой](#tagging). Используя метод `giveTagged`, вы можете легко их внедрить:

    $this->app->when(ReportAggregator::class)
        ->needs('$reports')
        ->giveTagged('reports');

Если вам нужно внедрить значение из одного из конфигурационных файлов вашего приложения, то вы можете использовать метод `giveConfig`:

    $this->app->when(ReportAggregator::class)
        ->needs('$timezone')
        ->giveConfig('app.timezone');

<a name="binding-typed-variadics"></a>
### Связывание типизированных вариаций

Иногда у вас может быть класс, который получает массив типизированных объектов с использованием переменного количества аргументов (_прим. перев.: далее «вариации»_) конструктора:

    <?php

    use App\Models\Filter;
    use App\Services\Logger;

    class Firewall
    {
        /**
         * Экземпляр регистратора.
         *
         * @var \App\Services\Logger
         */
        protected $logger;

        /**
         * Массив фильтров.
         *
         * @var array
         */
        protected $filters;

        /**
         * Создать новый экземпляр класса.
         *
         * @param  \App\Services\Logger  $logger
         * @param  array  $filters
         * @return void
         */
        public function __construct(Logger $logger, Filter ...$filters)
        {
            $this->logger = $logger;
            $this->filters = $filters;
        }
    }

Используя контекстную привязку, вы можете внедрить такую зависимость, используя метод `give` с замыканием, которое возвращает массив внедряемых экземпляров `Filter`:

    $this->app->when(Firewall::class)
              ->needs(Filter::class)
              ->give(function ($app) {
                    return [
                        $app->make(NullFilter::class),
                        $app->make(ProfanityFilter::class),
                        $app->make(TooLongFilter::class),
                    ];
              });

Для удобства вы также можете просто передать массив имен классов, которые будут предоставлены контейнером всякий раз, когда для `Firewall` нужны экземпляры `Filter`:

    $this->app->when(Firewall::class)
              ->needs(Filter::class)
              ->give([
                  NullFilter::class,
                  ProfanityFilter::class,
                  TooLongFilter::class,
              ]);

<a name="variadic-tag-dependencies"></a>
#### Метки вариативных зависимостей

Иногда класс может иметь вариативную зависимость, указывающую на тип как переданный класс (`Report ...$reports`). Используя методы `needs` и `giveTagged`, вы можете легко внедрить все привязки контейнера с этой [меткой](#tagging) для указанной зависимости:

    $this->app->when(ReportAggregator::class)
        ->needs(Report::class)
        ->giveTagged('reports');

<a name="tagging"></a>
### Добавление меток

Иногда требуется получить все связывания определенной «категории». Например, возможно, вы создаете анализатор отчетов, который получает массив из множества различных реализаций интерфейса `Report`. После регистрации реализаций `Report` вы можете назначить им метку с помощью метода `tag`:

    $this->app->bind(CpuReport::class, function () {
        //
    });

    $this->app->bind(MemoryReport::class, function () {
        //
    });

    $this->app->tag([CpuReport::class, MemoryReport::class], 'reports');

После того, как службы помечены, вы можете легко все их получить с помощью метода `tagged`:

    $this->app->bind(ReportAnalyzer::class, function ($app) {
        return new ReportAnalyzer($app->tagged('reports'));
    });

<a name="extending-bindings"></a>
### Расширяемость связываний

Метод `extend` позволяет модифицировать извлеченные службы. Например, когда служба получена, вы можете запустить дополнительный код для декорирования или конфигурирования службы. Метод `extend` принимает два аргумента: класс службы, который вы расширяете, и замыкание, которое должно возвращать измененную службу. Замыкание получает службу для извлечения и экземпляр контейнера:

    $this->app->extend(Service::class, function ($service, $app) {
        return new DecoratedService($service);
    });

<a name="resolving"></a>
## Извлечение

<a name="the-make-method"></a>
### Метод `make`

Вы можете использовать метод `make` для извлечения экземпляра класса из контейнера. Метод `make` принимает имя класса или интерфейса, который вы хотите получить:

    use App\Services\Transistor;

    $transistor = $this->app->make(Transistor::class);

Если некоторые зависимости вашего класса не могут быть разрешены через контейнер, вы можете ввести их, передав их как ассоциативный массив в метод `makeWith`. Например, мы можем вручную передать конструктору аргумент `$id`, требуемый службой `Transistor`:

    use App\Services\Transistor;

    $transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);

Если вы находитесь за пределами поставщика служб и не имеете доступа к переменной `$app`, вы можете использовать [фасад](facades.md) `App` для получения экземпляра класса из контейнера:

    use App\Services\Transistor;
    use Illuminate\Support\Facades\App;

    $transistor = App::make(Transistor::class);

Если вы хотите, чтобы сам экземпляр контейнера Laravel был внедрен в класс, извлекаемый контейнером, вы можете указать класс `Illuminate\Container\Container` в конструкторе вашего класса:

    use Illuminate\Container\Container;

    /**
     * Создать новый экземпляр класса.
     *
     * @param  \Illuminate\Container\Container  $container
     * @return void
     */
    public function __construct(Container $container)
    {
        $this->container = $container;
    }

<a name="automatic-injection"></a>
### Автоматическое внедрение зависимостей

В качестве альтернативы, что важно, вы можете объявить тип зависимости в конструкторе класса, который извлекается контейнером, включая [контроллеры](controllers.md), [слушатели событий](events.md), [посредники](middleware.md ) и т.д. Кроме того, вы можете объявить зависимости в методе `handle` [заданиям в очередях](queues.md). На практике именно так контейнер должен извлекать большинство ваших объектов.

Например, вы можете объявить репозиторий, определенный вашим приложением, в конструкторе контроллера. Репозиторий будет автоматически получен и внедрен в класс:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * Экземпляр репозитория Пользователь.
         *
         * @var \App\Repositories\UserRepository
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

        /**
         * Показать пользователя с переданным идентификатором.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="method-invocation-and-injection"></a>
## Вызов и внедрение метода

Иногда требуется вызвать метод экземпляра объекта и позволить контейнеру автоматически внедрить зависимости этого метода. Например, учитывая следующий класс:

    <?php

    namespace App;

    use App\Repositories\UserRepository;

    class UserReport
    {
        /**
         * Создать новый пользовательский отчет.
         *
         * @param  \App\Repositories\UserRepository  $repository
         * @return array
         */
        public function generate(UserRepository $repository)
        {
            // ...
        }
    }

Вы можете вызвать метод `generate` через контейнер следующим образом:

    use App\UserReport;
    use Illuminate\Support\Facades\App;

    $report = App::call([new UserReport, 'generate']);

Метод `call` принимает любой тип [callable](https://www.php.net/manual/ru/language.types.callable.php) PHP. Метод контейнера `call` может даже использоваться для вызова замыкания с автоматическим внедрением его зависимостей:

    use App\Repositories\UserRepository;
    use Illuminate\Support\Facades\App;

    $result = App::call(function (UserRepository $repository) {
        // ...
    });

<a name="container-events"></a>
## События контейнера

Контейнер служб инициирует событие каждый раз, когда извлекает объект. Вы можете прослушать это событие с помощью метода `resolving`:

    use App\Services\Transistor;

    $this->app->resolving(Transistor::class, function ($transistor, $app) {
        // Вызывается, когда контейнер извлекает объекты типа `Transistor` ...
    });

    $this->app->resolving(function ($object, $app) {
        // Вызывается, когда контейнер извлекает объект любого типа ...
    });

Как видите, извлекаемый объект будет передан в замыкание, что позволит вам установить любые дополнительные свойства объекта до того, как он будет передан его получателю.

<a name="psr-11"></a>
## PSR-11

Контейнер служб Laravel реализует интерфейс [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Следовательно, вы можете объявить тип зависимости от интерфейса контейнера, чтобы получить экземпляр контейнера Laravel:

    use App\Services\Transistor;
    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get(Transistor::class);

        //
    });

Если переданный идентификатор не может быть получен, то будет выброшено исключение. Исключение будет экземпляром `Psr\Container\NotFoundExceptionInterface`, если идентификатор никогда не был привязан. Если идентификатор был привязан, но не может быть извлечен, то будет выброшено исключение экземпляра `Psr\Container\ContainerExceptionInterface`.
