# Eloquent: начало работы

- [Введение](#introduction)
- [Определение моделей](#defining-models)
    - [Соглашения по именованию моделей Eloquent](#eloquent-model-conventions)
    - [Значения атрибутов по умолчанию](#default-attribute-values)
- [Получение моделей](#retrieving-models)
    - [Коллекции](#collections)
    - [Разбиение результатов](#chunking-results)
    - [Расширенные подзапросы](#advanced-subqueries)
- [Извлечение отдельных моделей](#retrieving-single-models)
    - [Извлечение Агрегатов](#retrieving-aggregates)
- [Вставка и обновление моделей](#inserting-and-updating-models)
    - [Вставка](#inserts)
    - [Обновление](#updates)
    - [Массовое присвоение](#mass-assignment)
    - [Другие методы создания](#other-creation-methods)
- [Удаление моделей](#deleting-models)
    - [Программное удаление](#soft-deleting)
    - [Запросы для моделей, использующих программное удаление](#querying-soft-deleted-models)
- [Репликация (тиражирование) моделей](#replicating-models)
- [Диапазоны запроса](#query-scopes)
    - [Глобальные диапазоны](#global-scopes)
    - [Локальные диапазоны](#local-scopes)
- [Сравнение моделей](#comparing-models)
- [События](#events)
    - [Использование замыканий](#events-using-closures)
    - [Наблюдатели](#observers)
    - [Подавление событий](#muting-events)

<a name="introduction"></a>
## Введение

Eloquent ORM, включенный в Laravel, предоставляет красивую и простую реализацию ActiveRecord для работы с вашей базой данных. Каждая таблица базы данных имеет соответствующую «Модель», которая используется для взаимодействия с этой таблицей. Модели позволяют запрашивать данные в таблицах, а также вставлять новые записи в таблицу.

Перед началом работы обязательно настройте соединение с базой данных в `config/database.php`. Для получения дополнительной информации о настройке базы данных ознакомьтесь с [документацией](database.md#configuration).

<a name="defining-models"></a>
## Определение моделей

Для начала создадим модель Eloquent. Модели обычно находятся в каталоге `app\Models`, но вы можете разместить их в любом месте, которое может быть автоматически загружено в соответствии с вашим файлом `composer.json`. Все модели Eloquent расширяют класс `Illuminate\Database\Eloquent\Model`.

Самый простой способ создать экземпляр модели – использовать команду `make:model` [консоли Artisan](artisan.md):

    php artisan make:model Flight

При создании модели вы можете сгенерировать [миграцию базы данных](migrations.md), используя параметр `--migration` или `-m`:

    php artisan make:model Flight --migration

    php artisan make:model Flight -m

При создании модели вы можете также создавать другие различные типы классов, например фабрики, наполнители и контроллеры. Кроме того, эти параметры можно комбинировать для создания сразу нескольких классов:

    php artisan make:model Flight --factory
    php artisan make:model Flight -f

    php artisan make:model Flight --seed
    php artisan make:model Flight -s

    php artisan make:model Flight --controller
    php artisan make:model Flight -c

    php artisan make:model Flight -mfsc

<a name="eloquent-model-conventions"></a>
### Соглашения по именованию моделей Eloquent

Теперь давайте рассмотрим пример модели `Flight`, которую мы будем использовать для извлечения и сохранения информации в нашей таблице базы данных `flights`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }

<a name="table-names"></a>
#### Именование таблиц

Обратите внимание, что мы не указали Eloquent, какую таблицу использовать для нашей модели `Flight`. По соглашению, в качестве имени таблицы будет использоваться имя класса в «змеином_регистре», во множественном числе, если явно не указано другое. В нашем случае, Eloquent будет предполагать, что модель `Flight` хранит записи в таблице `flights`, а модель `AirTrafficController` – в таблице `air_traffic_controllers`.

Вы можете указать иное имя таблицы, определив свойство `table` в модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Таблица БД, ассоциированная с моделью.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

<a name="primary-keys"></a>
#### Первичные ключи

Eloquent также предполагает, что каждая таблица имеет столбец первичного ключа с именем `id`. Вы можете указать защищенное свойство `$primaryKey`, чтобы переопределить это соглашение:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Первичный ключ таблицы БД.
         *
         * @var string
         */
        protected $primaryKey = 'flight_id';
    }

Кроме того, Eloquent предполагает, что первичный ключ является автоинкрементным целочисленным значением, что означает, что по умолчанию первичный ключ будет автоматически приведен к `int`. Если вы хотите использовать неинкрементный или нечисловой первичный ключ, вы должны указать публичное свойство `$incrementing` в модели как false:

    <?php

    class Flight extends Model
    {
        /**
         * Указывает, что идентификаторы являются автоинкрементными.
         *
         * @var bool
         */
        public $incrementing = false;
    }

Если первичный ключ не является целочисленным, вы должны указать значение `string` для защищенного свойства `$keyType`:

    <?php

    class Flight extends Model
    {
        /**
         * Тип автоинкрементного идентификатора.
         *
         * @var string
         */
        protected $keyType = 'string';
    }

<a name="timestamps"></a>
#### Временные метки

По умолчанию Eloquent ожидает, что в таблицах существуют столбцы `created_at` и `updated_at`. Если вы не хотите, чтобы Eloquent автоматически управлял этими столбцами, укажите значение `false` для свойства `$timestamps` в модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Следует ли обрабатывать временные метки модели.
         *
         * @var bool
         */
        public $timestamps = false;
    }

Если вам нужно настроить формат временных меток, то укажите необходимый формат для свойства `$dateFormat`. Это свойство определяет, как атрибуты даты хранятся в базе данных, а также их формат при сериализации модели в массив или JSON:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Формат хранения столбцов даты модели.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

Если вам нужно настроить имена столбцов, используемых для хранения временных меток, то укажите значения для констант `CREATED_AT` и `UPDATED_AT` в модели:

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

<a name="database-connection"></a>
#### Соединение с базой данных

Все модели Eloquent будут использовать подключение к базе данных по умолчанию, настроенное для вашего приложения. Если вы хотите указать другое соединение для модели, используйте свойство `$connection`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Имя соединения с БД для модели.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="default-attribute-values"></a>
### Значения атрибутов по умолчанию

Если вы хотите определить значения по умолчанию для некоторых атрибутов модели, то укажите необходимые значения в свойстве `$attributes` модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Значения по умолчанию для атрибутов модели.
         *
         * @var array
         */
        protected $attributes = [
            'delayed' => false,
        ];
    }

<a name="retrieving-models"></a>
## Получение моделей

Создав модель и [связанную с ней таблицу базы данных](migrations.md#writing-migrations), все готово к извлечению данных из базы данных. Думайте о каждой модели Eloquent как о мощном [построителе запросов](queries.md), позволяющем свободно выполнять запросы к таблице базы данных, связанной с моделью. Например:

    <?php

    $flights = App\Models\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="adding-additional-constraints"></a>
#### Добавление дополнительных условий

Метод Eloquent `all` возвращает все результаты из таблицы модели. Поскольку каждая модель Eloquent служит [построителем запросов](queries.md), вы также можете добавить условия к запросам, а затем использовать метод `get` для получения результатов:

    $flights = App\Models\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} Поскольку модель Eloquent является построителем запросов, вам следует просмотреть все методы, доступные [построителю запросов](queries.md). Вы можете использовать любой из этих методов в запросах Eloquent.

<a name="refreshing-models"></a>
#### Обновление моделей

Вы можете обновить модели, используя методы `fresh` и `refresh`. Метод `fresh` повторно извлечет модель из базы данных. Существующий экземпляр модели не будет затронут:

    $flight = App\Models\Flight::where('number', 'FR 900')->first();

    $freshFlight = $flight->fresh();

Метод `refresh` повторно обновит существующую модель, используя свежие данные из базы данных. Кроме того, будут обновлены все загруженные отношения:

    $flight = App\Models\Flight::where('number', 'FR 900')->first();

    $flight->number = 'FR 456';

    $flight->refresh();

    $flight->number; // "FR 900"

<a name="collections"></a>
### Коллекции

Для методов Eloquent, таких как `all` и `get`, которые получают множественные результаты, будет возвращен экземпляр `Illuminate\Database\Eloquent\Collection`. Класс `Collection` предоставляет [множество полезных методов](eloquent-collections.md#available-methods) для работы с результатами Eloquent:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

Вы также можете перебрать коллекцию как массив:

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### Разбиение результатов

Если вам нужно обработать тысячи записей Eloquent, используйте метод `chunk`. Метод `chunk` будет извлекать «порцию» моделей Eloquent, передавая их в замыкание для обработки. Использование метода `chunk` сэкономит память при работе с большим результирующим набором:

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

Первым аргументом, передаваемым методу, является количество записей, которые вы хотите получить за «порцию». Замыкание, переданное в качестве второго аргумента, будет вызываться для каждой части записей, полученной из базы данных. Будет выполнен запрос к базе данных для получения каждой части записей, переданных замыканию.

Если вы фильтруете результаты метода `chunk` на основе столбца, который вы также будете обновлять при итерации результатов, вам следует использовать метод `chunkById`. Использование метода `chunk` в этих сценариях может привести к неожиданным и противоречивым результатам:

    Flight::where('departed', true)->chunkById(200, function ($flights) {
        $flights->each->update(['departed' => false]);
    });

<a name="using-cursors"></a>
#### Использование курсоров

Метод `cursor` позволяет перебирать записи базы данных с помощью курсора, который будет выполнять только один запрос. При обработке больших объемов данных можно использовать метод `cursor` для значительного уменьшения использования памяти:

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

Курсор возвращает экземпляр `Illuminate\Support\LazyCollection`. [Отложенные коллекции](collections.md#lazy-collections) позволяют использовать многие методы коллекций, доступные в типичных коллекциях Laravel, при одновременной загрузке в память только одной модели:

    $users = App\Models\User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

<a name="advanced-subqueries"></a>
### Расширенные подзапросы

<a name="subquery-selects"></a>
#### Выборка

Eloquent также предлагает поддержку расширенных подзапросов, которая позволяет извлекать информацию из связанных таблиц в одном запросе. Например, давайте представим, что у нас есть таблица `destinations` (пункты назначения) и `flights` (рейсы). В таблице `flights` содержится столбец `arrived_at`, который указывает, когда рейс прибыл в пункт назначения.

Используя функциональность подзапроса, доступную для методов `select` и `addSelect`, мы можем выбрать все `destinations` и название рейса, который последним прибыл в этот пункт назначения, используя один запрос:

    use App\Models\Destination;
    use App\Models\Flight;

    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
    ])->get();

<a name="subquery-ordering"></a>
#### Сортировка

Кроме того, метод `orderBy` построителя запросов поддерживает подзапросы. Мы можем использовать эту функцию для сортировки всех пунктов назначения в зависимости от того, когда последний рейс прибыл в этот пункт назначения. Опять же, это можно сделать при выполнении одного запроса к базе данных:

    return Destination::orderByDesc(
        Flight::select('arrived_at')
            ->whereColumn('destination_id', 'destinations.id')
            ->orderBy('arrived_at', 'desc')
            ->limit(1)
    )->get();

<a name="retrieving-single-models"></a>
## Извлечение отдельных моделей

В дополнение к извлечению всех записей таблицы, вы также можете извлечь отдельные записи, используя `find`, `first` или `firstWhere`. Вместо того, чтобы возвращать коллекцию моделей, эти методы возвращают один экземпляр модели:

    // Получить модель по ее первичному ключу ...
    $flight = App\Models\Flight::find(1);

    // Получить первую модель, соответствующую условиям запроса ...
    $flight = App\Models\Flight::where('active', 1)->first();

    // Сокращение для получения первой модели, соответствующей условиям запроса ...
    $flight = App\Models\Flight::firstWhere('active', 1);

Вы также можете вызвать метод `find` с массивом первичных ключей, который вернет коллекцию соответствующих записей:

    $flights = App\Models\Flight::find([1, 2, 3]);

По желанию можно получить первый результат запроса или выполнить какое-либо другое действие, если результаты не найдены. Метод `firstOr` вернет первый найденный результат или, если результаты не найдены, выполнит замыкание, результат которого будет считаться результатом метода `firstOr`:

    $model = App\Models\Flight::where('legs', '>', 100)->firstOr(function () {
            // ...
    });

Метод `firstOr` также принимает массив столбцов для извлечения:

    $model = App\Models\Flight::where('legs', '>', 100)
                ->firstOr(['id', 'legs'], function () {
                    // ...
                });

<a name="not-found-exceptions"></a>
#### Исключения при отсутствии результатов запроса

По желанию можно выбросить исключение, если модель не найдена. Это особенно полезно в маршрутах или контроллерах. Методы `findOrFail` и `firstOrFail` будут получать первый результат запроса; однако, если результат не найден, будет выброшено исключение `Illuminate\Database\Eloquent\ModelNotFoundException`:

    $model = App\Models\Flight::findOrFail(1);

    $model = App\Models\Flight::where('legs', '>', 100)->firstOrFail();

Если исключение не перехвачено, то пользователю автоматически отправляется HTTP-ответ `404`. Нет необходимости писать явные проверки для возврата ответов `404` при использовании этих методов:

    Route::get('/api/flights/{id}', function ($id) {
        return App\Models\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### Извлечение Агрегатов

Вы также можете использовать `count`,` sum`, `max` и другие [агрегатные методы](queries.md#aggregates), предоставляемые [построителем запросов](queries.md). Эти методы возвращают соответствующее скалярное значение вместо полноценного экземпляра модели:

    $count = App\Models\Flight::where('active', 1)->count();

    $max = App\Models\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Вставка и обновление моделей

<a name="inserts"></a>
### Вставка

Чтобы создать новую запись в базе данных, создайте новый экземпляр модели, установите атрибуты для модели, затем вызовите метод `save`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Flight;
    use Illuminate\Http\Request;

    class FlightController extends Controller
    {
        /**
         * Создать новый экземпляр рейса.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Валидация запроса ...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

В этом примере мы присваиваем параметр `name` из входящего HTTP-запроса атрибуту `name` экземпляра модели `App\Models\Flight`. Когда мы вызываем метод `save`, запись будет вставлена в базу данных. Временные метки `created_at` и `updated_at` будут автоматически установлены при вызове метода `save`, поэтому нет необходимости устанавливать их вручную.

<a name="updates"></a>
### Обновление

Метод `save` также может использоваться для обновления моделей, которые уже существуют в базе данных. Чтобы обновить модель, вы должны извлечь ее, установить любые атрибуты, которые вы хотите обновить, а затем вызвать метод `save`. Опять же, временная метка `updated_at` будет автоматически обновлена, поэтому нет необходимости вручную устанавливать ее значение:

    $flight = App\Models\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

<a name="mass-updates"></a>
#### Массовые обновления

Обновления также могут выполняться для любого количества моделей, соответствующих переданному запросу. В этом примере все рейсы, которые активны и имеют пункт назначения в Сан-Диего, будут помечены как задержанные:

    App\Models\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

Метод `update` ожидает массив пар ключей и значений, представляющих столбцы, которые должны быть обновлены.

> {note} События модели Eloquent `saving`, `saved`, `updating`, и `updated` при массовом обновлении **не будут инициированы** для затронутых моделей. Это связано с тем, что модели фактически никогда не извлекаются при массовом обновлении.

<a name="examining-attribute-changes"></a>
#### Изучение изменений атрибутов

Eloquent предоставляет методы `isDirty`, `isClean` и `wasChanged` для проверки внутреннего состояния модели и определения того, как ее атрибуты изменились с момента их первоначальной загрузки.

Метод `isDirty` определяет, были ли изменены какие-либо атрибуты с момента загрузки модели. Вы можете передать конкретное имя атрибута, чтобы определить, является ли конкретный атрибут «грязным». Метод `isClean` является противоположностью метода `isDirty` и также принимает необязательный аргумент атрибута:

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';

    $user->isDirty(); // true
    $user->isDirty('title'); // true
    $user->isDirty('first_name'); // false

    $user->isClean(); // false
    $user->isClean('title'); // false
    $user->isClean('first_name'); // true

    $user->save();

    $user->isDirty(); // false
    $user->isClean(); // true

Метод `wasChanged` определяет, были ли изменены какие-либо атрибуты при последнем сохранении модели в текущем цикле запроса. Вы также можете передать имя атрибута, чтобы узнать, был ли изменен конкретный атрибут:

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';
    $user->save();

    $user->wasChanged(); // true
    $user->wasChanged('title'); // true
    $user->wasChanged('first_name'); // false

Метод `getOriginal` возвращает массив, содержащий исходные атрибуты модели, независимо от каких-либо изменений с момента загрузки модели. Вы можете передать определенное имя атрибута, чтобы получить исходное значение определенного атрибута:

    $user = User::find(1);

    $user->name; // John
    $user->email; // john@example.com

    $user->name = "Jack";
    $user->name; // Jack

    $user->getOriginal('name'); // John
    $user->getOriginal(); // Массив исходных атрибутов ...

<a name="mass-assignment"></a>
### Массовое присвоение

Вы также можете использовать метод `create` для сохранения новой модели в одну строку. Вставленный экземпляр модели будет возвращен из метода. Однако, перед этим вам нужно будет указать в модели атрибут `fillable` или `guarded`, так как все модели Eloquent по умолчанию защищены от массового присвоения

Уязвимость массового присвоения возникает, когда пользователь передает неожиданный параметр HTTP через запрос, и этот параметр изменяет столбец в базе данных, чего вы никак не ожидали. Например, злоумышленник может отправить параметр `is_admin` через HTTP-запрос, который затем передается в метод `create` модели, позволяющий пользователю повысить свои права до администратора.

Итак, для начала вы должны определить, какие атрибуты модели вы хотите сделать массово-назначаемыми. Вы можете сделать это используя свойство `$fillable` модели. Например, давайте сделаем атрибут `name` нашей модели `Flight` массово-назначаемым:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Атрибуты, для которых разрешено массовое присвоение значений.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Как только мы сделали атрибут массово-назначаемым, мы можем использовать метод `create` для вставки новой записи в базу данных. Метод `create` возвращает сохраненный экземпляр модели:

    $flight = App\Models\Flight::create(['name' => 'Flight 10']);

Если у вас уже есть экземпляр модели, вы можете использовать метод `fill` для заполнения его массивом атрибутов:

    $flight->fill(['name' => 'Flight 22']);

<a name="mass-assignment-json-columns"></a>
#### Массовое присвоение и JSON-столбцы

При назначении JSON-столбцов необходимо указать массово назначаемый ключ для каждого столбца в массиве `$fillable` модели. В целях безопасности Laravel не поддерживает обновление вложенных атрибутов JSON при использовании свойства `guarded`:

    /**
     * Атрибуты, для которых разрешено массовое присвоение значений.
     *
     * @var array
     */
    $fillable = [
        'options->enabled',
    ];

<a name="allowing-mass-assignment"></a>
#### Защита массового присвоения

Если вы хотите сделать все атрибуты массово-назначаемыми, вы можете определить свойство `$guarded` как пустой массив:

    /**
     * Атрибуты, для которых НЕ разрешено массовое присвоение значений.
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### Другие методы создания

<a name="firstorcreate-firstornew"></a>
#### `firstOrCreate`/ `firstOrNew`

Есть два других метода, которые вы можете использовать для создания моделей путем массового присвоения атрибутов: `firstOrCreate` и `firstOrNew`. Метод `firstOrCreate` попытается найти запись в базе данных, используя заданные пары столбец / значение. Если модель не может быть найдена в базе данных, будет вставлена запись с атрибутами из первого параметра вместе с атрибутами из необязательного второго параметра.

Метод `firstOrNew`, подобно `firstOrCreate`, попытается найти в базе данных запись, соответствующую заданным атрибутам. Однако, если модель не найдена, будет возвращен новый экземпляр модели. Обратите внимание, что модель, возвращаемая `firstOrNew`, еще не сохранена в базе данных. Вам нужно будет вызвать `save` самостоятельно, чтобы сохранить её:

    // Получить рейс по `name` или создать его, если он не существует ...
    $flight = App\Models\Flight::firstOrCreate(['name' => 'Flight 10']);

    // Получить рейс по `name` или создать его с атрибутами `name`, `delayed`, и `arrival_time` ...
    $flight = App\Models\Flight::firstOrCreate(
        ['name' => 'Flight 10'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

    // Получить по `name` или создать экземпляр ...
    $flight = App\Models\Flight::firstOrNew(['name' => 'Flight 10']);

    // Получить по `name` или создать экземпляр с атрибутами `name`, `delayed`, и `arrival_time` ...
    $flight = App\Models\Flight::firstOrNew(
        ['name' => 'Flight 10'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

<a name="updateorcreate"></a>
#### `updateOrCreate`

Вы также можете столкнуться с ситуациями, когда вы хотите обновить существующую модель или создать новую модель, если таковой не существует. Laravel предоставляет метод `updateOrCreate`, чтобы сделать это за один шаг. Как и метод `firstOrCreate`, `updateOrCreate` сохраняет модель, поэтому нет необходимости вызывать `save()`:

    // Если есть рейс из Окленда в Сан-Диего, установить цену в $99 ...
    // Если подходящей модели не существует, создать ее ...
    $flight = App\Models\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99, 'discounted' => 1]
    );

Если вы хотите выполнить несколько «обновлений-вставок» в одном запросе, вам следует использовать вместо этого метод `upsert`. Первый аргумент метода состоит из значений для вставки или обновления, а второй аргумент перечисляет столбцы, которые однозначно идентифицируют записи в связанной таблице. Третий и последний аргументы метода – это массив столбцов, которые следует обновить, если соответствующая запись уже существует в базе данных. Метод `upsert` автоматически устанавливает временные метки `created_at` и `updated_at`, если они разрешены в модели:

    App\Models\Flight::upsert([
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ], ['departure', 'destination'], ['price']);

> {note} Все базы данных, кроме SQL Server, требуют, чтобы столбцы во втором аргументе метода `upsert` имели «первичный» или «уникальный» индекс.

<a name="deleting-models"></a>
## Удаление моделей

Чтобы удалить модель, вызовите метод `delete` экземпляра модели:

    $flight = App\Models\Flight::find(1);

    $flight->delete();

<a name="deleting-an-existing-model-by-key"></a>
#### Удаление существующей модели по ключу

В приведенном выше примере мы извлекаем модель из базы данных перед вызовом метода `delete`. Однако если вы знаете первичный ключ модели, вы можете удалить модель, не извлекая ее, вызвав метод `destroy`. В дополнение к одному первичному ключу в качестве аргумента, метод `destroy` может принимать несколько первичных ключей, массив первичных ключей или [коллекцию](collections.md) первичных ключей:

    App\Models\Flight::destroy(1);

    App\Models\Flight::destroy(1, 2, 3);

    App\Models\Flight::destroy([1, 2, 3]);

    App\Models\Flight::destroy(collect([1, 2, 3]));

> {note} Метод `destroy` загружает каждую модель отдельно и вызывает для них метод `delete`, чтобы сработали события `deleting` и `deleted`.

<a name="deleting-models-by-query"></a>
#### Удаление моделей через запрос

Вы также можете запустить оператор удаления для набора моделей. В этом примере мы удалим все рейсы, помеченные как неактивные. Как и массовые обновления, массовые удаления не вызывают никаких событий модели для удаляемых моделей:

    $deletedRows = App\Models\Flight::where('active', 0)->delete();

> {note} События модели Eloquent `deleting`, и `deleted` при массовом удалении не будут инициированы для удаленных моделей. Это связано с тем, что модели фактически не извлекаются при выполнении оператора `delete`.

<a name="soft-deleting"></a>
### Программное удаление

Помимо фактического удаления записей из базы данных, Eloquent может также «программно удалять» модели. При таком удалении, модели фактически не удаляются из базы данных. Вместо этого для модели устанавливается атрибут `deleted_at` и вставляется в базу данных. Если модель имеет _non-null_ значение `deleted_at`, значит она была программно удалена. Чтобы включить программное удаление для модели, используйте трейт `Illuminate\Database\Eloquent\SoftDeletes`

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;
    }

> {tip} Трейт `SoftDeletes` автоматически типизирует атрибут `deleted_at` к экземпляру `DateTime` / `Carbon`.

Вам также следует добавить столбец `deleted_at` в таблицу базы данных. [Построитель схемы](migrations.md) Laravel содержит метод для создания этого столбца:

    public function up()
    {
        Schema::table('flights', function (Blueprint $table) {
            $table->softDeletes();
        });
    }

    public function down()
    {
        Schema::table('flights', function (Blueprint $table) {
            $table->dropSoftDeletes();
        });
    }

Теперь, когда вы вызываете метод `delete` модели, в столбце `deleted_at` будут установлены текущие дата и время. При запросе модели, которая использует программное удаление, программно удаленные модели будут автоматически исключены из всех результатов запроса.

Чтобы определить, был ли данный экземпляр модели программно удален, используйте метод `trashed`:

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### Запросы для моделей, использующих программное удаление

<a name="including-soft-deleted-models"></a>
#### Включение программно удаленных моделей

Как отмечалось выше, программно удаленные модели будут автоматически исключены из результатов запроса. Однако, вы можете принудительно отобразить такие модели в результирующем наборе, используя метод `withTrashed` в запросе:

    $flights = App\Models\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

Метод `withTrashed` также может использоваться в запросе, использующем [отношения](eloquent-relationships.md):

    $flight->history()->withTrashed()->get();

<a name="retrieving-only-soft-deleted-models"></a>
#### Извлечение только программно удаленных моделей

Метод `onlyTrashed` будет извлекать **только** программно удаленные модели:

    $flights = App\Models\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

<a name="restoring-soft-deleted-models"></a>
#### Восстановление программно удаленных моделей

По желанию можно «восстановить» программно удаленную модель. Чтобы восстановить такую модель в активное состояние, используйте метод `restore` экземпляра модели:

    $flight->restore();

Вы также можете использовать метод `restore` в запросе для быстрого восстановления нескольких моделей. Опять же, как и другие «массовые» операции, это не вызовет никаких событий модели для восстанавливаемых моделей:

    App\Models\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Как и метод `withTrashed`, метод `restore` используется и в [отношениях](eloquent-relationships.md):

    $flight->history()->restore();

<a name="permanently-deleting-models"></a>
#### Удаление моделей без возможности восстановления

По желанию можно действительно удалить модель из вашей базы данных. Чтобы окончательно удалить программно удаленную модель из базы данных, используйте метод `forceDelete`:

    // Принудительно удалить один экземпляр модели ...
    $flight->forceDelete();

    // Принудительно удалить все связанные модели ...
    $flight->history()->forceDelete();

<a name="replicating-models"></a>
## Репликация (тиражирование) моделей

Вы можете создать несохраненную копию экземпляра модели, используя метод `replicate`. Это особенно полезно, когда у вас есть экземпляры модели, которые имеют много одинаковых атрибутов:

    $shipping = App\Models\Address::create([
        'type' => 'shipping',
        'line_1' => '123 Example Street',
        'city' => 'Victorville',
        'state' => 'CA',
        'postcode' => '90001',
    ]);

    $billing = $shipping->replicate()->fill([
        'type' => 'billing'
    ]);

    $billing->save();

<a name="query-scopes"></a>
## Диапазоны запроса

<a name="global-scopes"></a>
### Глобальные диапазоны

Глобальные диапазоны позволяют добавлять ограничения ко всем запросам для данной модели. [Программное удаление](#soft-deleting) в Laravel использует глобальные диапазоны для извлечения только «не удаленных» моделей из базы данных. Написание пользовательских глобальных диапазонов предоставляют удобный и простой способ, гарантирующий, что в каждом запросе конкретной модели будут применены определенные ограничения.

<a name="writing-global-scopes"></a>
#### Написание глобальных диапазонов

Написание глобального диапазона — это просто. Определите класс, который реализует интерфейс `Illuminate\Database\Eloquent\Scope`. Этот интерфейс требует от вас реализации одного метода: `apply`. Метод `apply` может при необходимости добавлять ограничения `where` к запросу:

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Scope;

    class AgeScope implements Scope
    {
        /**
         * Применить диапазон к переданному построителю запросов.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }

> {tip} Если диапазон добавляет столбцы в конструкцию Select-запроса, вы должны использовать метод `addSelect` вместо `select`. Это предотвратит непреднамеренную замену существующих Select-конструкций в запросе.

<a name="applying-global-scopes"></a>
#### Применение глобальных диапазонов

Чтобы назначить глобальный диапазон модели, вы должны переопределить метод `booted` модели и использовать метод `addGlobalScope`:

    <?php

    namespace App\Models;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Метод «booted» модели.
         *
         * @return void
         */
        protected static function booted()
        {
            static::addGlobalScope(new AgeScope);
        }
    }

После добавления диапазона, запрос `User::all()` выдаст следующий SQL:

    select * from `users` where `age` > 200

<a name="anonymous-global-scopes"></a>
#### Анонимные глобальные диапазоны

Eloquent также позволяет определять глобальные диапазоны с использованием замыкания, что особенно полезно для простейших диапазонов, которые не требуют отдельного класса:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Метод «booted» модели.
         *
         * @return void
         */
        protected static function booted()
        {
            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

<a name="removing-global-scopes"></a>
#### Игнорирование глобальных диапазонов

Для исключения глобального диапазона в текущем запросе, используйте метод `withoutGlobalScope`. Метод принимает имя класса глобального диапазона в качестве единственного аргумента:

    User::withoutGlobalScope(AgeScope::class)->get();

Или, если вы определили глобальный диапазон с помощью замыкания:

    User::withoutGlobalScope('age')->get();

Если вы хотите удалить несколько или даже все глобальные диапазоны, вы можете использовать метод `withoutGlobalScopes`:

    // Игнорировать все глобальные диапазоны ...
    User::withoutGlobalScopes()->get();

    // Игнорировать некоторые глобальные диапазоны ...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### Локальные диапазоны

Локальные диапазоны позволяют определять общие наборы ограничений, которые вы можете легко использовать повторно приложении. Например, можно получить всех пользователей, которые считаются «популярными». Чтобы определить диапазон, добавьте префикс `scope` к методу модели Eloquent.

Диапазоны должны всегда возвращать экземпляр построителя запросов:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Диапазон запроса, включающий только популярных пользователей.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * Диапазон запроса, включающий только активных пользователей.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

<a name="utilizing-a-local-scope"></a>
#### Использование локальных диапазонов

После определения диапазона можно вызвать метод при выполнении запроса модели. Однако вы не должны включать префикс `scope` при вызове метода. Вы можете даже объединять вызовы в различные диапазоны, например:

    $users = App\Models\User::popular()->active()->orderBy('created_at')->get();

Объединение нескольких диапазонов модели Eloquent с помощью оператора запроса `or` может потребовать использования замыканий:

    $users = App\Models\User::popular()->orWhere(function (Builder $query) {
        $query->active();
    })->get();

Поскольку это может быть громоздким, Laravel предоставляет метод `orWhere` «более высокого порядка», который позволяет вам свободно объединять эти диапазоны вместе без использования замыканий:

    $users = App\Models\User::popular()->orWhere->active()->get();

<a name="dynamic-scopes"></a>
#### Динамические диапазоны

По желанию можно определить диапазон, который принимает параметры. Просто добавьте дополнительные параметры в диапазон. Параметры диапазона должны быть определены после параметра `$query`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Диапазон запроса, включающий пользователей только определенного типа.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $query
         * @param  mixed  $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

Теперь вы можете передать параметры при вызове диапазона:

    $users = App\Models\User::ofType('admin')->get();

<a name="comparing-models"></a>
## Сравнение моделей

По желанию можно определить, являются ли две модели «одинаковыми». Метод `is` используется для быстрой проверки того, что две модели имеют одинаковый первичный ключ, таблицу и соединение с базой данных:

    if ($post->is($anotherPost)) {
        //
    }

Метод `is` также доступен при использовании отношений `belongsTo`, `hasOne`, `morphTo`, и `morphOne`. Этот метод особенно полезен, если вы хотите сравнить связанную модель без запроса на получение этой модели:

    if ($post->author()->is($user)) {
        //
    }

<a name="events"></a>
## События

Модели Eloquent инициируют некоторые события, что позволяет использовать следующие хуки жизненного цикла модели: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`, `replicating`. События позволяют легко выполнять код каждый раз, когда определенный класс модели сохраняется или обновляется в базе данных. Каждое событие получает экземпляр модели через свой конструктор.

Событие `retrieved` сработает, когда существующая модель будет извлечена из базы данных. Когда новая модель сохраняется в первый раз, инициируются события `creating` и `created`. События `update` / `updated` будут выполняться при изменении существующей модели и вызове метода `save`. События `saving` / `saved` будут выполняться при создании или обновлении модели.

> {note} События модели Eloquent `saved`, `updated`, `deleting`, и `deleted` при массовом обновлении или удалении **не будут инициированы** для затронутых моделей. Это связано с тем, что модели фактически не извлекаются при массовом обновлении или удалении.

Для начала, определите свойство `$dispatchesEvents` в модели Eloquent, которое сопоставляет различные хуки жизненного цикла модели Eloquent с пользовательскими [классами событий](events.md):

    <?php

    namespace App\Models;

    use App\Events\UserDeleted;
    use App\Events\UserSaved;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Карта событий для модели.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

После определения и сопоставления событий вы можете использовать [слушателей событий](events.md#defining-listeners) для их обработки.

<a name="events-using-closures"></a>
### Использование замыканий

Вместо использования пользовательских классов событий можно регистрировать замыкания, которые выполняются при инициировании различных событий модели. Как правило, вы должны зарегистрировать эти замыкания в методе `booted` модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Метод «booted» модели.
         *
         * @return void
         */
        protected static function booted()
        {
            static::created(function ($user) {
                //
            });
        }
    }

При необходимости вы можете использовать [последовательных анонимных слушателей событий](events.md#queuable-anonymous-event-listeners) при регистрации событий модели. Это проинструктирует Laravel выполнить слушателя событий модели, используя [очередь](queues.md):

    use function Illuminate\Events\queueable;

    static::created(queueable(function ($user) {
        //
    }));

<a name="observers"></a>
### Наблюдатели

<a name="defining-observers"></a>
#### Определение наблюдателей

Если прослушивается множество событий в модели, то можно использовать наблюдателей, чтобы сгруппировать пользовательских слушателей в одном классе. Классы наблюдателей имеют имена методов, созвучные событиям Eloquent, которые необходимо прослушивать. Каждый из этих методов получает модель в качестве единственного аргумента. Команда Artisan `make:observer` — это самый простой способ создать новый класс наблюдателя:

    php artisan make:observer UserObserver --model=User

Данная команда разместит нового наблюдателя в каталоге `App/Observers`. Если каталог не существует, то Artisan создаст ее. Созданный наблюдатель может выглядеть следующим образом:

    <?php

    namespace App\Observers;

    use App\Models\User;

    class UserObserver
    {
        /**
         * Обработать событие «created» модели User.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * Обработать событие «updated» модели User.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function updated(User $user)
        {
            //
        }

        /**
         * Обработать событие «deleted» модели User.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function deleted(User $user)
        {
            //
        }

        /**
         * Обработать событие «forceDeleted» модели User.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function forceDeleted(User $user)
        {
            //
        }
    }

Чтобы зарегистрировать наблюдателя, используйте метод `observe` наблюдаемой модели. Зарегистрировать наблюдателей можно в методе `boot` одного из поставщиков служб. В этом примере мы зарегистрируем наблюдателя в `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use App\Observers\UserObserver;
    use App\Models\User;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
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
            User::observe(UserObserver::class);
        }
    }

<a name="muting-events"></a>
### Подавление событий

По желанию можно временно «заглушить» все события, запускаемые моделью. Вы можете добиться этого, используя метод `withoutEvents`. Метод `withoutEvents` принимает замыкание как единственный аргумент. Любой код, выполняемый в этом замыкании, не будет запускать события модели. Например, следующее извлечет и удалит экземпляр `App\User`, не вызывая никаких событий модели. Любое значение, возвращаемое переданным замыканием, будет возвращено методом `withoutEvents`:

    use App\Models\User;

    $user = User::withoutEvents(function () use () {
        User::findOrFail(1)->delete();

        return User::find(2);
    });

<a name="saving-a-single-model-without-events"></a>
#### Тихое сохранение одной модели

По желанию можно «сохранить» конкретную модель, не вызывая никаких событий. Вы можете сделать это с помощью метода `saveQuietly`:

    $user = User::findOrFail(1);

    $user->name = 'Victoria Faith';

    $user->saveQuietly();
