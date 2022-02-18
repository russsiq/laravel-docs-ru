# Laravel 9 · Eloquent · Сериализация

- [Введение](#introduction)
- [Сериализация моделей и коллекций](#serializing-models-and-collections)
    - [Сериализация в массивы](#serializing-to-arrays)
    - [Сериализация в JSON](#serializing-to-json)
- [Скрытие атрибутов из JSON](#hiding-attributes-from-json)
- [Добавление значений в JSON](#appending-values-to-json)
- [Сериализация Даты](#date-serialization)

<a name="introduction"></a>
## Введение

При создании API-интерфейсов с использованием Laravel вам часто нужно преобразовывать свои модели и отношения в массивы или JSON. Eloquent включает удобные методы для выполнения этих преобразований, а также для управления тем, какие атрибуты включаются в сериализованное представление ваших моделей.

> {tip} Чтобы получить еще более надежный способ обработки JSON-сериализации модели Eloquent и коллекции, ознакомьтесь с документацией на [Ресурсы API Eloquent](eloquent-resources.md).

<a name="serializing-models-and-collections"></a>
## Сериализация моделей и коллекций

<a name="serializing-to-arrays"></a>
### Сериализация в массивы

Чтобы преобразовать модель и ее загруженные [отношения](eloquent-relationships.md) в массив, вы должны использовать метод `toArray`. Этот метод является рекурсивным, поэтому все атрибуты и все отношения (включая отношения отношений) будут преобразованы в массивы:

    use App\Models\User;

    $user = User::with('roles')->first();

    return $user->toArray();

Метод `attributesToArray` используется для преобразования атрибутов модели в массив, но не его отношений:

    $user = User::first();

    return $user->attributesToArray();

Вы также можете преобразовать целые [коллекции](eloquent-collections.md) моделей в массивы, вызвав метод `toArray` экземпляра коллекции:

    $users = User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### Сериализация в JSON

Чтобы преобразовать модель в JSON, вы должны использовать метод `toJson`. Как и `toArray`, метод `toJson` является рекурсивным, поэтому все атрибуты и отношения будут преобразованы в JSON. Вы также можете указать любые параметры кодировки JSON, которые [поддерживаются PHP](https://www.php.net/manual/ru/function.json-encode.php):

    use App\Models\User;

    $user = User::find(1);

    return $user->toJson();

    return $user->toJson(JSON_PRETTY_PRINT);

В качестве альтернативы вы можете преобразовать модель или коллекцию в строку, которая автоматически вызовет метод `toJson` модели или коллекции:

    return (string) User::find(1);

Поскольку модели и коллекции преобразуются в JSON при преобразовании в строку, вы можете возвращать объекты Eloquent непосредственно из маршрутов или контроллеров вашего приложения. Laravel автоматически сериализует ваши модели и коллекции Eloquent в JSON, когда они возвращаются из маршрутов или контроллеров:

    Route::get('users', function () {
        return User::all();
    });

<a name="relationships"></a>
#### Отношения

Когда модель Eloquent преобразуется в JSON, ее загруженные отношения автоматически включаются в качестве атрибутов в объект JSON. Кроме того, хотя методы-отношения Eloquent определены с использованием имен методов в «верблюжьей нотации», атрибут JSON отношения будет в «змеиной нотации».

<a name="hiding-attributes-from-json"></a>
## Скрытие атрибутов из JSON

По желанию можно исключить атрибуты, такие как пароли, содержащиеся в массиве модели или представлении JSON. Для этого добавьте в модель свойство `$hidden`. Атрибуты, перечисленные в массиве свойств `$hidden`, не будут включены в сериализованное представление модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть скрыты из массивов.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> {tip} Чтобы скрыть отношения, добавьте имя метода-отношения к свойству `$hidden` модели Eloquent.

В качестве альтернативы вы можете использовать свойство `visible` для определения «разрешенного списка» атрибутов, которые должны быть включены в массив модели и представление JSON. Все атрибуты, отсутствующие в массиве `$visible`, будут скрыты при преобразовании модели в массив или JSON:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть видны в массивах.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

<a name="temporarily-modifying-attribute-visibility"></a>
#### Временное изменение видимости атрибута

По желанию можно сделать некоторые обычно скрытые атрибуты видимыми на конкретном экземпляре модели, для этого используйте метод `makeVisible`. Метод `makeVisible` возвращает экземпляр модели:

    return $user->makeVisible('attribute')->toArray();

Аналогично, если вы хотите скрыть некоторые атрибуты, которые обычно видны, вы можете использовать метод `makeHidden`:

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## Добавление значений в JSON

Иногда при преобразовании моделей в массивы или JSON вы можете добавить атрибуты, которым нет соответствующего столбца в вашей базе данных. Для этого сначала определите [аксессоры](eloquent-mutators.md) для значения:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Определить, является ли пользователь администратором.
         *
         * @return \Illuminate\Database\Eloquent\Casts\Attribute
         */
        protected function isAdmin(): Attribute
        {
            return new Attribute(
                get: fn () => 'yes',
            );
        }
    }

После создания аксессора добавьте имя атрибута к свойству `appends` модели. Обратите внимание, что на имена атрибутов обычно ссылаются в «змеиной нотации», даже если аксессор определяется с помощью «верблюжьей нотации»:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Аксессоры, добавляемые к массиву модели.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

После того, как атрибут был добавлен в список `appends`, он будет включен как в массив модели, так и в представления JSON. Атрибуты в массиве `appends` также будут учитывать настройки `visible` и `hidden`, заданные в модели.

<a name="appending-at-run-time"></a>
#### Добавление во время запроса

Во время выполнения скрипта вы можете указать экземпляру модели добавить дополнительные атрибуты с помощью метода `append`. Или вы можете использовать метод `setAppends`, чтобы переопределить весь массив добавленных свойств для конкретного экземпляра модели:

    return $user->append('is_admin')->toArray();

    return $user->setAppends(['is_admin'])->toArray();

<a name="date-serialization"></a>
## Сериализация Даты

<a name="customizing-the-default-date-format"></a>
#### Настройка формата даты по умолчанию

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

<a name="customizing-the-date-format-per-attribute"></a>
#### Настройка формата даты для каждого атрибута

Вы можете настроить формат сериализации отдельных атрибутов даты, указав формат даты при [объявлении типизации](eloquent-mutators.md#attribute-casting) модели:

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];
