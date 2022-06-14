# Laravel 9 · Глобальные помощники

- [Введение](#introduction)
- [Доступные методы](#available-methods)

<a name="introduction"></a>
## Введение

Laravel содержит множество глобальных «вспомогательных» функций PHP. Многие из этих функций используются самим фреймворком; однако, вы можете использовать их в своих собственных приложениях, если сочтете их удобными.

<a name="available-methods"></a>
## Доступные методы

<!-- <style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style> -->

<a name="arrays-and-objects-method-list"></a>
### Массивы и объекты

<!-- <div class="collection-method-list" markdown="1"> -->

- [Arr::accessible](#method-array-accessible)
- [Arr::add](#method-array-add)
- [Arr::collapse](#method-array-collapse)
- [Arr::crossJoin](#method-array-crossjoin)
- [Arr::divide](#method-array-divide)
- [Arr::dot](#method-array-dot)
- [Arr::except](#method-array-except)
- [Arr::exists](#method-array-exists)
- [Arr::first](#method-array-first)
- [Arr::flatten](#method-array-flatten)
- [Arr::forget](#method-array-forget)
- [Arr::get](#method-array-get)
- [Arr::has](#method-array-has)
- [Arr::hasAny](#method-array-hasany)
- [Arr::isAssoc](#method-array-isassoc)
- [Arr::isList](#method-array-islist)
- [Arr::join](#method-array-join)
- [Arr::keyBy](#method-array-keyby)
- [Arr::last](#method-array-last)
- [Arr::map](#method-array-map)
- [Arr::only](#method-array-only)
- [Arr::pluck](#method-array-pluck)
- [Arr::prepend](#method-array-prepend)
- [Arr::prependKeysWith](#method-array-prependkeyswith)
- [Arr::pull](#method-array-pull)
- [Arr::query](#method-array-query)
- [Arr::random](#method-array-random)
- [Arr::set](#method-array-set)
- [Arr::shuffle](#method-array-shuffle)
- [Arr::sort](#method-array-sort)
- [Arr::sortRecursive](#method-array-sort-recursive)
- [Arr::toCssClasses](#method-array-to-css-classes)
- [Arr::undot](#method-array-undot)
- [Arr::where](#method-array-where)
- [Arr::whereNotNull](#method-array-where-not-null)
- [Arr::wrap](#method-array-wrap)
- [data_fill](#method-data-fill)
- [data_get](#method-data-get)
- [data_set](#method-data-set)
- [head](#method-head)
- [last](#method-last)
<!-- </div> -->

<a name="paths-method-list"></a>
### Пути

<!-- <div class="collection-method-list" markdown="1"> -->

- [app_path](#method-app-path)
- [base_path](#method-base-path)
- [config_path](#method-config-path)
- [database_path](#method-database-path)
- [lang_path](#method-lang-path)
- [mix](#method-mix)
- [public_path](#method-public-path)
- [resource_path](#method-resource-path)
- [storage_path](#method-storage-path)

<!-- </div> -->

<a name="strings-method-list"></a>
### Строки

<!-- <div class="collection-method-list" markdown="1"> -->

- [\__](#method-__)
- [class_basename](#method-class-basename)
- [e](#method-e)
- [preg_replace_array](#method-preg-replace-array)
- [Str::after](#method-str-after)
- [Str::afterLast](#method-str-after-last)
- [Str::ascii](#method-str-ascii)
- [Str::before](#method-str-before)
- [Str::beforeLast](#method-str-before-last)
- [Str::between](#method-str-between)
- [Str::betweenFirst](#method-str-between-first)
- [Str::camel](#method-camel-case)
- [Str::contains](#method-str-contains)
- [Str::containsAll](#method-str-contains-all)
- [Str::endsWith](#method-ends-with)
- [Str::excerpt](#method-excerpt)
- [Str::finish](#method-str-finish)
- [Str::headline](#method-str-headline)
- [Str::is](#method-str-is)
- [Str::isAscii](#method-str-is-ascii)
- [Str::isJson](#method-str-is-json)
- [Str::isUuid](#method-str-is-uuid)
- [Str::kebab](#method-kebab-case)
- [Str::lcfirst](#method-str-lcfirst)
- [Str::length](#method-str-length)
- [Str::limit](#method-str-limit)
- [Str::lower](#method-str-lower)
- [Str::markdown](#method-str-markdown)
- [Str::mask](#method-str-mask)
- [Str::orderedUuid](#method-str-ordered-uuid)
- [Str::padBoth](#method-str-padboth)
- [Str::padLeft](#method-str-padleft)
- [Str::padRight](#method-str-padright)
- [Str::plural](#method-str-plural)
- [Str::pluralStudly](#method-str-plural-studly)
- [Str::random](#method-str-random)
- [Str::remove](#method-str-remove)
- [Str::replace](#method-str-replace)
- [Str::replaceArray](#method-str-replace-array)
- [Str::replaceFirst](#method-str-replace-first)
- [Str::replaceLast](#method-str-replace-last)
- [Str::reverse](#method-str-reverse)
- [Str::singular](#method-str-singular)
- [Str::slug](#method-str-slug)
- [Str::snake](#method-snake-case)
- [Str::squish](#method-str-squish)
- [Str::start](#method-str-start)
- [Str::startsWith](#method-starts-with)
- [Str::studly](#method-studly-case)
- [Str::substr](#method-str-substr)
- [Str::substrCount](#method-str-substrcount)
- [Str::substrReplace](#method-str-substrreplace)
- [Str::swap](#method-str-swap)
- [Str::title](#method-title-case)
- [Str::toHtmlString](#method-str-to-html-string)
- [Str::ucfirst](#method-str-ucfirst)
- [Str::ucsplit](#method-str-ucsplit)
- [Str::upper](#method-str-upper)
- [Str::uuid](#method-str-uuid)
- [Str::wordCount](#method-str-word-count)
- [Str::words](#method-str-words)
- [str](#method-str)
- [trans](#method-trans)
- [trans_choice](#method-trans-choice)

<!-- </div> -->

<a name="fluent-strings-method-list"></a>
### Строки Fluent

<!-- <div class="collection-method-list" markdown="1"> -->

- [after](#method-fluent-str-after)
- [afterLast](#method-fluent-str-after-last)
- [append](#method-fluent-str-append)
- [ascii](#method-fluent-str-ascii)
- [basename](#method-fluent-str-basename)
- [before](#method-fluent-str-before)
- [beforeLast](#method-fluent-str-before-last)
- [between](#method-fluent-str-between)
- [betweenFirst](#method-fluent-str-between-first)
- [camel](#method-fluent-str-camel)
- [classBasename](#method-fluent-str-class-basename)
- [contains](#method-fluent-str-contains)
- [containsAll](#method-fluent-str-contains-all)
- [dirname](#method-fluent-str-dirname)
- [endsWith](#method-fluent-str-ends-with)
- [excerpt](#method-fluent-str-excerpt)
- [exactly](#method-fluent-str-exactly)
- [explode](#method-fluent-str-explode)
- [finish](#method-fluent-str-finish)
- [is](#method-fluent-str-is)
- [isAscii](#method-fluent-str-is-ascii)
- [isEmpty](#method-fluent-str-is-empty)
- [isNotEmpty](#method-fluent-str-is-not-empty)
- [isJson](#method-fluent-str-is-json)
- [isUuid](#method-fluent-str-is-uuid)
- [kebab](#method-fluent-str-kebab)
- [lcfirst](#method-fluent-str-lcfirst)
- [length](#method-fluent-str-length)
- [limit](#method-fluent-str-limit)
- [lower](#method-fluent-str-lower)
- [ltrim](#method-fluent-str-ltrim)
- [markdown](#method-fluent-str-markdown)
- [mask](#method-fluent-str-mask)
- [match](#method-fluent-str-match)
- [matchAll](#method-fluent-str-match-all)
- [newLine](#method-fluent-str-new-line)
- [padBoth](#method-fluent-str-padboth)
- [padLeft](#method-fluent-str-padleft)
- [padRight](#method-fluent-str-padright)
- [pipe](#method-fluent-str-pipe)
- [plural](#method-fluent-str-plural)
- [prepend](#method-fluent-str-prepend)
- [remove](#method-fluent-str-remove)
- [replace](#method-fluent-str-replace)
- [replaceArray](#method-fluent-str-replace-array)
- [replaceFirst](#method-fluent-str-replace-first)
- [replaceLast](#method-fluent-str-replace-last)
- [replaceMatches](#method-fluent-str-replace-matches)
- [rtrim](#method-fluent-str-rtrim)
- [scan](#method-fluent-str-scan)
- [singular](#method-fluent-str-singular)
- [slug](#method-fluent-str-slug)
- [snake](#method-fluent-str-snake)
- [split](#method-fluent-str-split)
- [squish](#method-fluent-str-squish)
- [start](#method-fluent-str-start)
- [startsWith](#method-fluent-str-starts-with)
- [studly](#method-fluent-str-studly)
- [substr](#method-fluent-str-substr)
- [substrReplace](#method-fluent-str-substrreplace)
- [swap](#method-fluent-str-swap)
- [tap](#method-fluent-str-tap)
- [test](#method-fluent-str-test)
- [title](#method-fluent-str-title)
- [trim](#method-fluent-str-trim)
- [ucfirst](#method-fluent-str-ucfirst)
- [ucsplit](#method-fluent-str-ucsplit)
- [upper](#method-fluent-str-upper)
- [when](#method-fluent-str-when)
- [whenContains](#method-fluent-str-when-contains)
- [whenContainsAll](#method-fluent-str-when-contains-all)
- [whenEmpty](#method-fluent-str-when-empty)
- [whenNotEmpty](#method-fluent-str-when-not-empty)
- [whenStartsWith](#method-fluent-str-when-starts-with)
- [whenEndsWith](#method-fluent-str-when-ends-with)
- [whenExactly](#method-fluent-str-when-exactly)
- [whenIs](#method-fluent-str-when-is)
- [whenIsAscii](#method-fluent-str-when-is-ascii)
- [whenIsUuid](#method-fluent-str-when-is-uuid)
- [whenTest](#method-fluent-str-when-test)
- [wordCount](#method-fluent-str-word-count)
- [words](#method-fluent-str-words)

<!-- </div> -->

<a name="urls-method-list"></a>
### URL-адреса

<!-- <div class="collection-method-list" markdown="1"> -->

- [action](#method-action)
- [asset](#method-asset)
- [route](#method-route)
- [secure_asset](#method-secure-asset)
- [secure_url](#method-secure-url)
- [to_route](#method-to-route)
- [url](#method-url)

<!-- </div> -->

<a name="miscellaneous-method-list"></a>
### Разное

<!-- <div class="collection-method-list" markdown="1"> -->

- [abort](#method-abort)
- [abort_if](#method-abort-if)
- [abort_unless](#method-abort-unless)
- [app](#method-app)
- [auth](#method-auth)
- [back](#method-back)
- [bcrypt](#method-bcrypt)
- [blank](#method-blank)
- [broadcast](#method-broadcast)
- [cache](#method-cache)
- [class_uses_recursive](#method-class-uses-recursive)
- [collect](#method-collect)
- [config](#method-config)
- [cookie](#method-cookie)
- [csrf_field](#method-csrf-field)
- [csrf_token](#method-csrf-token)
- [decrypt](#method-decrypt)
- [dd](#method-dd)
- [dispatch](#method-dispatch)
- [dump](#method-dump)
- [encrypt](#method-encrypt)
- [env](#method-env)
- [event](#method-event)
- [filled](#method-filled)
- [info](#method-info)
- [logger](#method-logger)
- [method_field](#method-method-field)
- [now](#method-now)
- [old](#method-old)
- [optional](#method-optional)
- [policy](#method-policy)
- [redirect](#method-redirect)
- [report](#method-report)
- [request](#method-request)
- [rescue](#method-rescue)
- [resolve](#method-resolve)
- [response](#method-response)
- [retry](#method-retry)
- [session](#method-session)
- [tap](#method-tap)
- [throw_if](#method-throw-if)
- [throw_unless](#method-throw-unless)
- [today](#method-today)
- [trait_uses_recursive](#method-trait-uses-recursive)
- [transform](#method-transform)
- [validator](#method-validator)
- [value](#method-value)
- [view](#method-view)
- [with](#method-with)

<!-- </div> -->

<a name="method-listing"></a>
## Список методов

<!-- <style>
    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style> -->

<a name="arrays"></a>
## Массивы и объекты

<a name="method-array-accessible"></a>
#### `Arr::accessible()`

Метод `Arr::accessible` определяет, доступно ли переданное значение массиву:

    use Illuminate\Support\Arr;
    use Illuminate\Support\Collection;

    $isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

    // true

    $isAccessible = Arr::accessible(new Collection);

    // true

    $isAccessible = Arr::accessible('abc');

    // false

    $isAccessible = Arr::accessible(new stdClass);

    // false

<a name="method-array-add"></a>
#### `Arr::add()`

Метод `Arr::add` добавляет переданную пару ключ / значение в массив, если указанный ключ еще не существует в массиве или установлен как `null`:

    use Illuminate\Support\Arr;

    $array = Arr::add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

    $array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]


<a name="method-array-collapse"></a>
#### `Arr::collapse()`

Метод `Arr::collapse` сворачивает массив массивов в один массив:

    use Illuminate\Support\Arr;

    $array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-crossjoin"></a>
#### `Arr::crossJoin()`

Метод `Arr::crossJoin` перекрестно соединяет указанные массивы, возвращая декартово произведение со всеми возможными перестановками:

    use Illuminate\Support\Arr;

    $matrix = Arr::crossJoin([1, 2], ['a', 'b']);

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-array-divide"></a>
#### `Arr::divide()`

Метод `Arr::divide` возвращает два массива: один содержит ключи, а другой – значения переданного массива:

    use Illuminate\Support\Arr;

    [$keys, $values] = Arr::divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `Arr::dot()`

Метод `Arr::dot` объединяет многомерный массив в одноуровневый, использующий «точечную нотацию» для обозначения глубины:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = Arr::dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `Arr::except()`

Метод `Arr::except` удаляет переданные пары ключ / значение из массива:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = Arr::except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-exists"></a>
#### `Arr::exists()`

Метод `Arr::exists` проверяет, существует ли переданный ключ в указанном массиве:

    use Illuminate\Support\Arr;

    $array = ['name' => 'John Doe', 'age' => 17];

    $exists = Arr::exists($array, 'name');

    // true

    $exists = Arr::exists($array, 'salary');

    // false

<a name="method-array-first"></a>
#### `Arr::first()`

Метод `Arr::first` возвращает первый элемент массива, прошедший тест переданного замыкания на истинность:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300];

    $first = Arr::first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

Значение по умолчанию может быть передано в качестве третьего аргумента методу. Это значение будет возвращено, если ни одно из значений не пройдет проверку на истинность:

    use Illuminate\Support\Arr;

    $first = Arr::first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `Arr::flatten()`

Метод `Arr::flatten` объединяет многомерный массив в одноуровневый:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = Arr::flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `Arr::forget()`

Метод `Arr::forget` удаляет переданную пару ключ / значение из глубоко вложенного массива, используя «точечную нотацию»:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `Arr::get()`

Метод `Arr::get` извлекает значение из глубоко вложенного массива, используя «точечную нотацию»:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = Arr::get($array, 'products.desk.price');

    // 100

Метод `Arr::get` также принимает значение по умолчанию, которое будет возвращено, если указанный ключ отсутствует в массиве:

    use Illuminate\Support\Arr;

    $discount = Arr::get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `Arr::has()`

Метод `Arr::has` проверяет, существует ли переданный элемент или элементы в массиве, используя «точечную нотацию»:

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::has($array, 'product.name');

    // true

    $contains = Arr::has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-hasany"></a>
#### `Arr::hasAny()`

Метод `Arr::hasAny` проверяет, существует ли какой-либо элемент в переданном наборе в массиве, используя «точечную нотацию»:

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::hasAny($array, 'product.name');

    // true

    $contains = Arr::hasAny($array, ['product.name', 'product.discount']);

    // true

    $contains = Arr::hasAny($array, ['category', 'product.discount']);

    // false

<a name="method-array-isassoc"></a>
#### `Arr::isAssoc()`

Метод `Arr::isAssoc` возвращает `true`, если переданный массив является ассоциативным. Массив считается ассоциативным, если в нем нет последовательных цифровых ключей, начинающихся с нуля:

    use Illuminate\Support\Arr;

    $isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

    // true

    $isAssoc = Arr::isAssoc([1, 2, 3]);

    // false

<a name="method-array-islist"></a>
#### `Arr::isList()`

Метод `Arr::isList` возвращает `true`, если ключи переданного массива являются последовательными целыми числами, начинающимися с нуля:

    use Illuminate\Support\Arr;

    $isAssoc = Arr::isList(['foo', 'bar', 'baz']);

    // true

    $isAssoc = Arr::isList(['product' => ['name' => 'Desk', 'price' => 100]]);

    // false

<a name="method-array-join"></a>
#### `Arr::join()`

Метод `Arr::join` объединяет элементы массива в строку. Используя третий аргумент метода, вы также можете указать объединяющую строку для последнего элемента массива:

    use Illuminate\Support\Arr;

    $array = ['Tailwind', 'Alpine', 'Laravel', 'Livewire'];

    $joined = Arr::join($array, ', ');

    // Tailwind, Alpine, Laravel, Livewire

    $joined = Arr::join($array, ', ', ' and ');

    // Tailwind, Alpine, Laravel and Livewire

<a name="method-array-keyby"></a>
#### `Arr::keyBy()`

Метод `Arr::keyBy` группирует массив по значениям переданного ключа. Если несколько элементов имеют один и тот же ключ, в новом массиве появится только последний:

    use Illuminate\Support\Arr;

    $array = [
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ];

    $keyed = Arr::keyBy($array, 'product_id');

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-array-last"></a>
#### `Arr::last()`

Метод `Arr::last` возвращает последний элемент массива, прошедший тест переданного замыкания на истинность:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300, 110];

    $last = Arr::last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

Значение по умолчанию может быть передано в качестве третьего аргумента методу. Это значение будет возвращено, если ни одно из значений не пройдет проверку на истинность:

    use Illuminate\Support\Arr;

    $last = Arr::last($array, $callback, $default);

<a name="method-array-map"></a>
#### `Arr::map()`

Метод `Arr::map` перебирает массив и передает каждое значение и ключ указанному замыканию. Значение массива заменяется значением, которое было возвращено замыканием:

    use Illuminate\Support\Arr;

    $array = ['first' => 'james', 'last' => 'kirk'];

    $mapped = Arr::map($array, function ($value, $key) {
        return ucfirst($value);
    });

    // ['first' => 'James', 'last' => 'Kirk']

<a name="method-array-only"></a>
#### `Arr::only()`

Метод `Arr::only` возвращает только указанные пары ключ / значение из переданного массива:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = Arr::only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `Arr::pluck()`

Метод `Arr::pluck` извлекает все значения для указанного ключа из массива:

    use Illuminate\Support\Arr;

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = Arr::pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

Вы также можете задать ключ результирующего списка:

    use Illuminate\Support\Arr;

    $names = Arr::pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `Arr::prepend()`

Метод `Arr::prepend` помещает элемент в начало массива:

    use Illuminate\Support\Arr;

    $array = ['one', 'two', 'three', 'four'];

    $array = Arr::prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

При необходимости вы можете указать ключ, который следует использовать для значения:

    use Illuminate\Support\Arr;

    $array = ['price' => 100];

    $array = Arr::prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-prependkeyswith"></a>
#### `Arr::prependKeysWith()`

Метод `Arr::prependKeysWith` добавляет ко всем именам ключей ассоциативного массива переданный префикс:

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Desk',
        'price' => 100,
    ];

    $keyed = Arr::prependKeysWith($array, 'product.');

    /*
        [
            'product.name' => 'Desk',
            'product.price' => 100,
        ]
    */

<a name="method-array-pull"></a>
#### `Arr::pull()`

Метод `Arr::pull` возвращает и удаляет пару ключ / значение из массива:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $name = Arr::pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Значение по умолчанию может быть передано в качестве третьего аргумента методу. Это значение будет возвращено, если ключ не существует:

    use Illuminate\Support\Arr;

    $value = Arr::pull($array, $key, $default);

<a name="method-array-query"></a>
#### `Arr::query()`

Метод `Arr::query` преобразует массив в строку запроса:

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Taylor',
        'order' => [
            'column' => 'created_at',
            'direction' => 'desc'
        ]
    ];

    Arr::query($array);

    // name=Taylor&order[column]=created_at&order[direction]=desc

<a name="method-array-random"></a>
#### `Arr::random()`

Метод `Arr::random` возвращает случайное значение из массива:

    use Illuminate\Support\Arr;

    $array = [1, 2, 3, 4, 5];

    $random = Arr::random($array);

    // 4 - (retrieved randomly)

Вы также можете указать количество элементов для возврата в качестве необязательного второго аргумента. Обратите внимание, что при указании этого аргумента, будет возвращен массив, даже если требуется только один элемент:

    use Illuminate\Support\Arr;

    $items = Arr::random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `Arr::set()`

Метод `Arr::set` устанавливает значение с помощью «точечной нотации» во вложенном массиве:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-shuffle"></a>
#### `Arr::shuffle()`

Метод `Arr::shuffle` случайным образом перемешивает элементы в массиве:

    use Illuminate\Support\Arr;

    $array = Arr::shuffle([1, 2, 3, 4, 5]);

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-array-sort"></a>
#### `Arr::sort()`

Метод `Arr::sort` сортирует массив по его значениям:

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sort($array);

    // ['Chair', 'Desk', 'Table']

Вы также можете отсортировать массив по результатам переданного замыкания:

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()`

Метод `Arr::sortRecursive` рекурсивно сортирует массив с помощью метода `sort` для числовых подмассивов и `ksort` для ассоциативных подмассивов:

    use Illuminate\Support\Arr;

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
        ['one' => 1, 'two' => 2, 'three' => 3],
    ];

    $sorted = Arr::sortRecursive($array);

    /*
        [
            ['JavaScript', 'PHP', 'Ruby'],
            ['one' => 1, 'three' => 3, 'two' => 2],
            ['Li', 'Roman', 'Taylor'],
        ]
    */

<a name="method-array-to-css-classes"></a>
#### `Arr::toCssClasses()`

Метод `Arr::toCssClasses` условно компилирует строку класса CSS. Метод принимает массив классов, где ключ массива содержит класс или классы, которые вы хотите добавить, а значение является логическим выражением. Если элемент массива имеет числовой ключ, то он всегда будет включен в отображаемый список классов:

    use Illuminate\Support\Arr;

    $isActive = false;
    $hasError = true;

    $array = ['p-4', 'font-bold' => $isActive, 'bg-red' => $hasError];

    $classes = Arr::toCssClasses($array);

    /*
        'p-4 bg-red'
    */

Этот метод обеспечивает функциональность Laravel, позволяя [объединять классы с коллекцией атрибутов компонента Blade](blade.md#conditionally-merge-classes), а также использовать [директиву `@class` Blade](blade.md#conditional-classes).

<a name="method-array-undot"></a>
#### `Arr::undot()`

Метод `Arr::undot` преобразует одноуровневый массив с «точечной нотацией» в многомерный массив:

    use Illuminate\Support\Arr;

    $array = [
        'user.name' => 'Kevin Malone',
        'user.occupation' => 'Accountant',
    ];

    $array = Arr::undot($array);

    // ['user' => ['name' => 'Kevin Malone', 'occupation' => 'Accountant']]

<a name="method-array-where"></a>
#### `Arr::where()`

Метод `Arr::where` фильтрует массив, используя переданное замыкание:

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-where-not-null"></a>
#### `Arr::whereNotNull()`

Метод `Arr::whereNotNull` удаляет все `null`-значения массива:

    use Illuminate\Support\Arr;

    $array = [0, null];

    $filtered = Arr::whereNotNull($array);

    // [0 => 0]

<a name="method-array-wrap"></a>
#### `Arr::wrap()`

Метод `Arr::wrap` оборачивает переданное значение в массив. Если переданное значение уже является массивом, то оно будет возвращено без изменений:

    use Illuminate\Support\Arr;

    $string = 'Laravel';

    $array = Arr::wrap($string);

    // ['Laravel']

Если переданное значение равно `null`, то будет возвращен пустой массив:

    use Illuminate\Support\Arr;

    $array = Arr::wrap(null);

    // []

<a name="method-data-fill"></a>
#### `data_fill()`

Функция `data_fill` устанавливает отсутствующее значение с помощью «точечной нотации» во вложенном массиве или объекте:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

Допускается использование метасимвола подстановки `*`:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>
#### `data_get()`

Функция `data_get` возвращает значение с помощью «точечной нотации» из вложенного массива или объекта:

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

Функция `data_get` также принимает значение по умолчанию, которое будет возвращено, если указанный ключ не найден:

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

Допускается использование метасимвола подстановки `*`, предназначенный для любого ключа массива или объекта:

    $data = [
        'product-one' => ['name' => 'Desk 1', 'price' => 100],
        'product-two' => ['name' => 'Desk 2', 'price' => 150],
    ];

    data_get($data, '*.name');

    // ['Desk 1', 'Desk 2'];

<a name="method-data-set"></a>
#### `data_set()`

Функция `data_set` устанавливает значение с помощью «точечной нотации» во вложенном массиве или объекте:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Допускается использование метасимвола подстановки `*`:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

По умолчанию все существующие значения перезаписываются. Если вы хотите, чтобы значение было установлено только в том случае, если оно не существует, вы можете передать `false` в качестве четвертого аргумента:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, $overwrite = false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-head"></a>
#### `head()`

Функция `head` возвращает первый элемент переданного массива:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()`

Функция `last` возвращает последний элемент переданного массива:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## Пути

<a name="method-app-path"></a>
#### `app_path()`

Функция `app_path` возвращает полный путь к каталогу вашего приложения `app`. Вы также можете использовать функцию `app_path` для создания полного пути к файлу относительно каталога приложения:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()`

Функция `base_path` возвращает полный путь к корневому каталогу вашего приложения. Вы также можете использовать функцию `base_path` для генерации полного пути к заданному файлу относительно корневого каталога проекта:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()`

Функция `config_path` возвращает полный путь к каталогу `config` вашего приложения. Вы также можете использовать функцию `config_path` для создания полного пути к заданному файлу в каталоге конфигурации приложения:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()`

Функция `database_path` возвращает полный путь к каталогу `database` вашего приложения. Вы также можете использовать функцию `database_path` для генерации полного пути к заданному файлу в каталоге базы данных:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-lang-path"></a>
#### `lang_path()`

Функция `lang_path` возвращает полный путь к каталогу `lang` вашего приложения. Вы также можете использовать функцию `lang_path` для создания полного пути к конкретному файлу в каталоге:

    $path = lang_path();

    $path = lang_path('en/messages.php');

<a name="method-mix"></a>
#### `mix()`

Функция `mix` возвращает путь к [версионированному файлу Mix](mix.md#versioning-and-cache-busting):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()`

Функция `public_path` возвращает полный путь к каталогу `public` вашего приложения. Вы также можете использовать функцию `public_path` для генерации полного пути к заданному файлу в публичном каталоге:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()`

Функция `resource_path` возвращает полный путь к каталогу `resources` вашего приложения. Вы также можете использовать функцию `resource_path`, чтобы сгенерировать полный путь к заданному файлу в каталоге исходников:

    $path = resource_path();

    $path = resource_path('sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()`

Функция `storage_path` возвращает полный путь к каталогу `storage` вашего приложения. Вы также можете использовать функцию `storage_path` для генерации полного пути к заданному файлу в каталоге хранилища:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Строки

<a name="method-__"></a>
#### `__()`

Функция `__` переводит переданную строку перевода или ключ перевода, используя ваши [файлы локализации](localization.md):

    echo __('Welcome to our application');

    echo __('messages.welcome');

Если указанная строка перевода или ключ не существует, то функция `__` вернет переданное значение. Итак, используя приведенный выше пример, функция `__` вернет `messages.welcome`, если этот ключ перевода не существует.

<a name="method-class-basename"></a>
#### `class_basename()`

Функция `class_basename` возвращает имя переданного класса с удаленным пространством имен этого класса:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()`

Функция `e` запускает PHP-функцию `htmlspecialchars` с параметром `double_encode`, установленным по умолчанию в `true`:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()`

Функция `preg_replace_array` последовательно заменяет переданный шаблон в строке, используя массив:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-after"></a>
#### `Str::after()`

Метод `Str::after` возвращает все после переданного значения в строке. Если значение не существует в строке, то будет возвращена вся строка:

    use Illuminate\Support\Str;

    $slice = Str::after('This is my name', 'This is');

    // ' my name'

<a name="method-str-after-last"></a>
#### `Str::afterLast()`

Метод `Str::afterLast` возвращает все после последнего вхождения переданного значения в строке. Если значение не существует в строке, то будет возвращена вся строка:

    use Illuminate\Support\Str;

    $slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

    // 'Controller'

<a name="method-str-ascii"></a>
#### `Str::ascii()`

Метод `Str::ascii` попытается транслитерировать строку в ASCII значение:

    use Illuminate\Support\Str;

    $slice = Str::ascii('û');

    // 'u'

<a name="method-str-before"></a>
#### `Str::before()`

Метод `Str :: before` возвращает все до переданного значения в строке:

    use Illuminate\Support\Str;

    $slice = Str::before('This is my name', 'my name');

    // 'This is '

<a name="method-str-before-last"></a>
#### `Str::beforeLast()`

Метод `Str::beforeLast` возвращает все до последнего вхождения переданного значения в строке:

    use Illuminate\Support\Str;

    $slice = Str::beforeLast('This is my name', 'is');

    // 'This '

<a name="method-str-between"></a>
#### `Str::between()`

Метод `Str::between` возвращает часть строки между двумя значениями:

    use Illuminate\Support\Str;

    $slice = Str::between('This is my name', 'This', 'name');

    // ' is my '

<a name="method-str-between-first"></a>
#### `Str::betweenFirst()`

Метод `Str::betweenFirst` возвращает наименьшую возможную часть строки между двумя значениями:

    use Illuminate\Support\Str;

    $slice = Str::betweenFirst('[a] bc [d]', '[', ']');

    // 'a'

<a name="method-camel-case"></a>
#### `Str::camel()`

Метод `Str::camel` преобразует переданную строку в `camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::camel('foo_bar');

    // fooBar

<a name="method-str-contains"></a>
#### `Str::contains()`

Метод `Str::contains` определяет, содержит ли переданная строка указанное значение (с учетом регистра):

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', 'my');

    // true

Вы также можете указать массив значений, чтобы определить, содержит ли переданная строка какое-либо из значений:

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-contains-all"></a>
#### `Str::containsAll()`

Метод `Str::containsAll` определяет, содержит ли переданная строка все значения массива:

    use Illuminate\Support\Str;

    $containsAll = Str::containsAll('This is my name', ['my', 'name']);

    // true

<a name="method-ends-with"></a>
#### `Str::endsWith()`

Метод `Str::endsWith` определяет, заканчивается ли переданная строка указанным значением:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', 'name');

    // true


Вы также можете указать массив значений, чтобы определить, заканчивается ли переданная строка каким-либо из значений:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', ['name', 'foo']);

    // true

    $result = Str::endsWith('This is my name', ['this', 'foo']);

    // false

<a name="method-excerpt"></a>
#### `Str::excerpt()`

Метод `Str::excerpt` извлекает отрывок из переданной строки, который соответствует первому экземпляру фразы в этой строке:

    use Illuminate\Support\Str;

    $excerpt = Str::excerpt('This is my name', 'my', [
        'radius' => 3
    ]);

    // '...is my na...'

Параметр `radius`, который по умолчанию равен `100`, позволяет вам определить количество символов, которые должны присутствовать с каждой стороны усеченной строки.

Кроме того, вы можете использовать параметр `omission`, чтобы определить строку, которая будет добавлена к усеченной строке:

    use Illuminate\Support\Str;

    $excerpt = Str::excerpt('This is my name', 'name', [
        'radius' => 3,
        'omission' => '(...) '
    ]);

    // '(...) my name'

<a name="method-str-finish"></a>
#### `Str::finish()`

Метод `Str::finish` добавляет один экземпляр указанного значения в переданную строку, если она еще не заканчивается этим значением:

    use Illuminate\Support\Str;

    $adjusted = Str::finish('this/string', '/');

    // this/string/

    $adjusted = Str::finish('this/string/', '/');

    // this/string/

<a name="method-str-headline"></a>
#### `Str::headline()`

Метод `Str::headline` преобразует строки, разделенные регистром, дефисами или подчеркиванием, в строку, разделенную пробелами, с заглавной первой буквой каждого слова:

    use Illuminate\Support\Str;

    $headline = Str::headline('steve_jobs');

    // Steve Jobs

    $headline = Str::headline('EmailNotificationSent');

    // Email Notification Sent

<a name="method-str-is"></a>
#### `Str::is()`

Метод `Str::is` определяет, соответствует ли переданная строка указанному шаблону. Допускается использование метасимвола подстановки `*`:

    use Illuminate\Support\Str;

    $matches = Str::is('foo*', 'foobar');

    // true

    $matches = Str::is('baz*', 'foobar');

    // false

<a name="method-str-is-ascii"></a>
#### `Str::isAscii()`

Метод `Str::isAscii` определяет, является ли переданная строка 7-битной ASCII:

    use Illuminate\Support\Str;

    $isAscii = Str::isAscii('Taylor');

    // true

    $isAscii = Str::isAscii('ü');

    // false

<a name="method-str-is-json"></a>
#### `Str::isJson()`

Метод `Str::isJson` определяет, является ли переданная строка корректно закодированной в формат JSON:

    use Illuminate\Support\Str;

    $result = Str::isJson('[1,2,3]');

    // true

    $result = Str::isJson('{"first": "John", "last": "Doe"}');

    // true

    $result = Str::isJson('{first: "John", last: "Doe"}');

    // false

<a name="method-str-is-uuid"></a>
#### `Str::isUuid()`

Метод `Str::isUuid` определяет, является ли переданная строка допустимым UUID:

    use Illuminate\Support\Str;

    $isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

    // true

    $isUuid = Str::isUuid('laravel');

    // false

<a name="method-kebab-case"></a>
#### `Str::kebab()`

Метод `Str::kebab` преобразует переданную строку в `kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::kebab('fooBar');

    // foo-bar

<a name="method-str-lcfirst"></a>
#### `Str::lcfirst()`

Метод `Str::lcfirst` возвращает переданную строку с первым символом в нижнем регистре:

    use Illuminate\Support\Str;

    $string = Str::lcfirst('Foo Bar');

    // foo Bar

<a name="method-str-length"></a>
#### `Str::length()`

Метод `Str::length` возвращает длину переданной строки:

    use Illuminate\Support\Str;

    $length = Str::length('Laravel');

    // 7

<a name="method-str-limit"></a>
#### `Str::limit()`

Метод `Str::limit` усекает переданную строку до указанной длины:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

Вы также можете передать третий строковый аргумент, содержимое которого будет добавлено в конец:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-lower"></a>
#### `Str::lower()`

Метод `Str::lower` преобразует переданную строку в нижний регистр:

    use Illuminate\Support\Str;

    $converted = Str::lower('LARAVEL');

    // laravel

<a name="method-str-markdown"></a>
#### `Str::markdown()`

Метод `Str::markdown` конвертирует текст с разметкой [GitHub flavored Markdown](https://github.github.com/gfm/) в HTML с помощью [CommonMark](https://commonmark.thephpleague.com/):

    use Illuminate\Support\Str;

    $html = Str::markdown('# Laravel');

    // <h1>Laravel</h1>

    $html = Str::markdown('# Taylor <b>Otwell</b>', [
        'html_input' => 'strip',
    ]);

    // <h1>Taylor Otwell</h1>

<a name="method-str-mask"></a>
#### `Str::mask()`

Метод `Str::mask` маскирует часть строки повторяющимся символом и может использоваться для [обфускации](https://ru.wikipedia.org/wiki/Обфускация_(программное_обеспечение)) сегментов строк, таких как адреса электронной почты и номера телефонов:

    use Illuminate\Support\Str;

    $string = Str::mask('taylor@example.com', '*', 3);

    // tay***************

При необходимости вы можете указать отрицательное число в качестве третьего аргумента метода `mask`, который даст указание методу начать маскировку на заданном расстоянии от конца строки:

    $string = Str::mask('taylor@example.com', '*', -15, 3);

    // tay***@example.com

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()`

Метод `Str::orderedUuid` генерирует UUID с «префиксом временной метки», который может быть эффективно сохранен в индексированном столбце базы данных. Каждый UUID, созданный с помощью этого метода, будет отсортирован после UUID, ранее созданных с помощью этого метода:

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-padboth"></a>
#### `Str::padBoth()`

Метод `Str::padBoth` оборачивает функцию `str_pad` PHP, заполняя обе стороны строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::padBoth('James', 10, '_');

    // '__James___'

    $padded = Str::padBoth('James', 10);

    // '  James   '

<a name="method-str-padleft"></a>
#### `Str::padLeft()`

Метод `Str::padLeft` оборачивает функцию `str_pad` PHP, заполняя левую часть строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::padLeft('James', 10, '-=');

    // '-=-=-James'

    $padded = Str::padLeft('James', 10);

    // '     James'

<a name="method-str-padright"></a>
#### `Str::padRight()`

Метод `Str::padRight` оборачивает функцию `str_pad` PHP, заполняя правую часть строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::padRight('James', 10, '-');

    // 'James-----'

    $padded = Str::padRight('James', 10);

    // 'James     '

<a name="method-str-plural"></a>
#### `Str::plural()`

Метод `Str::plural` преобразует слово в форму множественного числа. Этот метод поддерживает [любой из языков, доступных построителю слов во множественном числе Laravel](localization.md#pluralization-language):

    use Illuminate\Support\Str;

    $plural = Str::plural('car');

    // cars

    $plural = Str::plural('child');

    // children

Вы можете передать целое число в качестве второго аргумента метода для получения строки в единственном или множественном числе:

    use Illuminate\Support\Str;

    $plural = Str::plural('child', 2);

    // children

    $singular = Str::plural('child', 1);

    // child

<a name="method-str-plural-studly"></a>
#### `Str::pluralStudly()`

Метод `Str::pluralStudly` преобразует строку единственного числа формата `StudlyCase` в форму множественного числа. Этот метод поддерживает [любой из языков, доступных построителю слов во множественном числе Laravel](localization.md#pluralization-language):

    use Illuminate\Support\Str;

    $plural = Str::pluralStudly('VerifiedHuman');

    // VerifiedHumans

    $plural = Str::pluralStudly('UserFeedback');

    // UserFeedback

Вы можете передать целое число в качестве второго аргумента метода для получения строки в единственном или множественном числе:

    use Illuminate\Support\Str;

    $plural = Str::pluralStudly('VerifiedHuman', 2);

    // VerifiedHumans

    $singular = Str::pluralStudly('VerifiedHuman', 1);

    // VerifiedHuman

<a name="method-str-random"></a>
#### `Str::random()`

Метод `Str::random` генерирует случайную строку указанной длины. Этот метод использует функцию `random_bytes` PHP:

    use Illuminate\Support\Str;

    $random = Str::random(40);

<a name="method-str-remove"></a>
#### `Str::remove()`

Метод `Str::remove` удаляет указанную подстроку или массив подстрок в строке:

    use Illuminate\Support\Str;

    $string = 'Peter Piper picked a peck of pickled peppers.';

    $removed = Str::remove('e', $string);

    // Ptr Pipr pickd a pck of pickld ppprs.

Вы можете передать `false` в качестве третьего аргумента для игнорирования регистра удаляемых подстрок.

<a name="method-str-replace"></a>
#### `Str::replace()`

Метод `Str::replace` заменяет указанное значение в строке:

    use Illuminate\Support\Str;

    $string = 'Laravel 8.x';

    $replaced = Str::replace('8.x', '9.x', $string);

    // Laravel 9.x

<a name="method-str-replace-array"></a>
#### `Str::replaceArray()`

Метод `Str::replaceArray` последовательно заменяет указанное значение в строке, используя массив:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `Str::replaceFirst()`

Метод `Str::replaceFirst` заменяет первое вхождение переданного значения в строке:

    use Illuminate\Support\Str;

    $replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `Str::replaceLast()`

Метод `Str::replaceLast` заменяет последнее вхождение переданного значения в строке:

    use Illuminate\Support\Str;

    $replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<!--  -->
<a name="method-str-reverse"></a>
#### `Str::reverse()`

Метод `Str::reverse` переворачивает переданную строку:

    use Illuminate\Support\Str;

    $reversed = Str::reverse('Hello World');

    // dlroW olleH

<a name="method-str-singular"></a>
#### `Str::singular()`

Метод `Str::singular` преобразует слово в форму единственного числа. Этот метод поддерживает [любой из языков, доступных построителю слов во множественном числе Laravel](localization.md#pluralization-language):

    use Illuminate\Support\Str;

    $singular = Str::singular('cars');

    // car

    $singular = Str::singular('children');

    // child

<a name="method-str-slug"></a>
#### `Str::slug()`

Метод `Str::slug` создает «дружественный фрагмент» URL-адреса из переданной строки:

    use Illuminate\Support\Str;

    $slug = Str::slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-snake-case"></a>
#### `Str::snake()`

Метод `Str::snake` преобразует переданную строку в `snake_case`:

    use Illuminate\Support\Str;

    $converted = Str::snake('fooBar');

    // foo_bar

    $converted = Str::snake('fooBar', '-');

    // foo-bar

<a name="method-str-squish"></a>
#### `Str::squish()`

Метод `Str::squish` удаляет все лишние пробелы из строки, включая лишние пробелы между словами:

    use Illuminate\Support\Str;

    $string = Str::squish('    laravel    framework    ');

    // laravel framework

<a name="method-str-start"></a>
#### `Str::start()`

Метод `Str::start` добавляет один экземпляр указанного значения в переданную строку, если она еще не начинается этим значением:

    use Illuminate\Support\Str;

    $adjusted = Str::start('this/string', '/');

    // /this/string

    $adjusted = Str::start('/this/string', '/');

    // /this/string

<a name="method-starts-with"></a>
#### `Str::startsWith()`

Метод `Str::startsWith` определяет, начинается ли переданная строка с указанного значения:

    use Illuminate\Support\Str;

    $result = Str::startsWith('This is my name', 'This');

    // true

Если передан массив возможных значений, то метод `startsWith` вернет `true`, если строка начинается с любого из указанных значений:

    $result = Str::startsWith('This is my name', ['This', 'That', 'There']);

    // true

<a name="method-studly-case"></a>
#### `Str::studly()`

Метод `Str::studly` преобразует переданную строку в `StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::studly('foo_bar');

    // FooBar

<a name="method-str-substr"></a>
#### `Str::substr()`

Метод `Str::substr` возвращает часть строки, заданную параметрами «начало» и «длина»:

    use Illuminate\Support\Str;

    $converted = Str::substr('The Laravel Framework', 4, 7);

    // Laravel

<a name="method-str-substrcount"></a>
#### `Str::substrCount()`

Метод `Str::substrCount` возвращает число вхождений подстроки в строку:

    use Illuminate\Support\Str;

    $count = Str::substrCount('If you like ice cream, you will like snow cones.', 'like');

    // 2

<a name="method-str-substrreplace"></a>
#### `Str::substrReplace()`

Метод `Str::substrReplace` заменяет часть строки, начиная с позиции, указанной третьим аргументом, и заменяет число символов, указанное четвертым аргументом. Передача `0` в четвертый аргумент метода вставит строку в указанную позицию без замены каких-либо существующих символов в строке:

    use Illuminate\Support\Str;

    $result = Str::substrReplace('1300', ':', 2);
    // 13:

    $result = Str::substrReplace('1300', ':', 2, 0);
    // 13:00

<a name="method-str-swap"></a>
#### `Str::swap()`

Метод `Str::swap` заменяет несколько значений в переданной строке, используя функцию `strtr` PHP:

    use Illuminate\Support\Str;

    $string = Str::swap([
        'Tacos' => 'Burritos',
        'great' => 'fantastic',
    ], 'Tacos are great!');

    // Burritos are fantastic!

<a name="method-title-case"></a>
#### `Str::title()`

Метод `Str::title` преобразует переданную строку в `Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-to-html-string"></a>
#### `Str::toHtmlString()`

Метод `Str::toHtmlString` преобразует экземпляр строки в экземпляр `Illuminate\Support\HtmlString`, который может быть отображен в шаблонах Blade:

    use Illuminate\Support\Str;

    $htmlString = Str::of('Nuno Maduro')->toHtmlString();

<a name="method-str-ucfirst"></a>
#### `Str::ucfirst()`

Метод `Str::ucfirst` возвращает переданную строку с первой заглавной буквой:

    use Illuminate\Support\Str;

    $string = Str::ucfirst('foo bar');

    // Foo bar

<a name="method-str-ucsplit"></a>
#### `Str::ucsplit()`

Метод `Str::ucsplit` разбивает переданную строку на массив по символам верхнего регистра:

    use Illuminate\Support\Str;

    $segments = Str::ucsplit('FooBar');

    // [0 => 'Foo', 1 => 'Bar']

<a name="method-str-upper"></a>
#### `Str::upper()`

Метод `Str::upper` преобразует переданную строку в верхний регистр:

    use Illuminate\Support\Str;

    $string = Str::upper('laravel');

    // LARAVEL

<a name="method-str-uuid"></a>
#### `Str::uuid()`

Метод `Str::uuid` генерирует UUID (версия 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="method-str-word-count"></a>
#### `Str::wordCount()`

Метод `Str::wordCount` возвращает количество слов, содержащихся в строке:

```php
use Illuminate\Support\Str;

Str::wordCount('Hello, world!'); // 2
```

<a name="method-str-words"></a>
#### `Str::words()`

Метод `Str::words` ограничивает количество слов в строке. Дополнительная строка может быть передана этому методу через его третий аргумент, чтобы указать, какая строка должна быть добавлена в конец усеченной строки:

    use Illuminate\Support\Str;

    return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

    // Perfectly balanced, as >>>

<a name="method-str"></a>
#### `str()`

Функция `str` возвращает новый экземпляр `Illuminate\Support\Stringable` переданной строки. Эта функция эквивалентна методу `Str::of`:

    $string = str('Taylor')->append(' Otwell');

    // 'Taylor Otwell'

Если для функции `str` не указан аргумент, то функция возвращает экземпляр `Illuminate\Support\Str`:

    $snake = str()->snake('FooBar');

    // 'foo_bar'

<a name="method-trans"></a>
#### `trans()`

Функция `trans` переводит переданный ключ перевода, используя ваши [файлы локализации](localization.md):

    echo trans('messages.welcome');

Если указанный ключ перевода не существует, функция `trans` вернет данный ключ. Итак, используя приведенный выше пример, функция `trans` вернет `messages.welcome`, если ключ перевода не существует.

<a name="method-trans-choice"></a>
#### `trans_choice()`

Функция `trans_choice` переводит заданный ключ перевода с изменением формы слова:

    echo trans_choice('messages.notifications', $unreadCount);

Если указанный ключ перевода не существует, функция `trans_choice` вернет данный ключ. Итак, используя приведенный выше пример, функция `trans_choice` вернет `messages.notifications`, если ключ перевода не существует.

<a name="fluent-strings"></a>
## Строки Fluent

Строки Fluent обеспечивают более гибкий объектно-ориентированный интерфейс для работы со строковыми значениями, позволяя объединять несколько строковых операций вместе с использованием более удобочитаемого синтаксиса по сравнению с традиционными строковыми операциями.

<a name="method-fluent-str-after"></a>
#### `after`

Метод `after` возвращает все после переданного значения в строке. Вся строка будет возвращена, если значение не существует в строке:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->after('This is');

    // ' my name'

<a name="method-fluent-str-after-last"></a>
#### `afterLast`

Метод `afterLast` возвращает все после последнего вхождения переданного значения в строке. Вся строка будет возвращена, если значение не существует в строке:

    use Illuminate\Support\Str;

    $slice = Str::of('App\Http\Controllers\Controller')->afterLast('\\');

    // 'Controller'

<a name="method-fluent-str-append"></a>
#### `append`

Метод `append` добавляет указанные значения в строку:

    use Illuminate\Support\Str;

    $string = Str::of('Taylor')->append(' Otwell');

    // 'Taylor Otwell'

<a name="method-fluent-str-ascii"></a>
#### `ascii`

Метод `ascii` попытается транслитерировать строку в значение ASCII:

    use Illuminate\Support\Str;

    $string = Str::of('ü')->ascii();

    // 'u'

<a name="method-fluent-str-basename"></a>
#### `basename`

Метод `basename` вернет завершающий компонент имени переданной строки:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->basename();

    // 'baz'

При необходимости вы можете указать «расширение», которое будет удалено из завершающего компонента:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz.jpg')->basename('.jpg');

    // 'baz'

<a name="method-fluent-str-before"></a>
#### `before`

Метод `before` возвращает все до указанного значения в строке:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->before('my name');

    // 'This is '

<a name="method-fluent-str-before-last"></a>
#### `beforeLast`

Метод `beforeLast` возвращает все до последнего вхождения переданного значения в строку:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->beforeLast('is');

    // 'This '

<a name="method-fluent-str-between"></a>
#### `between`

Метод `between` возвращает часть строки между двумя значениями:

    use Illuminate\Support\Str;

    $converted = Str::of('This is my name')->between('This', 'name');

    // ' is my '

<a name="method-fluent-str-between-first"></a>
#### `betweenFirst`

Метод `betweenFirst` возвращает наименьшую возможную часть строки между двумя значениями:

    use Illuminate\Support\Str;

    $converted = Str::of('[a] bc [d]')->betweenFirst('[', ']');

    // 'a'

<a name="method-fluent-str-camel"></a>
#### `camel`

Метод `camel` преобразует переданную строку в `camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->camel();

    // fooBar

<a name="method-fluent-str-class-basename"></a>
#### `classBasename`

Метод `classBasename` возвращает имя класса переданного класса с удаленным пространством имен класса:

    use Illuminate\Support\Str;

    $class = Str::of('Foo\Bar\Baz')->classBasename();

    // Baz

<a name="method-fluent-str-contains"></a>
#### `contains`

Метод `contains` определяет, содержит ли переданная строка указанное значение (с учетом регистра):

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains('my');

    // true

Вы также можете указать массив значений, чтобы определить, содержит ли переданная строка какое-либо из этих значений:

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains(['my', 'foo']);

    // true

<a name="method-fluent-str-contains-all"></a>
#### `containsAll`

Метод `containsAll` определяет, содержит ли переданная строка все значения массива:

    use Illuminate\Support\Str;

    $containsAll = Str::of('This is my name')->containsAll(['my', 'name']);

    // true

<a name="method-fluent-str-dirname"></a>
#### `dirname`

Метод `dirname` возвращает родительскую часть директории переданной строки:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname();

    // '/foo/bar'

При желании вы можете указать, сколько уровней каталогов вы хотите вырезать из строки:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname(2);

    // '/foo'

<a name="method-fluent-str-excerpt"></a>
#### `excerpt`

Метод `excerpt` извлекает отрывок из переданной строки, который соответствует первому экземпляру фразы в этой строке:

    use Illuminate\Support\Str;

    $excerpt = Str::of('This is my name')->excerpt('my', [
        'radius' => 3
    ]);

    // '...is my na...'

Параметр `radius`, который по умолчанию равен `100`, позволяет вам определить количество символов, которые должны присутствовать с каждой стороны усеченной строки.

Кроме того, вы можете использовать параметр `omission`, чтобы определить строку, которая будет добавлена к усеченной строке:

    use Illuminate\Support\Str;

    $excerpt = Str::of('This is my name')->excerpt('name', [
        'radius' => 3,
        'omission' => '(...) '
    ]);

    // '(...) my name'

<a name="method-fluent-str-ends-with"></a>
#### `endsWith`

Метод `endsWith` определяет, заканчивается ли переданная строка указанным значением:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith('name');

    // true

Вы также можете указать массив значений, чтобы определить, заканчивается ли переданная строка каким-либо из указанных значений:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith(['name', 'foo']);

    // true

    $result = Str::of('This is my name')->endsWith(['this', 'foo']);

    // false

<a name="method-fluent-str-exactly"></a>
#### `exactly`

Метод `exactly` определяет, является ли переданная строка точным совпадением с другой строкой:

    use Illuminate\Support\Str;

    $result = Str::of('Laravel')->exactly('Laravel');

    // true

<a name="method-fluent-str-explode"></a>
#### `explode`

Метод `explode` разделяет строку по заданному разделителю и возвращает коллекцию, содержащую каждый раздел строки разбиения:

    use Illuminate\Support\Str;

    $collection = Str::of('foo bar baz')->explode(' ');

    // collect(['foo', 'bar', 'baz'])

<a name="method-fluent-str-finish"></a>
#### `finish`

Метод `finish` добавляет один экземпляр указанного значения в переданную строку, если она еще не заканчивается этим значением:

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->finish('/');

    // this/string/

    $adjusted = Str::of('this/string/')->finish('/');

    // this/string/

<a name="method-fluent-str-is"></a>
#### `is`

Метод `is` определяет, соответствует ли переданная строка указанному шаблону. Допускается использование метасимвола подстановки `*`:

    use Illuminate\Support\Str;

    $matches = Str::of('foobar')->is('foo*');

    // true

    $matches = Str::of('foobar')->is('baz*');

    // false

<a name="method-fluent-str-is-ascii"></a>
#### `isAscii`

Метод `isAscii` определяет, является ли переданная строка строкой ASCII:

    use Illuminate\Support\Str;

    $result = Str::of('Taylor')->isAscii();

    // true

    $result = Str::of('ü')->isAscii();

    // false

<a name="method-fluent-str-is-empty"></a>
#### `isEmpty`

Метод `isEmpty` определяет, является ли переданная строка пустой:

    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isEmpty();

    // true

    $result = Str::of('Laravel')->trim()->isEmpty();

    // false

<a name="method-fluent-str-is-not-empty"></a>
#### `isNotEmpty`

Метод `isNotEmpty` определяет, является ли переданная строка не пустой:


    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isNotEmpty();

    // false

    $result = Str::of('Laravel')->trim()->isNotEmpty();

    // true

<a name="method-fluent-str-is-json"></a>
#### `isJson`

Метод `isJson` определяет, является ли переданная строка корректно закодированной в формат JSON:

    use Illuminate\Support\Str;

    $result = Str::of('[1,2,3]')->isJson();

    // true

    $result = Str::of('{"first": "John", "last": "Doe"}')->isJson();

    // true

    $result = Str::of('{first: "John", last: "Doe"}')->isJson();

    // false

<a name="method-fluent-str-is-uuid"></a>
#### `isUuid`

Метод `isUuid` определяет, является ли переданная строка строкой в формате UUID:

    use Illuminate\Support\Str;

    $result = Str::of('5ace9ab9-e9cf-4ec6-a19d-5881212a452c')->isUuid();

    // true

    $result = Str::of('Taylor')->isUuid();

    // false

<a name="method-fluent-str-kebab"></a>
#### `kebab`

Метод `kebab` преобразует переданную строку в `kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->kebab();

    // foo-bar

<a name="method-fluent-str-lcfirst"></a>
#### `lcfirst`

Метод `lcfirst` возвращает переданную строку с первым символом в нижнем регистре:

    use Illuminate\Support\Str;

    $string = Str::of('Foo Bar')->lcfirst();

    // foo Bar

<!--  -->
<a name="method-fluent-str-length"></a>
#### `length`

Метод `length` возвращает длину переданной строки:

    use Illuminate\Support\Str;

    $length = Str::of('Laravel')->length();

    // 7

<a name="method-fluent-str-limit"></a>
#### `limit`

Метод `limit` усекает переданную строку до указанной длины:

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20);

    // The quick brown fox...

Вы также можете передать второй строковый аргумент, содержимое которого будет добавлено в конец:

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20, ' (...)');

    // The quick brown fox (...)

<a name="method-fluent-str-lower"></a>
#### `lower`

Метод `lower` преобразует переданную строку в нижний регистр:

    use Illuminate\Support\Str;

    $result = Str::of('LARAVEL')->lower();

    // 'laravel'

<a name="method-fluent-str-ltrim"></a>
#### `ltrim`

Метод `ltrim` удаляет символы из начала строки:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->ltrim();

    // 'Laravel  '

    $string = Str::of('/Laravel/')->ltrim('/');

    // 'Laravel/'

<a name="method-fluent-str-markdown"></a>
#### `markdown`

Метод `markdown` конвертирует текст с разметкой [GitHub flavored Markdown](https://github.github.com/gfm/) в HTML:

    use Illuminate\Support\Str;

    $html = Str::of('# Laravel')->markdown();

    // <h1>Laravel</h1>

    $html = Str::of('# Taylor <b>Otwell</b>')->markdown([
        'html_input' => 'strip',
    ]);

    // <h1>Taylor Otwell</h1>

<a name="method-fluent-str-mask"></a>
#### `mask`

Метод `mask` маскирует часть строки повторяющимся символом и может использоваться для [обфускации](https://ru.wikipedia.org/wiki/Обфускация_(программное_обеспечение)) сегментов строк, таких как адреса электронной почты и номера телефонов:

    use Illuminate\Support\Str;

    $string = Str::of('taylor@example.com')->mask('*', 3);

    // tay***************

При необходимости вы можете указать отрицательное число в качестве третьего аргумента метода `mask`, который даст указание методу начать маскировку на заданном расстоянии от конца строки:

    $string = Str::of('taylor@example.com')->mask('*', -15, 3);

    // tay***@example.com

<a name="method-fluent-str-match"></a>
#### `match`

Метод `match` вернет часть строки, которая соответствует указанному шаблону регулярного выражения:

    use Illuminate\Support\Str;

    $result = Str::of('foo bar')->match('/bar/');

    // 'bar'

    $result = Str::of('foo bar')->match('/foo (.*)/');

    // 'bar'

<a name="method-fluent-str-match-all"></a>
#### `matchAll`

Метод `matchAll` вернет коллекцию, содержащую части строки, которые соответствуют указанному шаблону регулярного выражения:

    use Illuminate\Support\Str;

    $result = Str::of('bar foo bar')->matchAll('/bar/');

    // collect(['bar', 'bar'])

Если вы укажете группировку в выражении, то Laravel вернет коллекцию совпадений этой группы:

    use Illuminate\Support\Str;

    $result = Str::of('bar fun bar fly')->matchAll('/f(\w*)/');

    // collect(['un', 'ly']);

Если совпадений не найдено, то будет возвращена пустая коллекция.

<a name="method-fluent-str-new-line"></a>
#### `newLine`

Метод `newLine` добавляет к строке символ «конца строки»:

    use Illuminate\Support\Str;

    $padded = Str::of('Laravel')->newLine()->append('Framework');

    // 'Laravel
    //  Framework'

<a name="method-fluent-str-padboth"></a>
#### `padBoth`

Метод `padBoth` оборачивает функцию `str_pad` PHP, заполняя обе стороны строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padBoth(10, '_');

    // '__James___'

    $padded = Str::of('James')->padBoth(10);

    // '  James   '

<a name="method-fluent-str-padleft"></a>
#### `padLeft`

Метод `padLeft` оборачивает функцию `str_pad` PHP, заполняя левую часть строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padLeft(10, '-=');

    // '-=-=-James'

    $padded = Str::of('James')->padLeft(10);

    // '     James'

<a name="method-fluent-str-padright"></a>
#### `padRight`

Метод `padRight` оборачивает функцию `str_pad` PHP, заполняя правую часть строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padRight(10, '-');

    // 'James-----'

    $padded = Str::of('James')->padRight(10);

    // 'James     '

<a name="method-fluent-str-pipe"></a>
#### `pipe`

Метод `pipe` позволяет вам преобразовать строку, передав ее текущее значение указанной функции обратного вызова:

    use Illuminate\Support\Str;

    $hash = Str::of('Laravel')->pipe('md5')->prepend('Checksum: ');

    // 'Checksum: a5c95b86291ea299fcbe64458ed12702'

    $closure = Str::of('foo')->pipe(function ($str) {
        return 'bar';
    });

    // 'bar'

<a name="method-fluent-str-plural"></a>
#### `plural`

Метод `plural` преобразует слово в форму множественного числа. Этот метод поддерживает [любой из языков, доступных построителю слов во множественном числе Laravel](localization.md#pluralization-language):

    use Illuminate\Support\Str;

    $plural = Str::of('car')->plural();

    // cars

    $plural = Str::of('child')->plural();

    // children

Вы можете передать целое число в качестве второго аргумента метода для получения строки в единственном или множественном числе:

    use Illuminate\Support\Str;

    $plural = Str::of('child')->plural(2);

    // children

    $plural = Str::of('child')->plural(1);

    // child

<a name="method-fluent-str-prepend"></a>
#### `prepend`

Метод `prepend` добавляет указанные значения в начало строки:

    use Illuminate\Support\Str;

    $string = Str::of('Framework')->prepend('Laravel ');

    // Laravel Framework

<a name="method-fluent-str-remove"></a>
#### `remove`

Метод `remove` удаляет указанную подстроку или массив подстрок в строке:

    use Illuminate\Support\Str;

    $string = Str::of('Arkansas is quite beautiful!')->remove('quite');

    // Arkansas is beautiful!

Вы можете передать `false` в качестве второго аргумента для игнорирования регистра удаляемых подстрок.

<a name="method-fluent-str-replace"></a>
#### `replace`

Метод `replace` заменяет указанную строку внутри строки:

    use Illuminate\Support\Str;

    $replaced = Str::of('Laravel 6.x')->replace('6.x', '7.x');

    // Laravel 7.x

<a name="method-fluent-str-replace-array"></a>
#### `replaceArray`

Метод `replaceArray` последовательно заменяет указанное значение в строке, используя массив:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::of($string)->replaceArray('?', ['8:30', '9:00']);

    // The event will take place between 8:30 and 9:00

<a name="method-fluent-str-replace-first"></a>
#### `replaceFirst`

Метод `replaceFirst` заменяет первое вхождение указанного значения в строке:

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceFirst('the', 'a');

    // a quick brown fox jumps over the lazy dog

<a name="method-fluent-str-replace-last"></a>
#### `replaceLast`

Метод `replaceLast` заменяет последнее вхождение указанного значения в строке:

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceLast('the', 'a');

    // the quick brown fox jumps over a lazy dog

<a name="method-fluent-str-replace-matches"></a>
#### `replaceMatches`

Метод `replaceMatches` заменяет все части строки, соответствующие указанному шаблону, переданной строки:

    use Illuminate\Support\Str;

    $replaced = Str::of('(+1) 501-555-1000')->replaceMatches('/[^A-Za-z0-9]++/', '')

    // '15015551000'

Метод `replaceMatches` также принимает замыкание, которое будет вызвано для каждой части строки, соответствующей шаблону, что позволяет вам выполнять логику замены в замыкании и возвращать замененное значение:

    use Illuminate\Support\Str;

    $replaced = Str::of('123')->replaceMatches('/\d/', function ($match) {
        return '['.$match[0].']';
    });

    // '[1][2][3]'

<a name="method-fluent-str-rtrim"></a>
#### `rtrim`

Метод `rtrim` удаляет символы из конца строки:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->rtrim();

    // '  Laravel'

    $string = Str::of('/Laravel/')->rtrim('/');

    // '/Laravel'

<a name="method-fluent-str-scan"></a>
#### `scan`

Метод `scan` преобразует входные данные из строки в коллекцию в соответствии с форматом, поддерживаемым [функцией `sscanf` PHP](https://www.php.net/manual/ru/function.sscanf.php):

    use Illuminate\Support\Str;

    $collection = Str::of('filename.jpg')->scan('%[^.].%s');

    // collect(['filename', 'jpg'])

<a name="method-fluent-str-singular"></a>
#### `singular`

Метод `singular` преобразует слово в форму единственного числа. Этот метод поддерживает [любой из языков, доступных построителю слов во множественном числе Laravel](localization.md#pluralization-language):

    use Illuminate\Support\Str;

    $singular = Str::of('cars')->singular();

    // car

    $singular = Str::of('children')->singular();

    // child

<a name="method-fluent-str-slug"></a>
#### `slug`

Метод `slug` создает «дружественный фрагмент» URL-адреса из переданной строки:

    use Illuminate\Support\Str;

    $slug = Str::of('Laravel Framework')->slug('-');

    // laravel-framework

<a name="method-fluent-str-snake"></a>
#### `snake`

Метод `snake` преобразует переданную строку в `snake_case`:

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->snake();

    // foo_bar

<a name="method-fluent-str-split"></a>
#### `split`

Метод `split` разбивает строку на коллекцию с помощью регулярного выражения:

    use Illuminate\Support\Str;

    $segments = Str::of('one, two, three')->split('/[\s,]+/');

    // collect(["one", "two", "three"])

<a name="method-fluent-str-squish"></a>
#### `squish`

Метод `squish` удаляет все лишние пробелы из строки, включая лишние пробелы между словами:

    use Illuminate\Support\Str;

    $string = Str::of('    laravel    framework    ')->squish();

    // laravel framework

<a name="method-fluent-str-start"></a>
#### `start`

Метод `start` добавляет один экземпляр указанного значения в переданную строку, если она еще не начинается этим значением:

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->start('/');

    // /this/string

    $adjusted = Str::of('/this/string')->start('/');

    // /this/string

<a name="method-fluent-str-starts-with"></a>
#### `startsWith`

Метод `startsWith` определяет, начинается ли переданная строка с указанного значения:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->startsWith('This');

    // true

<a name="method-fluent-str-studly"></a>
#### `studly`

Метод `studly` преобразует переданную строку в `StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->studly();

    // FooBar

<a name="method-fluent-str-substr"></a>
#### `substr`

Метод `substr` возвращает часть строки, заданную параметрами «начало» и «длина»:

    use Illuminate\Support\Str;

    $string = Str::of('Laravel Framework')->substr(8);

    // Framework

    $string = Str::of('Laravel Framework')->substr(8, 5);

    // Frame

<a name="method-fluent-str-substrreplace"></a>
#### `substrReplace`

Метод `substrReplace` заменяет часть строки, начиная с позиции, указанной вторым аргументом, и заменяет число символов, указанное третьим аргументом. Передача `0` в третий аргумент метода вставит строку в указанную позицию без замены каких-либо существующих символов в строке:

    use Illuminate\Support\Str;

    $string = Str::of('1300')->substrReplace(':', 2);

    // 13:

    $string = Str::of('The Framework')->substrReplace(' Laravel', 3, 0);

    // The Laravel Framework

<a name="method-fluent-str-swap"></a>
#### `swap`

Метод `swap` заменяет несколько значений в переданной строке, используя функцию `strtr` PHP:

    use Illuminate\Support\Str;

    $string = Str::of('Tacos are great!')
        ->swap([
            'Tacos' => 'Burritos',
            'great' => 'fantastic',
        ]);

    // Burritos are fantastic!

<a name="method-fluent-str-tap"></a>
#### `tap`

Метод `tap` передает строку заданному замыканию, позволяя вам взаимодействовать с ней, не затрагивая при этом саму строку. Исходная строка возвращается методом `tap` независимо от того, что возвращает замыкание:

    use Illuminate\Support\Str;

    $string = Str::of('Laravel')
        ->append(' Framework')
        ->tap(function ($string) {
            dump('String after append: ' . $string);
        })
        ->upper();

    // LARAVEL FRAMEWORK

<a name="method-fluent-str-test"></a>
#### `test`

Метод `test` определяет, соответствует ли строка переданному шаблону регулярного выражения:

    use Illuminate\Support\Str;

    $result = Str::of('Laravel Framework')->test('/Laravel/');

    // true

<a name="method-fluent-str-title"></a>
#### `title`

Метод `title` преобразует переданную строку в `Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::of('a nice title uses the correct case')->title();

    // A Nice Title Uses The Correct Case

<a name="method-fluent-str-trim"></a>
#### `trim`

Метод `trim` обрезает переданную строку:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->trim();

    // 'Laravel'

    $string = Str::of('/Laravel/')->trim('/');

    // 'Laravel'

<a name="method-fluent-str-ucfirst"></a>
#### `ucfirst`

Метод `ucfirst` возвращает переданную строку с первой заглавной буквой:

    use Illuminate\Support\Str;

    $string = Str::of('foo bar')->ucfirst();

    // Foo bar

<a name="method-fluent-str-ucsplit"></a>
#### `ucsplit`

Метод `ucsplit` разбивает переданную строку на коллекцию по символам верхнего регистра:

    use Illuminate\Support\Str;

    $string = Str::of('Foo Bar')->ucsplit();

    // collect(['Foo', 'Bar'])

<a name="method-fluent-str-upper"></a>
#### `upper`

Метод `upper` преобразует переданную строку в верхний регистр:

    use Illuminate\Support\Str;

    $adjusted = Str::of('laravel')->upper();

    // LARAVEL

<a name="method-fluent-str-when"></a>
#### `when`

Метод `when` вызывает указанное замыкание, если переданное условие истинно. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('Taylor')
                    ->when(true, function ($string) {
                        return $string->append(' Otwell');
                    });

    // 'Taylor Otwell'

При необходимости вы можете передать другое замыкание в качестве третьего параметра методу `when`. Это замыкание будет выполнено, если параметр условия оценивается как `false`.

<a name="method-fluent-str-when-contains"></a>
#### `whenContains`

Метод `whenContains` вызывает указанное замыкание, если строка содержит переданное значение. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('tony stark')
                ->whenContains('tony', function ($string) {
                    return $string->title();
                });

    // 'Tony Stark'

При необходимости вы можете передать другое замыкание в качестве третьего параметра метода `whenContains`. Это замыкание будет выполнено, если строка не содержит переданного значения.

Вы также можете передать массив значений, чтобы определить, содержит ли переданная строка какие-либо значения в массиве:

    use Illuminate\Support\Str;

    $string = Str::of('tony stark')
                ->whenContains(['tony', 'hulk'], function ($string) {
                    return $string->title();
                });

    // Tony Stark

<a name="method-fluent-str-when-contains-all"></a>
#### `whenContainsAll`

Метод `whenContainsAll` вызывает указанное замыкание, если строка содержит все переданные подстроки. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('tony stark')
                    ->whenContainsAll(['tony', 'stark'], function ($string) {
                        return $string->title();
                    });

    // 'Tony Stark'

При необходимости вы можете передать другое замыкание в качестве третьего параметра методу `whenContainsAll`. Это замыкание будет выполнено, если параметр условия оценивается как `false`.

<a name="method-fluent-str-when-empty"></a>
#### `whenEmpty`

Метод `whenEmpty` вызывает переданное замыкание, если строка пуста. Если замыкание возвращает значение, то это значение будет возвращено методом `whenEmpty`. Если замыкание не возвращает значение, будет возвращен экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('  ')->whenEmpty(function ($string) {
        return $string->trim()->prepend('Laravel');
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-empty"></a>
#### `whenNotEmpty`

Метод `whenNotEmpty` вызывает переданное замыкание, если строка не пуста. Если замыкание возвращает значение, то это значение также будет возвращено методом `whenNotEmpty`. Если замыкание не возвращает значение, будет возвращен Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('Framework')->whenNotEmpty(function ($string) {
        return $string->prepend('Laravel ');
    });

    // 'Laravel Framework'

<a name="method-fluent-str-when-starts-with"></a>
#### `whenStartsWith`

Метод `whenStartsWith` вызывает переданное замыкание, если строка начинается с заданной подстроки. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('disney world')->whenStartsWith('disney', function ($string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-ends-with"></a>
#### `whenEndsWith`

Метод `whenEndsWith` вызывает переданное замыкание, если строка заканчивается заданной подстрокой. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('disney world')->whenEndsWith('world', function ($string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-exactly"></a>
#### `whenExactly`

Метод `whenExactly` вызывает переданное замыкание, если строка точно соответствует заданной строке. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('laravel')->whenExactly('laravel', function ($string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-is"></a>
#### `whenIs`

Метод `whenIs` вызывает переданное замыкание, если строка соответствует заданному шаблону. Допускается использование метасимвола подстановки `*`. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('foo/bar')->whenIs('foo/*', function ($string) {
        return $string->append('/baz');
    });

    // 'foo/bar/baz'

<a name="method-fluent-str-when-is-ascii"></a>
#### `whenIsAscii`

Метод `whenIsAscii` вызывает переданное замыкание, если строка представляет собой 7-битный ASCII. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('foo/bar')->whenIsAscii('laravel', function ($string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-is-uuid"></a>
#### `whenIsUuid`

Метод `whenIsUuid` вызывает переданное замыкание, если строка является допустимым UUID. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('foo/bar')->whenIsUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de', function ($string) {
        return $string->substr(0, 8);
    });

    // 'a0a2a2d2'

<a name="method-fluent-str-when-test"></a>
#### `whenTest`

Метод `whenTest` вызывает переданное замыкание, если строка соответствует заданному регулярному выражению. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;

    $string = Str::of('laravel framework')->whenTest('/laravel/', function ($string) {
        return $string->title();
    });

    // 'Laravel Framework'

<a name="method-str-word-count"></a>
#### `wordCount`

Метод `wordCount` возвращает количество слов, содержащихся в строке:

```php
use Illuminate\Support\Str;

Str::of('Hello, world!')->wordCount(); // 2
```

<a name="method-fluent-str-words"></a>
#### `words`

Метод `words` ограничивает количество слов в строке. Дополнительная строка может быть передана этому методу, чтобы указать, какая строка должна быть добавлена в конец усеченной строки:

    use Illuminate\Support\Str;

    $string = Str::of('Perfectly balanced, as all things should be.')->words(3, ' >>>');

    // Perfectly balanced, as >>>

<a name="urls"></a>
## URL-адреса

<a name="method-action"></a>
#### `action()`

Функция `action` генерирует URL-адрес для переданного действия контроллера:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Если метод принимает параметры маршрута, вы можете передать их как второй аргумент методу:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="method-asset"></a>
#### `asset()`

Функция `asset` генерирует URL для веб-актива, используя текущую схему запроса (HTTP или HTTPS):

    $url = asset('img/photo.jpg');

Вы можете изменить хост URL веб-актива, указав значение переменной `ASSET_URL` в вашем файле `.env`. Это может быть полезно, если вы размещаете свои веб-активы на стороннем сервисе, таком как Amazon S3 или другой CDN:

    // ASSET_URL=http://example.com/assets

    $url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg

<a name="method-route"></a>
#### `route()`

Функция `route` генерирует URL для переданного [именованного маршрута](routing.md#named-routes):

    $url = route('route.name');

Если маршрут принимает параметры, вы можете передать их в качестве второго аргумента методу:

    $url = route('route.name', ['id' => 1]);

По умолчанию функция `route` генерирует абсолютный URL. Если вы хотите создать относительный URL, вы можете передать `false` в качестве третьего аргумента:

    $url = route('route.name', ['id' => 1], false);

<a name="method-secure-asset"></a>
#### `secure_asset()`

Функция `secure_asset` генерирует URL для веб-актива, используя HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-secure-url"></a>
#### `secure_url()`

Функция `secure_url` генерирует полный URL-адрес для указанного пути, используя HTTPS. Дополнительные сегменты URL могут быть переданы во втором аргументе функции:

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-to-route"></a>
#### `to_route()`

Функция `to_route` генерирует [HTTP-ответ перенаправления](responses.md#redirects) на [именованный маршрут](routing.md#named-routes):

    return to_route('users.show', ['user' => 1]);

При необходимости вы можете передать код состояния HTTP, который должен быть назначен перенаправлению, и любые дополнительные заголовки ответа в качестве третьего и четвертого аргументов метода `to_route`, соответственно:

    return to_route('users.show', ['user' => 1], 302, ['X-Framework' => 'Laravel']);

<a name="method-url"></a>
#### `url()`

Функция `url` генерирует полный URL-адрес для указанного пути:

    $url = url('user/profile');

    $url = url('user/profile', [1]);

Если путь не указан, будет возвращен экземпляр `Illuminate\Routing\UrlGenerator`:

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## Разное

<a name="method-abort"></a>
#### `abort()`

Функция `abort` генерирует [HTTP-исключение](errors.md#http-exceptions), которое будет обработано [обработчиком исключения](errors.md#the-exception-handler):

    abort(403);

Вы также можете указать текст ответа исключения и пользовательские заголовки ответа, которые должны быть отправлены в браузер:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()`

Функция `abort_if` генерирует исключение HTTP, если переданное логическое выражение имеет значение `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Подобно методу `abort`, вы также можете указать текст ответа исключения третьим аргументом и массив пользовательских заголовков ответа в качестве четвертого аргумента.

<a name="method-abort-unless"></a>
#### `abort_unless()`

Функция `abort_unless` генерирует исключение HTTP, если переданное логическое выражение оценивается как `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Подобно методу `abort`, вы также можете указать текст ответа исключения третьим аргументом и массив пользовательских заголовков ответа в качестве четвертого аргумента.

<a name="method-app"></a>
#### `app()`

Функция `app` возвращает экземпляр [контейнера служб](container.md):

    $container = app();

Вы можете передать имя класса или интерфейса для извлечения его из контейнера:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()`

Функция `auth` возвращает экземпляр [аутентификатора](authentication.md). Вы можете использовать его вместо фасада `Auth` для удобства:

    $user = auth()->user();

При необходимости вы можете указать, к какому экземпляру охранника вы хотите получить доступ:

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()`

Функция `back` генерирует [HTTP-ответ перенаправления](responses.md#redirects) в предыдущее расположение пользователя:

    return back($status = 302, $headers = [], $fallback = '/');

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()`

Функция `bcrypt` [хеширует](hashing.md) переданное значение, используя Bcrypt. Вы можете использовать его как альтернативу фасаду `Hash`:

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>
#### `blank()`

Функция `blank` проверяет, является ли переданное значение «пустым»:

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

Обратной функции `blank` является функция [`filled`](#method-filled).

<a name="method-broadcast"></a>
#### `broadcast()`

Функция `broadcast` [транслирует](broadcasting.md) переданное [событие](events.md) своим слушателям:

    broadcast(new UserRegistered($user));

    broadcast(new UserRegistered($user))->toOthers();

<a name="method-cache"></a>
#### `cache()`

Функция `cache` используется для получения значений из [кеша](cache.md). Если переданный ключ не существует в кеше, будет возвращено необязательное значение по умолчанию:

    $value = cache('key');

    $value = cache('key', 'default');

Вы можете добавлять элементы в кеш, передавая массив пар ключ / значение в функцию. Вы также должны передать количество секунд или продолжительность актуальности кешированного значения:

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()`

Функция `class_uses_recursive` возвращает все трейты, используемые классом, включая трейты, используемые всеми его родительскими классами:

    $traits = class_uses_recursive(App\Models\User::class);

<a name="method-collect"></a>
#### `collect()`

Функция `collect` создает экземпляр [коллекции](collections.md) переданного значения:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()`

Функция `config` получает значение переменной [конфигурации](configuration.md). Доступ к значениям конфигурации можно получить с помощью «точечной нотации», которое включает имя файла и параметр, к которому вы хотите получить доступ. Значение по умолчанию может быть указано и возвращается, если опция конфигурации не существует:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Вы можете установить переменные конфигурации на время выполнения скрипта, передав массив пар ключ / значение. Однако обратите внимание, что эта функция влияет только на значение конфигурации для текущего запроса и не обновляет фактические значения конфигурации:

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()`

Функция `cookie` создает новый экземпляр [Cookie](requests.md#cookies):

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()`

Функция `csrf_field` генерирует HTML «скрытого» поля ввода, содержащее значение токена CSRF. Например, используя [синтаксис Blade](blade.md):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()`

Функция `csrf_token` возвращает значение текущего токена CSRF:

    $token = csrf_token();

<a name="method-decrypt"></a>
#### `decrypt()`

Функция `decrypt` [расшифрует](encryption.md) переданное значение. Вы можете использовать эту функцию как альтернативу фасаду `Crypt`:

    $password = decrypt($value);

<a name="method-dd"></a>
#### `dd()`

Функция `dd` выводит переданные переменные и завершает выполнение скрипта:

    dd($value);

    dd($value1, $value2, $value3, ...);

Если вы не хотите останавливать выполнение вашего скрипта, используйте вместо этого функцию [`dump`](#method-dump).

<a name="method-dispatch"></a>
#### `dispatch()`

Функция `dispatch` помещает переданное [задание](queues.md#creating-jobs) в [очередь заданий](queues.md) Laravel:

    dispatch(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()`

Функция `dump` выводит переданные переменные:

    dump($value);

    dump($value1, $value2, $value3, ...);

Если вы хотите прекратить выполнение скрипта после вывода переменных, используйте вместо этого функцию [`dd`](#method-dd).

<a name="method-encrypt"></a>
#### `encrypt()`

Функция `encrypt` [зашифрует](encryption.md) переданное значение. Вы можете использовать эту функцию как альтернативу фасаду `Crypt`:

    $secret = encrypt('my-secret-value');

<a name="method-env"></a>
#### `env()`

Функция `env` возвращает значение [переменной окружения](configuration.md#environment-configuration) или значение по умолчанию:

    $env = env('APP_ENV');

    $env = env('APP_ENV', 'production');

> {note} Если вы выполнили команду `config:cache` во время процесса развертывания, вы должны быть уверены, что вызываете функцию `env` только из файлов конфигурации. Как только конфигурации будут кешированы, файл `.env` не будет загружаться, и все вызовы функции `env` будут возвращать `null`.

<a name="method-event"></a>
#### `event()`

Функция `event` отправляет переданное [событие](events.md) своим слушателям:

    event(new UserRegistered($user));

<a name="method-filled"></a>
#### `filled()`

Функция `filled` проверяет, является ли переданное значение не «пустым»:

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

Обратной функции `filled` является функция [`blank`](#method-blank).

<a name="method-info"></a>
#### `info()`

Функция `info` запишет информацию в [журнал](logging.md):

    info('Some helpful information!');

Также функции может быть передан массив контекстных данных:

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()`

Функцию `logger` можно использовать для записи сообщения уровня `debug` в [журнал](logging.md):

    logger('Debug message');

Также функции может быть передан массив контекстных данных:

    logger('User has logged in.', ['id' => $user->id]);

Если функции не передано значение, то будет возвращен экземпляр [регистратора](errors.md#logging):

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()`

Функция `method_field` генерирует HTML «скрытого» поле ввода, содержащее поддельное значение HTTP-метода формы. Например, используя [синтаксис Blade](blade.md):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()`

Функция `now` создает новый экземпляр `Illuminate\Support\Carbon` для текущего времени:

    $now = now();

<a name="method-old"></a>
#### `old()`

Функция `old` [возвращает](requests.md#retrieving-input) значение [прежнего ввода](requests.md#old-input), краткосрочно сохраненное в сессии:

    $value = old('value');

    $value = old('value', 'default');

Поскольку «значение по умолчанию», предоставляемое в качестве второго аргумента функции `old`, часто является атрибутом модели Eloquent, то Laravel позволяет вам просто передать всю модель Eloquent в качестве второго аргумента функции `old`. При этом Laravel будет предполагать, что первый аргумент, предоставленный функции `old`, является именем атрибута Eloquent, который следует считать «значением по умолчанию»:

    {{ old('name', $user->name) }}

    // Эквивалентно ...

    {{ old('name', $user) }}

<a name="method-optional"></a>
#### `optional()`

Функция `optional` принимает любой аргумент и позволяет вам получать доступ к свойствам или вызывать методы этого объекта. Если переданный объект имеет значение `null`, свойства и методы будут возвращать также `null` вместо вызова ошибки:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

Функция `optional` также принимает замыкание в качестве второго аргумента. Замыкание будет вызвано, если значение, указанное в качестве первого аргумента, не равно `null`:

    return optional(User::find($id), function ($user) {
        return $user->name;
    });

<a name="method-policy"></a>
#### `policy()`

Функция `policy` извлекает экземпляр [политики](authorization.md#creating-policies) для переданного класса:

    $policy = policy(App\Models\User::class);

<a name="method-redirect"></a>
#### `redirect()`

Функция `redirect` возвращает [HTTP-ответ перенаправления](responses.md#redirects) или возвращает экземпляр перенаправителя, если вызывается без аргументов:

    return redirect($to = null, $status = 302, $headers = [], $https = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()`

Функция `report` сообщит об исключении, используя ваш [обработчик исключений](errors.md#the-exception-handler):

    report($e);

Функция `report` также принимает строку в качестве аргумента. Когда в функцию передается строка, она создает исключение с переданной строкой в качестве сообщения:

    report('Something went wrong.');

<a name="method-request"></a>
#### `request()`

Функция `request` возвращает экземпляр текущего [запроса](requests.md) или получает значение поля ввода из текущего запроса:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()`

Функция `rescue` выполняет переданное замыкание и перехватывает любые исключения, возникающие во время его выполнения. Все перехваченные исключения будут отправлены вашему [обработчику исключений](errors.md#the-exception-handler); однако, обработка запроса будет продолжена:

    return rescue(function () {
        return $this->method();
    });

Вы также можете передать второй аргумент функции `rescue`. Этот аргумент будет значением «по умолчанию», которое должно быть возвращено, если во время выполнения замыкание возникнет исключение:

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-resolve"></a>
#### `resolve()`

Функция `resolve` извлекает экземпляр связанного с переданным классом или интерфейсом, используя [контейнер служб](container.md):

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()`

Функция `response` создает экземпляр [ответа](responses.md) или получает экземпляр фабрики ответов:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()`

Функция `retry` пытается выполнить переданное замыкание, пока не будет достигнут указанный лимит попыток. Если замыкание не выбросит исключение, то будет возвращено его значение. Если замыкание выбросит исключение, то замыкание будет автоматически повторено. Если максимальное количество попыток превышено, будет выброшено исключение:

    return retry(5, function () {
        // Попытаться выполнить 5 раз с перерывом 100 мс между попытками ...
    }, 100);

Если вы хотите указать количество миллисекунд между попытками, то вы можете передать замыкание в качестве третьего аргумента функции `retry`:

    return retry(5, function () {
        // ...
    }, function ($attempt, $exception) {
        return $attempt * 100;
    });

Для удобства вы можете предоставить массив в качестве первого аргумента функции `retry`. Этот массив будет использоваться для определения количества миллисекунд ожидания между последующими попытками:

    return retry([100, 200] function () {
        // Ждем 100 мс при первой попытке, 200 мс при второй попытке ...
    });

Для задания определенных условий выполнения попытки, вы можете передать замыкание в качестве четвертого аргумента функции `retry`:

    return retry(5, function () {
        // ...
    }, 100, function ($exception) {
        return $exception instanceof RetryException;
    });

<a name="method-session"></a>
#### `session()`

Функция `session` используется для получения или задания значений [сессии](session.md):

    $value = session('key');

Вы можете установить значения, передав массив пар ключ / значение в функцию:

    session(['chairs' => 7, 'instruments' => 3]);

Если в функцию не передано значение, то будет возвращен экземпляр хранилища сессий:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()`

Функция `tap` принимает два аргумента: произвольное значение и замыкание. Значение будет передано в замыкание, а затем возвращено функцией `tap`. Возвращаемое значение замыкания не имеет значения:

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

Если замыкание не передано функции `tap`, то вы можете вызвать любой метод с указанным значением. Возвращаемое значение вызываемого метода всегда будет изначально указанное, независимо от того, что метод фактически возвращает в своем определении. Например, метод Eloquent `update` обычно возвращает целочисленное значение. Однако, мы можем заставить метод возвращать саму модель, увязав вызов метода `update` с помощью функции `tap`:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

Чтобы добавить к своему классу метод `tap`, используйте трейт `Illuminate\Support\Traits\Tappable` в вашем классе. Метод `tap` этого трейта принимает замыкание в качестве единственного аргумента. Сам экземпляр объекта будет передан замыканию, а затем будет возвращен методом `tap`:

    return $user->tap(function ($user) {
        //
    });

<a name="method-throw-if"></a>
#### `throw_if()`

Функция `throw_if` выбрасывает переданное исключение, если указанное логическое выражение оценивается как `true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page.'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()`

Функция `throw_unless` выбрасывает переданное исключение, если указанное логическое выражение оценивается как `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page.'
    );

<a name="method-today"></a>
#### `today()`

Функция `today` создает новый экземпляр `Illuminate\Support\Carbon` для текущей даты:

    $today = today();

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()`

Функция `trait_uses_recursive` возвращает все трейты, используемые трейтом:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()`

Функция `transform` выполняет замыкание для переданного значения, если значение не [пустое](#method-blank), и возвращает результат замыкания:

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

В качестве третьего параметра могут быть указанны значение по умолчанию или замыкание. Это значение будет возвращено, если переданное значение пустое:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()`

Функция `validator` создает новый экземпляр [валидатора](validation.md) с указанными аргументами. Вы можете использовать его для удобства вместо фасада `Validator`:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()`

Функция `value` возвращает переданное значение. Однако, если вы передадите замыкание в функцию, то замыкание будет выполнено, и будет возвращен его результат:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

<a name="method-view"></a>
#### `view()`

Функция `view` возвращает экземпляр [представления](views.md):

    return view('auth.login');

<a name="method-with"></a>
#### `with()`

Функция `with` возвращает переданное значение. Если вы передадите замыкание в функцию в качестве второго аргумента, то замыкание будет выполнено и будет возвращен результат его выполнения:

    $callback = function ($value) {
        return is_numeric($value) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
