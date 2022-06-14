# Laravel 9 · Eloquent · Мутаторы и типизация

- [Введение](#introduction)
- [Аксессоры и мутаторы](#accessors-and-mutators)
    - [Определение аксессора](#defining-an-accessor)
    - [Определение мутатора](#defining-a-mutator)
- [Приведение атрибутов к типам](#attribute-casting)
    - [Преобразование в массив и JSON](#array-and-json-casting)
    - [Типизация даты](#date-casting)
    - [Типизация `Enum`](#enum-casting)
    - [Шифрованная типизация](#encrypted-casting)
    - [Типизация во время запроса](#query-time-casting)
- [Пользовательская типизация](#custom-casts)
    - [Типизация объект-значение](#value-object-casting)
    - [Сериализация в массив и JSON](#array-json-serialization)
    - [Входящая типизация](#inbound-casting)
    - [Параметры типизации](#cast-parameters)
    - [Интерфейс `Castable`](#castables)

<a name="introduction"></a>
## Введение

Аксессоры, мутаторы и приведение атрибутов к типам позволяют преобразовывать значения атрибутов Eloquent, когда вы извлекаете экземпляр модели или присваиваете их экземпляру модели. Например, вы можете использовать [шифровальщик Laravel](encryption.md), чтобы зашифровать значение при его сохранении в базу данных, а затем автоматически расшифровать атрибут при доступе к нему в модели Eloquent. Или вы можете преобразовать строку JSON, которая хранится в вашей базе данных, в массив при доступе к ней через вашу модель Eloquent.

<a name="accessors-and-mutators"></a>
## Аксессоры и мутаторы

<a name="defining-an-accessor"></a>
### Определение аксессора

Аксессор преобразует значение атрибута экземпляра Eloquent при обращении к нему. Чтобы определить метод доступа, создайте в модели защищенный метод для представления доступного атрибута. Это имя метода должно соответствовать атрибуту модели / столбца базы данных в «верблюжьем регистре» , когда это применимо.

В этом примере мы определим аксессор для атрибута `first_name`. Аксессор будет автоматически вызван Eloquent при попытке получить значение атрибута `first_name`. Все методы атрибутов (аксессоры / мутаторы) должны возвращать тип `Illuminate\Database\Eloquent\Casts\Attribute`:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Взаимодействие с именем пользователя.
         *
         * @return \Illuminate\Database\Eloquent\Casts\Attribute
         */
        protected function firstName(): Attribute
        {
            return Attribute::make(
                get: fn ($value) => ucfirst($value),
            );
        }
    }

Все методы доступа возвращают экземпляр `Attribute`, который определяет, как будет осуществляться доступ к атрибуту и, при необходимости, изменяться. В этом примере мы только определяем, как будет осуществляться доступ к атрибуту. Для этого мы передаем аргумент `get` конструктору класса `Attribute`.

Как видите, исходное значение столбца передается аксессору, что позволяет вам манипулировать и возвращать значение. Чтобы получить доступ к значению аксессора, вы можете просто получить доступ к атрибуту `first_name` экземпляра модели:

    use App\Models\User;

    $user = User::find(1);

    $firstName = $user->first_name;

> {tip} Если вы хотите, чтобы эти вычисленные значения были добавлены к представлениям массива / JSON вашей модели, [вам нужно будет добавить их](eloquent-serialization.md#appending-values-to-json).

<a name="building-value-objects-from-multiple-attributes"></a>
#### Построение объекта-значения из нескольких атрибутов

Иногда аксессору может потребоваться преобразовать несколько атрибутов модели в один «объект-значение». Для этого ваше замыкание `get` может принимать второй аргумент `$attributes`, который будет автоматически передан замыканию и будет содержать массив всех текущих атрибутов модели:

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Взаимодействие с адресом пользователя.
 *
 * @return  \Illuminate\Database\Eloquent\Casts\Attribute
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn ($value, $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}
```

<a name="accessor-caching"></a>
#### Кэширование аксессора

При возврате объектов-значений из аксессоров любые изменения, внесенные в объект-значение, будут автоматически синхронизированы с моделью перед ее сохранением. Это возможно, потому что Eloquent сохраняет экземпляры, возвращаемые аксессорами, поэтому он может возвращать один и тот же экземпляр каждый раз, когда вызывается аксессор:

    use App\Models\User;

    $user = User::find(1);

    $user->address->lineOne = 'Updated Address Line 1 Value';
    $user->address->lineTwo = 'Updated Address Line 2 Value';

    $user->save();

Иногда требуется включить кэширование для примитивных значений, таких как строки и логические значения, особенно если они требуют больших вычислительных ресурсов. Для этого вы можете вызвать метод `shouldCache` при определении вашего аксессора:

```php
protected function hash(): Attribute
{
    return Attribute::make(
        get: fn ($value) => bcrypt(gzuncompress($value)),
    )->shouldCache();
}
```

Если вы хотите избежать кэширования объектов в атрибутах, то вы можете вызвать метод `withoutObjectCaching` при определении атрибута:

```php
/**
 * Взаимодействие с адресом пользователя.
 *
 * @return  \Illuminate\Database\Eloquent\Casts\Attribute
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn ($value, $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    )->withoutObjectCaching();
}
```

<a name="defining-a-mutator"></a>
### Определение мутатора

Мутатор преобразует значение атрибута в момент их присвоения экземпляру Eloquent. Чтобы определить мутатор, вы можете указать аргумент `set` при определении вашего атрибута. Определим мутатор для атрибута `first_name`. Этот мутатор будет автоматически вызываться, когда мы попытаемся присвоить значение атрибута `first_name` модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Взаимодействие с именем пользователя.
         *
         *
         * @return \Illuminate\Database\Eloquent\Casts\Attribute
         */
        protected function firstName(): Attribute
        {
            return Attribute::make(
                get: fn ($value) => ucfirst($value),
                set: fn ($value) => strtolower($value),
            );
        }
    }

Замыкание мутатора получит значение, которое устанавливается для атрибута, позволяя вам манипулировать значением и возвращать измененное значение. Чтобы использовать наш мутатор, нам нужно только установить атрибут `first_name` для модели Eloquent:

    use App\Models\User;

    $user = User::find(1);

    $user->first_name = 'Sally';

В этом примере замыкание `set` будет вызвано со значением `Sally`. Затем мутатор применит к имени функцию `strtolower` и установит полученное значение во внутреннем массиве `$attributes` модели.

<a name="mutating-multiple-attributes"></a>
#### Преобразование нескольких атрибутов

Иногда мутатору может потребоваться установить несколько атрибутов модели. Для этого вы можете вернуть массив из замыкания `set`. Каждый ключ в массиве должен соответствовать атрибуту / столбцу базы данных, связанному с моделью:

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Взаимодействие с адресом пользователя.
 *
 * @return  \Illuminate\Database\Eloquent\Casts\Attribute
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn ($value, $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```

<a name="attribute-casting"></a>
## Приведение атрибутов к типам

Приведение атрибутов обеспечивает функциональность, аналогичную аксессорам и мутаторам, но без необходимости определения каких-либо дополнительных методов вашей модели. Вместо этого свойство `$casts` вашей модели представляет удобный способ преобразования атрибутов в распространенные типы данных.

Свойство `$casts` должно быть массивом, где ключ – это имя преобразуемого атрибута, а значение – это тип, к которому вы хотите привести столбец. Поддерживаемые типы преобразования:

<!-- <div class="content-list" markdown="1"> -->

- `array`
- `AsStringable::class`
- `boolean`
- `collection`
- `date`
- `datetime`
- `immutable_date`
- `immutable_datetime`
- `decimal:`<code>&lt;digits&gt;</code>
- `double`
- `encrypted`
- `encrypted:array`
- `encrypted:collection`
- `encrypted:object`
- `float`
- `integer`
- `object`
- `real`
- `string`
- `timestamp`

<!-- </div> -->

Чтобы продемонстрировать преобразование атрибутов, давайте преобразуем атрибут `is_admin`, который хранится в нашей базе данных в виде целого числа (`0` или `1`), в логическое значение:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть типизированы.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

После определения типизации, атрибут `is_admin` всегда будет преобразован в логическое значение при доступе к нему, даже если базовое значение хранится в базе данных как целое число:

    $user = App\Models\User::find(1);

    if ($user->is_admin) {
        //
    }

Если вам нужно добавить новую временную типизацию во время выполнения, то вы можете использовать метод `mergeCasts`. Эти определения типизации будут добавлены к набору уже определенных для модели типизаций:

    $user->mergeCasts([
        'is_admin' => 'integer',
        'options' => 'object',
    ]);

> {note} Атрибуты, которые имеют значение `null`, не будут преобразованы. Кроме того, вы никогда не должны определять типизацию (или атрибут), имя которого совпадает с именем отношения.

<a name="stringable-casting"></a>
#### Строковая типизация

Вы можете использовать класс `Illuminate\Database\Eloquent\Casts\AsStringable` для приведения атрибута модели к объекту [класса `Illuminate\Support\Stringable`](helpers.md#fluent-strings-method-list):

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\AsStringable;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть типизированы.
         *
         * @var array
         */
        protected $casts = [
            'directory' => AsStringable::class,
        ];
    }

<a name="array-and-json-casting"></a>
### Преобразование в массив и JSON

Преобразование в `array` особенно полезно при работе со столбцами, которые хранятся как сериализованный JSON. Например, если ваша база данных имеет поле типа `JSON` или `TEXT`, содержащее сериализованный JSON, то добавленная типизация `array` этому атрибуту автоматически десериализует атрибут модели Eloquent в массив PHP при обращении к нему:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть типизированы.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

Как только типизация определена, вы можете получить доступ к атрибуту `options`, и он будет автоматически десериализован из JSON в массив PHP. Когда вы устанавливаете значение атрибута `options`, данный массив будет автоматически сериализован обратно в JSON для сохранения:

    use App\Models\User;

    $user = User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();

Чтобы обновить одно поле JSON-атрибута с помощью краткого синтаксиса, используйте оператор `->` при вызове метода `update`:

    $user = User::find(1);

    $user->update(['options->key' => 'value']);

<a name="array-object-and-collection-casting"></a>
#### Типизация ArrayObject и Collection

Хотя типизации стандартного `array` достаточно для многих приложений, но у него есть некоторые недостатки. Поскольку типизация `array` возвращает примитивный тип, невозможно напрямую изменить смещение массива. Например, следующий код вызовет ошибку PHP:

    $user = User::find(1);

    $user->options['key'] = $value;

Чтобы решить эту проблему, Laravel предлагает типизацию `AsArrayObject`, которая преобразует ваш атрибут JSON в класс [ArrayObject](https://www.php.net/manual/ru/class.arrayobject.php). Эта функция реализована с использованием реализации [пользовательской типизации](#custom-casts) Laravel, которая позволяет Laravel интеллектуально кешировать и преобразовывать измененный объект таким образом, что отдельные смещения могли быть изменены без ошибок PHP. Чтобы использовать типизацию `AsArrayObject`, просто назначьте его атрибуту:

    use Illuminate\Database\Eloquent\Casts\AsArrayObject;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'options' => AsArrayObject::class,
    ];

Точно так же Laravel предлагает типизацию `AsCollection`, которая преобразует ваш атрибут JSON в экземпляр Laravel [Collection](collections.md):

    use Illuminate\Database\Eloquent\Casts\AsCollection;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'options' => AsCollection::class,
    ];

<a name="date-casting"></a>
### Типизация даты

По умолчанию Eloquent преобразует столбцы `created_at` и `updated_at` в экземпляры [Carbon](https://github.com/briannesbitt/Carbon), расширяющего класс DateTime PHP и предоставляющего набор полезных методов. Вы можете типизировать дополнительные атрибуты даты, определив дополнительные преобразования даты в массиве свойств `$casts` вашей модели. Обычно даты следует типизировать с использованием типов `datetime` или `immutable_datetime`.

При определении типизации `date` или `datetime` вы также можете указать формат даты. Этот формат будет использоваться, когда [модель сериализуется в массив или JSON](eloquent-serialization.md):

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];

Когда столбец типизирован как дата, вы можете установить соответствующее значение атрибута модели в виде временной метки форматов UNIX, строки даты (`Y-m-d`), строки даты-времени или экземпляров `DateTime` / `Carbon`. Значение даты будет правильно преобразовано и сохранено в вашей базе данных.

Вы можете настроить формат сериализации по умолчанию для всех дат вашей модели, переопределив метод `serializeDate` вашей модели. Этот метод не влияет на форматирование дат для их сохранения в базе данных:

    /**
     * Подготовить дату для сериализации массива / JSON.
     *
     * @param  \DateTimeInterface  $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date)
    {
        return $date->format('Y-m-d');
    }

Чтобы указать формат, который следует использовать при фактическом сохранении дат модели в вашей базе данных, вы должны определить свойство `$dateFormat` вашей модели:

    /**
     * Формат хранения столбцов даты модели.
     *
     * @var string
     */
    protected $dateFormat = 'U';

<a name="date-casting-and-timezones"></a>
#### Типизация даты, сериализация и часовые пояса

По умолчанию типизации `date` и `datetime` будут сериализовать даты в строковый формат UTC ISO-8601 (`1986-05-28T21:05:54.000000Z`) даты, независимо от часового пояса, указанного в параметре `timezone` конфигурации вашего приложения. Вам настоятельно рекомендуется всегда использовать этот формат сериализации, а также сохранять даты вашего приложения в часовом поясе UTC, не изменяя параметр конфигурации вашего приложения `timezone` по умолчанию, имеющий значение `UTC`. Последовательное использование часового пояса UTC во всем приложении обеспечит максимальный уровень взаимодействия с другими библиотеками обработки даты, написанными на PHP и JavaScript.

Если типизации `date` или `datetime` дополнены пользовательским форматом, например, `datetime:Y-m-d H:i:s`, то во время сериализации даты будет использован внутренний часовой пояс экземпляра Carbon. Обычно это часовой пояс, указанный в параметре `timezone` конфигурации вашего приложения.

<a name="enum-casting"></a>
### Типизация `Enum`

> {note} Перечисляемые типы доступны только в [PHP 8.1+](https://www.php.net/manual/ru/language.enumerations.php).

Eloquent также позволяет вам преобразовывать значения ваших атрибутов в [типизированные перечисления](https://www.php.net/manual/ru/language.enumerations.backed.php) PHP. Для этого вы можете указать атрибут, который вы хотите типизировать, и соответствующий класс перечисления в массиве `$casts` вашей модели:

    use App\Enums\ServerStatus;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'status' => ServerStatus::class,
    ];

После того, как вы определили типизацию в своей модели, указанный атрибут будет автоматически преобразован в перечисляемый тип и из него при взаимодействии с атрибутом:

    if ($server->status == ServerStatus::provisioned) {
        $server->status = ServerStatus::ready;

        $server->save();
    }

<a name="encrypted-casting"></a>
### Шифрованная типизация

Типизация `encrypted` зашифрует значение атрибута модели, используя встроенный в Laravel функционал [шифрования](encryption.md). Кроме того, типизации `encrypted:array`, `encrypted:collection`, `encrypted:object`, `AsEncryptedArrayObject` и `AsEncryptedCollection` работают так же, как и их обычные аналоги; однако, базовое значение будет зашифровано при сохранении в вашей базе данных.

Поскольку окончательная длина зашифрованного текста непредсказуема и больше, чем его копия в виде простого текста, убедитесь, что соответствующий столбец в базе данных имеет тип `TEXT` или больше. Кроме того, поскольку значения будут зашифрованы в базе данных, вы не сможете выполнять запросы или искать зашифрованные значения атрибутов.

<a name="key-rotation"></a>
#### Смена ключа приложения

Laravel шифрует строки, используя значение конфигурации `key`, указанное в конфигурационном файле `app` вашего приложения. Как правило, это значение соответствует значению переменной окружения `APP_KEY`. Если вам нужно сменить ключ шифрования вашего приложения, то вам нужно будет вручную повторно зашифровать ваши зашифрованные атрибуты с помощью нового ключа.

<a name="query-time-casting"></a>
### Типизация во время запроса

Иногда требуется применить типизацию при выполнении запроса, например, при выборе сырого значения из таблицы. Например, рассмотрим следующий запрос:

    use App\Models\Post;
    use App\Models\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

Атрибут `last_posted_at` результатов этого запроса будет простой строкой. Было бы замечательно, если бы мы могли применить типизацию `datetime` этого атрибута при выполнении запроса. К счастью, мы можем добиться этого с помощью метода `withCasts`:

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'datetime'
    ])->get();

<a name="custom-casts"></a>
## Пользовательская типизация

В Laravel есть множество встроенных полезных преобразователей; однако иногда требуется определить свои собственные. Вы можете добиться этого, определив класс, реализующий интерфейс `CastsAttributes`.

Классы, реализующие этот интерфейс, должны определять методы `get` и `set`. Метод `get` отвечает за преобразование сырого значения из базы данных к типизированному значению, а метод `set` – должен преобразовывать типизированное значение в сырое значение, которое можно сохранить в базе данных. В качестве примера мы повторно реализуем встроенный преобразователь `json` как пользовательский типизатор:

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Json implements CastsAttributes
    {
        /**
         * Преобразовать значение к пользовательскому типу.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return array
         */
        public function get($model, $key, $value, $attributes)
        {
            return json_decode($value, true);
        }

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return json_encode($value);
        }
    }

После того, как вы определили собственный типизатор, вы можете добавить его к атрибуту модели, используя его имя класса:

    <?php

    namespace App\Models;

    use App\Casts\Json;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть типизированы.
         *
         * @var array
         */
        protected $casts = [
            'options' => Json::class,
        ];
    }

<a name="value-object-casting"></a>
### Типизация объект-значение

Вы не ограничены приведением значений к примитивным типам. Вы также можете преобразовать значения к объектам. Определение пользовательских типизаторов, которые преобразуют значения в объекты, очень похоже на приведение к примитивным типам; однако метод `set` должен возвращать массив пар ключ / значение, который будет использоваться для установки сырых значений, сохраняемых в модели.

В качестве примера мы определим собственный класс типизатора, который преобразует несколько значений модели в один объект-значение `Address`. Предположим, что значение `Address` имеет два общедоступных свойства: `lineOne` и `lineTwo`:

    <?php

    namespace App\Casts;

    use App\Models\Address as AddressModel;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
    use InvalidArgumentException;

    class Address implements CastsAttributes
    {
        /**
         * Преобразовать значение к пользовательскому типу.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return \App\Models\Address
         */
        public function get($model, $key, $value, $attributes)
        {
            return new AddressModel(
                $attributes['address_line_one'],
                $attributes['address_line_two']
            );
        }

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  \App\Models\Address  $value
         * @param  array  $attributes
         * @return array
         */
        public function set($model, $key, $value, $attributes)
        {
            if (! $value instanceof AddressModel) {
                throw new InvalidArgumentException('The given value is not an Address instance.');
            }

            return [
                'address_line_one' => $value->lineOne,
                'address_line_two' => $value->lineTwo,
            ];
        }
    }

При приведении к объектам-значениям любые изменения, внесенные в объект-значения, будут автоматически синхронизированы с моделью до ее сохранения:

    use App\Models\User;

    $user = User::find(1);

    $user->address->lineOne = 'Updated Address Value';

    $user->save();

> {tip} Если вы планируете сериализовать свои модели Eloquent, содержащие объекты-значения, в JSON или массивы, вам следует реализовать интерфейсы `Illuminate\Contracts\Support\Arrayable` и `JsonSerializable` для объекта-значения.

<a name="array-json-serialization"></a>
### Сериализация в массив и JSON

Когда модель Eloquent преобразуется в массив или JSON с использованием методов `toArray` и `toJson`, ваши пользовательские типизаторы объекты-значения обычно будут сериализованы, в частности, пока они (типизаторы) реализуют интерфейсы `Illuminate\Contracts\Support\Arrayable` и `JsonSerializable`. Однако при использовании объектов-значений, предоставляемых сторонними библиотеками, у вас может не быть возможности добавить эти интерфейсы к объекту.

Поэтому вы можете указать, что ваш собственный класс типизатора будет отвечать за сериализацию объекта-значения. Для этого ваш собственный класс типизатора должно реализовывать интерфейс `Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes`. В этом интерфейсе указано, что ваш класс должен содержать метод `serialize`, возвращающий сериализованную форму вашего объекта значения:

    /**
     * Получить сериализованное представление значения.
     *
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @param  string  $key
     * @param  mixed  $value
     * @param  array  $attributes
     * @return mixed
     */
    public function serialize($model, string $key, $value, array $attributes)
    {
        return (string) $value;
    }

<a name="inbound-casting"></a>
### Входящая типизация

Иногда требуется написать свой типизатор, который только преобразует указанные значения атрибутов модели, и не выполняет никаких операций при обращении к этим атрибутам. Классическим примером только входящей типизации является «хеширование». Пользовательские типизаторы только для входящих значений должны реализовывать интерфейс `CastsInboundAttributes`, требующий определение метода `set`.

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;

    class Hash implements CastsInboundAttributes
    {
        /**
         * Алгоритм хеширования.
         *
         * @var string
         */
        protected $algorithm;

        /**
         * Создать новый экземпляр класса типизации.
         *
         * @param  string|null  $algorithm
         * @return void
         */
        public function __construct($algorithm = null)
        {
            $this->algorithm = $algorithm;
        }

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return is_null($this->algorithm)
                        ? bcrypt($value)
                        : hash($this->algorithm, $value);
        }
    }

<a name="cast-parameters"></a>
### Параметры типизации

При добавлении пользовательского типизатора к модели, параметры типизатора задаются отделением их от имени класса с помощью символа `:` и разделением нескольких параметров запятыми. Параметры будут переданы в конструктор класса типизатора:

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'secret' => Hash::class.':sha256',
    ];

<a name="castables"></a>
### Интерфейс `Castable`

Вы можете разрешить объектам-значениям вашего приложения определять свои собственные классы типизаторы. Вместо указания пользовательской типизации в модели, вы можете альтернативно указать класс, который реализует интерфейс `Illuminate\Contracts\Database\Eloquent\Castable`:

    use App\Models\Address;

    protected $casts = [
        'address' => Address::class,
    ];

Объекты, реализующие интерфейс `Castable`, должны определять метод `castUsing`, который возвращает имя [пользовательского класса типизатора](#value-object-casting), отвечающего за двустороннее преобразование:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use App\Casts\Address as AddressCast;

    class Address implements Castable
    {
        /**
         * Получить имя класса типизатора для использования двустороннего преобразования.
         *
         * @param  array  $arguments
         * @return string
         */
        public static function castUsing(array $arguments)
        {
            return AddressCast::class;
        }
    }

При использовании классов `Castable` вы все равно можете указывать аргументы в свойстве `$casts`. Аргументы будут переданы методу `castUsing`:

    use App\Models\Address;

    protected $casts = [
        'address' => Address::class.':argument',
    ];

<a name="anonymous-cast-classes"></a>
#### Интерфейс `Castable` и анонимные классы типизаторов

Комбинируя `castable` и [анонимными классами](https://www.php.net/manual/ru/language.oop5.anonymous.php) PHP, вы можете определить объект-значение и его логику преобразования как единый типизируемый объект. Для этого верните анонимный класс из метода `castUsing` вашего объекта-значения. Анонимный класс должен реализовывать интерфейс `CastsAttributes`:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Address implements Castable
    {
        // ...

        /**
         * Получить имя класса типизатора для использования двустороннего преобразования.
         *
         * @param  array  $arguments
         * @return object|string
         */
        public static function castUsing(array $arguments)
        {
            return new class implements CastsAttributes
            {
                public function get($model, $key, $value, $attributes)
                {
                    return new Address(
                        $attributes['address_line_one'],
                        $attributes['address_line_two']
                    );
                }

                public function set($model, $key, $value, $attributes)
                {
                    return [
                        'address_line_one' => $value->lineOne,
                        'address_line_two' => $value->lineTwo,
                    ];
                }
            };
        }
    }
