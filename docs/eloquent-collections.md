# Laravel 9 · Eloquent · Коллекции

- [Введение](#introduction)
- [Доступные методы](#available-methods)
- [Пользовательские коллекции](#custom-collections)

<a name="introduction"></a>
## Введение

Все методы Eloquent, которые возвращают более одного результата модели, будут возвращать экземпляры класса `Illuminate\Database\Eloquent\Collection`, включая результаты, полученные с помощью метода `get` или доступные через отношения. Объект коллекции Eloquent расширяет [базовую коллекцию](collections.md) Laravel, поэтому он естественным образом наследует десятки методов, использующих в работе текучий интерфейс с базовым массивом моделей Eloquent. Обязательно ознакомьтесь с документацией по коллекции Laravel, чтобы узнать все об этих полезных методах!

Все коллекции также являются итераторами, что позволяет вам перебирать их, как если бы они были простыми массивами PHP:

    use App\Models\User;

    $users = User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

Однако, как упоминалось ранее, коллекции намного мощнее массивов и предоставляют множество методов типа `map` / `reduce`, которые могут быть связаны с помощью интуитивно понятного интерфейса. Например, мы можем удалить все неактивные модели, а затем собрать имена оставшихся пользователей:

    $names = User::all()->reject(function ($user) {
        return $user->active === false;
    })->map(function ($user) {
        return $user->name;
    });

<a name="eloquent-collection-conversion"></a>
#### Преобразование коллекций Eloquent

В то время как большинство методов коллекции Eloquent возвращают новый экземпляр коллекции Eloquent, методы `collapse`, `flatten`, `flip`, `keys`, `pluck` и `zip` возвращают экземпляр [базовой коллекции](collections.md ). Аналогично, если метод `map` возвращает коллекцию, не содержащую никаких моделей Eloquent, она будет преобразована в экземпляр базовой коллекции.

<a name="available-methods"></a>
## Доступные методы

Все коллекции Eloquent расширяют базовый класс [коллекций Laravel](collections.md#available-methods); поэтому они наследуют все мощные методы, предоставляемые классом базовой коллекции.

Кроме того, класс `Illuminate\Database\Eloquent\Collection` содержит расширенный набор методов, помогающих управлять коллекциями моделей. Большинство методов возвращают экземпляры `Illuminate\Database\Eloquent\Collection`; однако некоторые методы, такие как `modelKeys`, возвращают экземпляр `Illuminate\Support\Collection`.

<!-- <style>
    .collection-method-list > p {
        columns: 14.4em 1; -moz-columns: 14.4em 1; -webkit-columns: 14.4em 1;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style> -->

<!-- <div class="collection-method-list" markdown="1"> -->

- [append](#method-append)
- [contains](#method-contains)
- [diff](#method-diff)
- [except](#method-except)
- [find](#method-find)
- [fresh](#method-fresh)
- [intersect](#method-intersect)
- [load](#method-load)
- [loadMissing](#method-loadMissing)
- [modelKeys](#method-modelKeys)
- [makeVisible](#method-makeVisible)
- [makeHidden](#method-makeHidden)
- [only](#method-only)
- [toQuery](#method-toquery)
- [unique](#method-unique)

<!-- </div> -->

<a name="method-append"></a>
#### `append($attributes)`

Метод `append`используется для указания того, что атрибут должен быть [добавлен](eloquent-serialization.md#appending-values-to-json) для каждой модели в коллекции. Этот метод принимает один атрибут или массив атрибутов:

    $users->append('team');

    $users->append(['team', 'is_admin']);

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)`

Метод `contains` используется для определения того, содержится ли переданный экземпляр модели в коллекции. Этот метод принимает первичный ключ или экземпляр модели:

    $users->contains(1);

    $users->contains(User::find(1));

<a name="method-diff"></a>
#### `diff($items)`

Метод `diff` возвращает все модели, которых нет в переданной коллекции:

    use App\Models\User;

    $users = $users->diff(User::whereIn('id', [1, 2, 3])->get());

<a name="method-except"></a>
#### `except($keys)`

Метод `except` возвращает все модели, у которых нет переданных первичных ключей:

    $users = $users->except([1, 2, 3]);

<a name="method-find"></a>
#### `find($key)`

Метод `find` возвращает модель, у которой есть первичный ключ, соответствующий переданному ключу. Если `$key` является экземпляром модели, `find` попытается вернуть модель, соответствующую первичному ключу. Если `$key` является массивом ключей, `find` вернет все модели, у которых есть первичный ключ в переданном массиве:

    $users = User::all();

    $user = $users->find(1);

<a name="method-fresh"></a>
#### `fresh($with = [])`

Метод `fresh` извлекает из базы данных свежий экземпляр каждой модели в коллекции. Кроме того, будут загружены любые указанные отношения:

    $users = $users->fresh();

    $users = $users->fresh('comments');

<a name="method-intersect"></a>
#### `intersect($items)`

Метод `intersect` возвращает все модели, которые также присутствуют в переданной коллекции:

    use App\Models\User;

    $users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());

<a name="method-load"></a>
#### `load($relations)`

Метод `load` нетерпеливо загружает указанные отношения для всех моделей в коллекции:

    $users->load(['comments', 'posts']);

    $users->load('comments.author');

<a name="method-loadMissing"></a>
#### `loadMissing($relations)`

Метод `loadMissing` нетерпеливо загружает указанные отношения для всех моделей в коллекции, если отношения еще не загружены:

    $users->loadMissing(['comments', 'posts']);

    $users->loadMissing('comments.author');

<a name="method-modelKeys"></a>
#### `modelKeys()`

Метод `modelKeys` возвращает первичные ключи для всех моделей в коллекции:

    $users->modelKeys();

    // [1, 2, 3, 4, 5]

<a name="method-makeVisible"></a>
#### `makeVisible($attributes)`

Метод `makeVisible` [делает видимыми атрибуты](eloquent-serialization.md#hiding-attributes-from-json), которые обычно «скрыты» для каждой модели коллекции:

    $users = $users->makeVisible(['address', 'phone_number']);

<a name="method-makeHidden"></a>
#### `makeHidden($attributes)`

Метод `makeHidden` [скрывает атрибуты](eloquent-serialization.md#hiding-attributes-from-json), которые обычно «видны» для каждой модели в коллекции:

    $users = $users->makeHidden(['address', 'phone_number']);

<a name="method-only"></a>
#### `only($keys)`

Метод `only` возвращает все модели с указанными первичными ключами:

    $users = $users->only([1, 2, 3]);

<a name="method-toquery"></a>
#### `toQuery()`

Метод `toQuery` возвращает экземпляр построителя запросов Eloquent, содержащий ограничение `whereIn` для первичных ключей модели коллекции:

    use App\Models\User;

    $users = User::where('status', 'VIP')->get();

    $users->toQuery()->update([
        'status' => 'Administrator',
    ]);

<a name="method-unique"></a>
#### `unique($key = null, $strict = false)`

Метод `unique` возвращает все уникальные модели в коллекции. Все модели того же типа с тем же первичным ключом, что и другая модель в коллекции, удаляются:

    $users = $users->unique();

<a name="custom-collections"></a>
## Пользовательские коллекции

Если вы хотите использовать пользовательский класс `Collection` при взаимодействии с конкретной моделью, вы можете определить метод `newCollection` модели:

    <?php

    namespace App\Models;

    use App\Support\UserCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Создать новый экземпляр коллекции Eloquent.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new UserCollection($models);
        }
    }

После того как вы определили метод `newCollection`, вы будете получать экземпляр своей пользовательской коллекции в любое время, когда Eloquent обычно возвращает экземпляр `Illuminate\Database\Eloquent\Collection`. Если вы хотите использовать собственную коллекцию для каждой модели в вашем приложении, вы должны определить метод `newCollection` для класса базовой модели, который расширяется всеми моделями вашего приложения.
