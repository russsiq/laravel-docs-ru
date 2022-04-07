# Laravel 9 · Eloquent · Ресурсы API

- [Введение](#introduction)
- [Генерация ресурсов](#generating-resources)
- [Обзор концепции](#concept-overview)
    - [Коллекции ресурса](#resource-collections)
- [Написание ресурсов](#writing-resources)
    - [Обертывание данных](#data-wrapping)
    - [Постраничная разбивка](#pagination)
    - [Условные атрибуты](#conditional-attributes)
    - [Условные отношения](#conditional-relationships)
    - [Добавление метаданных](#adding-meta-data)
- [Ответы ресурса](#resource-responses)

<a name="introduction"></a>
## Введение

При создании API вам может потребоваться слой преобразования, который находится между вашими моделями Eloquent и ответами JSON, которые фактически возвращаются пользователям вашего приложения. Например, бывает необходимо отображать определенные атрибуты только для некоторого сегмента пользователей, а не для всех, или бывает необходимо всегда отображать определенные отношения в JSON-представление ваших моделей. Классы ресурсов Eloquent позволяют легко и выразительно преобразовывать модели и коллекции моделей в JSON.

Конечно, вы всегда можете преобразовать модели или коллекции Eloquent в JSON, используя их методы `toJson`; однако ресурсы Eloquent обеспечивают более детальный и надежный контроль над сериализацией в JSON ваших моделей и их отношений.

<a name="generating-resources"></a>
## Генерация ресурсов

Ресурсы расширяют класс `Illuminate\Http\Resources\Json\JsonResource`. Чтобы сгенерировать новый ресурс, используйте команду `make:resource` [Artisan](artisan.md). Эта команда поместит новый класс ресурса в каталог `app/Http/Resources` вашего приложения:

```shell
php artisan make:resource UserResource
```

<a name="generating-resource-collections"></a>
#### Генерация коллекций ресурса

Помимо создания ресурсов, преобразующих отдельные модели, вы можете создавать ресурсы, отвечающие за преобразование коллекций моделей. Это позволяет вашим ответам JSON включать ссылки и другую метаинформацию, имеющую отношение ко всей коллекции конкретного ресурса.

Чтобы сгенерировать новую коллекцию ресурса, вы должны использовать флаг `--collection` при создании ресурса. Или включение слова `Collection` в имя ресурса укажет Laravel, что он должен создать коллекцию ресурса. Коллекции ресурса расширяют класс `Illuminate\Http\Resources\Json\ResourceCollection`:

```shell
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

<a name="concept-overview"></a>
## Обзор концепции

> {tip} Это лишь общий обзор ресурсов и коллекций ресурса. Мы настоятельно рекомендуем вам прочитать другие разделы этой документации, чтобы получить более глубокое понимание возможностей создания и настройки ресурса, предлагаемые вам.

Прежде чем углубляться во все варианты, доступные вам при написании ресурсов, давайте сначала рассмотрим, как ресурсы используются в Laravel. Класс ресурсов представляет собой единую модель, которую необходимо преобразовать в структуру JSON. Например, вот простой класс ресурса `UserResource`:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Преобразовать ресурс в массив.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Каждый класс ресурсов определяет метод `toArray`, который возвращает массив атрибутов, которые должны быть преобразованы в JSON, когда ресурс возвращается в качестве ответа из метода маршрута или контроллера.

Обратите внимание, что мы можем получить доступ к свойствам модели непосредственно из переменной `$this`. Это связано с тем, что класс ресурсов автоматически проксирует свойства и методы к базовой модели для удобства доступа. Как только ресурс определен, он может быть возвращен из маршрута или контроллера. Ресурс принимает основной экземпляр модели через свой конструктор:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function ($id) {
        return new UserResource(User::findOrFail($id));
    });

<a name="resource-collections"></a>
### Коллекции ресурса

Если вы возвращаете коллекцию ресурса или ответ с постраничной разбивкой, то вы должны использовать метод `collection` класса ресурса, при создании экземпляра ресурса в вашем маршруте или контроллере:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all());
    });

Обратите внимание, что это не позволит добавить пользовательские метаданные, которые могут потребоваться при возвращении с вашей коллекцией. Если вы хотите получить больший контроль над ответом коллекции ресурса, то вы можете создать выделенный ресурс для представления коллекции:

```shell
php artisan make:resource UserCollection
```

После создания класса коллекции ресурса, вы можете легко определить любые метаданные, которые должны быть включены в ответ:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Преобразовать коллекцию ресурса в массив.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

После определения вашей коллекции ресурса, ее можно вернуть из маршрута или контроллера:

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="preserving-collection-keys"></a>
#### Сохранение ключей коллекции

При возврате коллекции ресурсов из маршрута, Laravel сбрасывает ключи коллекции для расположения их в числовом порядке. Однако, вы можете добавить свойство `$preserveKeys` в свой класс ресурса, указывающее, должны ли сохраняться исходные ключи коллекции:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Указывает, следует ли сохранить ключи коллекции ресурса.
         *
         * @var bool
         */
        public $preserveKeys = true;
    }

Когда для свойства `$preserveKeys` установлено значение `true`, ключи коллекции будут сохранены, когда коллекция будет возвращена из маршрута или контроллера:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all()->keyBy->id);
    });

<a name="customizing-the-underlying-resource-class"></a>
#### Настройка базового класса ресурсов

Обычно свойство `$this->collection` коллекции ресурса автоматически заполняется результатом сопоставления каждого элемента коллекции с его единственным классом ресурсов. Предполагается, что единственным классом ресурса является имя класса коллекции без завершающей части `Collection`. Кроме того, в зависимости от личных предпочтений, класс ресурсов в единственном числе может иметь суффикс `Resource`, а может и не иметь его.

Например, `UserCollection` попытается сопоставить переданные экземпляры пользователя с ресурсом `UserResource`. Чтобы изменить это поведение, вы можете переопределить свойство `$collects` вашей коллекции ресурса:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Ресурс, используемый при формировании коллекции.
         *
         * @var string
         */
        public $collects = Member::class;
    }

<a name="writing-resources"></a>
## Написание ресурсов

> {tip} Если вы не читали [обзор концепции](#concept-overview), настоятельно рекомендуется сделать это, прежде чем приступить к работе с этой документацией.

По сути, ресурсы просты. Им нужно только преобразовать переданную модель в массив. Итак, каждый ресурс содержит метод `toArray`, который переводит атрибуты вашей модели в удобный для API массив, который может быть возвращен из маршрутов или контроллеров вашего приложения:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Преобразовать ресурс в массив.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Как только ресурс определен, он может быть возвращен непосредственно из маршрута или контроллера:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function ($id) {
        return new UserResource(User::findOrFail($id));
    });

<a name="relationships"></a>
#### Отношения

Если вы хотите включить связанные ресурсы в свой ответ, вы можете добавить их в массив, возвращаемый методом вашего ресурса `toArray`. В этом примере мы будем использовать метод `collection` ресурса `PostResource`, чтобы добавить посты пользователя из блога в ответ ресурса:

    use App\Http\Resources\PostResource;

    /**
     * Преобразовать ресурс в массив.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} Если вы хотите включить отношения только тогда, когда они уже загружены, ознакомьтесь с документацией по [условным отношениям](#conditional-relationships).

<a name="writing-resource-collections"></a>
#### Коллекции ресурса

В то время как ресурсы преобразуют одну модель в массив, коллекции ресурса преобразуют коллекцию моделей в массив. Однако, необязательно определять класс коллекции ресурса для каждой из ваших моделей, поскольку все ресурсы предоставляют метод `collection` для генерации «специальной» (ad hoc) коллекции ресурсов на лету:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all());
    });

Однако, если вам нужно настроить метаданные, возвращаемые с коллекцией, необходимо определить собственную коллекцию ресурса:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Преобразовать коллекцию ресурса в массив.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Как и отдельные ресурсы, коллекции ресурса могут быть возвращены непосредственно из маршрутов или контроллеров:

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### Обертывание данных

По умолчанию, ваш самый верхний ресурс будет заключен в ключ `data`, когда ответ ресурса преобразуется в JSON. Так, например, типичный ответ коллекции ресурса выглядит следующим образом:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ]
}
```

Если вы хотите использовать собственный ключ вместо `data`, вы можете определить свойство `$wrap` в классе ресурса:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Обертка «данных», которую следует применить.
         *
         * @var string
         */
        public static $wrap = 'user';
    }

Если вы хотите отключить обертывание самого верхнего ресурса, то вы должны вызвать метод `withoutWrapping` базового класса `Illuminate\Http\Resources\Json\JsonResource`. Как правило, вызов этого метода осуществляется в методе `boot` поставщика `App\Providers\AppServiceProvider` или другого [поставщика служб](providers.md), загружаемого при каждом запросе к вашему приложению:

    <?php

    namespace App\Providers;

    use Illuminate\Http\Resources\Json\JsonResource;
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
            JsonResource::withoutWrapping();
        }
    }

> {note} Метод `withoutWrapping` влияет только на самый верхний уровень ответа и не удаляет ключи `data`, которые вы вручную добавляете в свои собственные коллекции ресурса.

<a name="wrapping-nested-resources"></a>
#### Обертывание вложенных ресурсов

У вас есть полная свобода определять, как обернуты отношения между вашими ресурсами. Если вы хотите, чтобы все коллекции ресурсов были обернуты ключом `data`, независимо от их вложенности, то вы должны определить класс коллекции для каждого ресурса и вернуть коллекцию с ключом `data`.

Вам может быть интересно: не приведет ли это к тому, что верхний уровень вашего ресурса будет дважды обернут ключом `data`? Не волнуйтесь, Laravel не позволит вашим ресурсам быть случайно обернутыми двойной оберткой, поэтому вам не нужно беспокоиться об уровне вложенности трансформируемой коллекции ресурсов:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * Преобразовать коллекцию ресурса в массив.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

<a name="data-wrapping-and-pagination"></a>
#### Обертывание данных и постраничная разбивка

При возврате разбитых на страницы коллекций через ответ ресурса, Laravel обернет ваши данные ресурса в ключ `data`, даже если был вызван метод `withoutWrapping`. Это потому, что разбитые на страницы ответы всегда содержат ключи `meta` и `links` с информацией о состоянии постраничной разбивки:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="pagination"></a>
### Постраничная разбивка

Вы можете передать экземпляр пагинатора Laravel методу `collection` ресурса или вашей коллекции ресурса:

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

Ответы с постраничной разбивкой всегда содержат ключи `meta` и `links` с информацией о состоянии пагинатора:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="conditional-attributes"></a>
### Условные атрибуты

По желанию можно включить атрибут в ответ ресурса, только если какое-то условие выполнено. Например, бывает необходимо включить значение, только если текущий пользователь является «администратором». Laravel предлагает множество вспомогательных методов, которые помогут вам в этой ситуации. Метод `when` используется для условного добавления атрибута в ответ ресурса:

    /**
     * Преобразовать ресурс в массив.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

В этом примере ключ `secret` будет возвращен в конечном ответе ресурса только в том случае, если метод `isAdmin` аутентифицированного пользователя вернет `true`. Если метод возвращает `false`, то ключ `secret` будет удален из ответа ресурса перед его отправкой клиенту. Метод `when` позволяет вам выразительно определять ваши ресурсы, не прибегая к условным операторам при построении массива.

Метод `when` также принимает замыкание в качестве второго аргумента, позволяя вам вычислить результирующее значение, только если переданное условие истинно:

    'secret' => $this->when($request->user()->isAdmin(), function () {
        return 'secret-value';
    }),

Кроме того, метод `whenNotNull` может использоваться для включения атрибута в ответ ресурса, если атрибут не равен `null`:

    'name' => $this->whenNotNull($this->name),

<a name="merging-conditional-attributes"></a>
#### Слияние условных атрибутов

Иногда у вас может быть несколько атрибутов, которые следует включать в ответ ресурса только при одном и том же условии. В этом случае вы можете использовать метод `mergeWhen` для включения атрибутов в ответ только в том случае, если переданное условие истинно:

    /**
     * Преобразовать ресурс в массив.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($request->user()->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Опять же, если переданное условие равносильно `false`, то эти атрибуты будут удалены из ответа ресурса перед его отправкой клиенту.

> {note} Метод `mergeWhen` не следует использовать в массивах, в которых смешиваются строковые и числовые ключи. Кроме того, его не следует использовать в массивах с цифровыми ключами, которые не упорядочены последовательно.

<a name="conditional-relationships"></a>
### Условные отношения

В дополнение к условной загрузке атрибутов, вы можете условно включать отношения в свои ответы ресурса в зависимости от того, было ли отношение уже загружено в модель. Это позволяет вашему контроллеру решать, какие отношения должны быть загружены в модель, и ваш ресурс может легко включить их, только когда они действительно были загружены. В конечном итоге это позволяет избежать проблем «N+1» с запросами в ваших ресурсах.

Метод `whenLoaded` используется для условной загрузки отношения. Чтобы избежать ненужной загрузки отношений, этот метод принимает имя отношения вместо самого отношения:

    use App\Http\Resources\PostResource;

    /**
     * Преобразовать ресурс в массив.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

В этом примере, если отношение не было загружено, ключ `posts` будет удален из ответа ресурса перед его отправкой клиенту.

<a name="conditional-pivot-information"></a>
#### Условная сводная информация

В дополнение к условному включению информации об отношениях в ответах ваших ресурсов, вы можете условно включать данные из сводных таблиц отношений «многие ко многим» с помощью метода `whenPivotLoaded`. Метод `whenPivotLoaded` принимает имя сводной таблицы в качестве своего первого аргумента. Второй аргумент должен быть замыканием, возвращающем значение, если в модели доступна сводная информация:

    /**
     * Преобразовать ресурс в массив.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_user', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

Если ваши отношения используют [пользовательскую модель сводной таблицы](eloquent-relationships.md#defining-custom-intermediate-table-models), то вы можете передать экземпляр модели сводной таблицы в качестве первого аргумента методу `whenPivotLoaded`. :

    'expires_at' => $this->whenPivotLoaded(new Membership, function () {
        return $this->pivot->expires_at;
    }),

Если ваша сводная таблица использует аксессор, отличный от `pivot`, то вы можете использовать метод `whenPivotLoadedAs`:

    /**
     * Преобразовать ресурс в массив.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
                return $this->subscription->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>
### Добавление метаданных

Некоторые стандарты API JSON требуют добавления метаданных в ответы ваших ресурсов и коллекции ресурсов. Это часто включает такие вещи, как «ссылки» на ресурс или связанные ресурсы или метаданные о самом ресурсе. Если вам нужно вернуть дополнительные метаданные о ресурсе, включите их в свой метод `toArray`. Например, вы можете включить информацию `links` при преобразовании коллекции ресурса:

    /**
     * Преобразовать ресурс в массив.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

При возврате дополнительных метаданных из ваших ресурсов вам никогда не придется беспокоиться о случайном переопределении ключей `links` или `meta`, которые автоматически добавляются Laravel при возврате ответов с постраничной разбивкой. Любые дополнительные `links`, которые вы определяете, будут объединены с предоставленными пагинатором.

<a name="top-level-meta-data"></a>
#### Метаданные верхнего уровня

По желанию можно включить в ответ ресурса только определенные метаданные, если ресурс является самым верхним из возвращаемых ресурсов. Обычно это метаинформация об ответе в целом. Чтобы определить эти метаданные, добавьте метод `with` к вашему классу ресурсов. Этот метод должен возвращать массив метаданных, которые будут включены в ответ ресурса, только если ресурс является самым верхним ресурсом, который преобразуется:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Преобразовать коллекцию ресурса в массив.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * Получить дополнительные данные, возвращаемые с массивом ресурса.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

<a name="adding-meta-data-when-constructing-resources"></a>
#### Добавление метаданных при создании ресурсов

Вы также можете добавить данные верхнего уровня при создании экземпляров ресурсов в своем маршруте или контроллере. Метод `additional`, доступный для всех ресурсов, принимает массив данных, которые должны быть добавлены в ответ ресурса:

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## Ответы ресурса

Как вы уже читали, ресурсы могут быть возвращены напрямую из маршрутов и контроллеров:

    use App\Http\Resources\User as UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function ($id) {
        return new UserResource(User::findOrFail($id));
    });

Иногда требуется настроить исходящий HTTP-ответ перед его отправкой клиенту. Это можно сделать двумя способами. Во-первых, вы можете связать метод `response` с ресурсом. Этот метод вернет экземпляр `Illuminate\Http\JsonResponse`, что даст вам полный контроль над заголовками ответа:

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

В качестве альтернативы вы можете определить метод `withResponse` внутри самого ресурса. Этот метод будет вызываться, только когда ресурс будет возвращен как самый верхний ресурс в ответе:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Преобразовать ресурс в массив.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * Настроить исходящий ответ для ресурса.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Illuminate\Http\Response  $response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }
