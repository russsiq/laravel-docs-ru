# Laravel 9 · Тестирование · База данных

- [Введение](#introduction)
    - [Сброс базы данных после каждого теста](#resetting-the-database-after-each-test)
- [Определение фабрик моделей](#defining-model-factories)
    - [Обзор концепции](#concept-overview)
    - [Генерация фабрик](#generating-factories)
    - [Состояния фабрик](#factory-states)
    - [Хуки фабрик](#factory-callbacks)
- [Создание моделей с использованием фабрик](#creating-models-using-factories)
    - [Инициализация экземпляров моделей](#instantiating-models)
    - [Сохранение моделей](#persisting-models)
    - [Последовательность состояний](#sequences)
- [Отношения](#factory-relationships)
    - [Отношения Has Many](#has-many-relationships)
    - [Отношения Belongs To](#belongs-to-relationships)
    - [Отношения Many To Many](#many-to-many-relationships)
    - [Полиморфные отношения](#polymorphic-relationships)
    - [Определение отношений внутри фабрик](#defining-relationships-within-factories)
- [Запуск наполнителей](#running-seeders)
- [Доступные утверждения](#available-assertions)

<a name="introduction"></a>
## Введение

Laravel предлагает множество полезных инструментов и утверждений, чтобы упростить тестирование приложений, управляемых базой данных. Кроме того, фабрики моделей и наполнители Laravel позволяют безболезненно создавать записи тестовой базы данных с использованием моделей и отношений Eloquent вашего приложения. Мы обсудим все эти мощные функции в текущей документации.

<a name="resetting-the-database-after-each-test"></a>
### Сброс базы данных после каждого теста

Прежде чем продолжить, давайте обсудим, как сбрасывать вашу базу данных после каждого из ваших тестов, чтобы данные из предыдущего теста не мешали последующим тестам. Включенный в Laravel трейт `Illuminate\Foundation\Testing\RefreshDatabase` позаботится об этом за вас. Просто используйте трейт в своем тестовом классе:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * Отвлеченный пример функционального теста.
         *
         * @return void
         */
        public function test_basic_example()
        {
            $response = $this->get('/');

            // ...
        }
    }

Трейт `Illuminate\Foundation\Testing\RefreshDatabase` не мигрирует вашу базу данных, если ваша схема обновлена. Вместо этого он будет выполнять тест только в транзакции базы данных. Следовательно, любые записи, добавленные в базу данных тестом, не использующими этот трейт, могут по-прежнему существовать в базе данных.

Если вы хотите полностью сбросить базу данных с помощью миграции, то вы можете вместо этого использовать трейт `Illuminate\Foundation\Testing\DatabaseMigrations`. Однако использование трейта `DatabaseMigrations` значительно медленнее, чем использование трейта `RefreshDatabase`.

<a name="defining-model-factories"></a>
## Определение фабрик моделей

<a name="concept-overview"></a>
### Обзор концепции

Сначала поговорим о фабриках моделей Eloquent. При тестировании вам может потребоваться вставить несколько записей в вашу базу данных перед выполнением теста. Вместо того, чтобы вручную указывать значение каждого столбца при создании этих тестовых данных, Laravel позволяет вам определять набор атрибутов по умолчанию для каждой из ваших [моделей Eloquent](eloquent.md), используя фабрики моделей.

Чтобы увидеть пример написания фабрики, взгляните на файл `database/factories/UserFactory.php` в вашем приложении. Эта фабрика включена во все новые приложения Laravel и содержит следующее определение фабрики:

    namespace Database\Factories;

    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * Определить состояние модели по умолчанию.
         *
         * @return array
         */
        public function definition()
        {
            return [
                'name' => $this->faker->name(),
                'email' => $this->faker->unique()->safeEmail(),
                'email_verified_at' => now(),
                'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
                'remember_token' => Str::random(10),
            ];
        }
    }

Как видите, фабрики – это классы, которые расширяют базовый класс фабрики Laravel и определяют метод `definition`. Метод `definition` возвращает набор значений атрибутов по умолчанию, которые должны применяться при создании модели с использованием фабрики.

Через свойство `$faker` фабрики получают доступ к библиотеке [Faker](https://github.com/FakerPHP/Faker) PHP, которая позволяет вам удобно генерировать различные виды случайных данных для тестирования.

> {tip} Вы можете установить языковой стандарт Faker для своего приложения, добавив опцию `faker_locale` в конфигурационном файле `config/app.php`.

<a name="generating-factories"></a>
### Генерация фабрик

Чтобы сгенерировать новую фабрику, используйте команду `make:factory` [Artisan](artisan.md):

```shell
php artisan make:factory PostFactory
```

Эта команда поместит новый класс фабрики в каталог `database/factories` вашего приложения.

<a name="factory-and-model-discovery-conventions"></a>
#### Соглашения обнаружения фабрики и модели

После того определения фабрик, вы можете использовать статический метод `factory` трейта `Illuminate\Database\Eloquent\Factories\HasFactory` для создания экземпляра фабрики текущей модели.

Метод `factory` трейта `HasFactory` будет использовать соглашения для определения подходящей фабрики для модели, которой назначен трейт. В частности, метод будет искать фабрику в пространстве имен `Database\Factories`, имя класса которой соответствует имени модели и имеет суффикс `Factory`. Если эти соглашения не применимы к вашему конкретному приложению или фабрике, то вы можете переопределить метод `newFactory` вашей модели, чтобы напрямую вернуть экземпляр соответствующей фабрики модели:

    use Database\Factories\Administration\FlightFactory;

    /**
     * Создать новый экземпляр фабрики для модели.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    protected static function newFactory()
    {
        return FlightFactory::new();
    }

Затем определите свойство `model` соответствующей фабрики:

    use App\Administration\Flight;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class FlightFactory extends Factory
    {
        /**
         * Имя модели соответствующей фабрики.
         *
         * @var string
         */
        protected $model = Flight::class;
    }

<a name="factory-states"></a>
### Состояния фабрик

Методы управления состоянием позволяют вам определять дискретные изменения, которые могут быть применены к вашим фабрикам моделей в любой их комбинации. Например, ваша фабрика `Database\Factories\UserFactory` может содержать метод состояния `suspended`, который изменяет одно из значений атрибута по умолчанию.

Методы преобразования состояния обычно вызывают метод `state` базового класса фабрики Laravel. Метод `state` принимает замыкание, которое получит массив изначально определенных для фабрики атрибутов, и должен вернуть массив изменяемых атрибутов:

    /**
     * Указать, что аккаунт пользователя временно приостановлен.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    public function suspended()
    {
        return $this->state(function (array $attributes) {
            return [
                'account_status' => 'suspended',
            ];
        });
    }

#### Состояние `trashed` (временно удаленных)

Если ваша модель Eloquent поддерживает [программное удаление](eloquent.md#soft-deleting), то вы можете вызвать метод состояния `trashed` для указания, что созданная модель уже должна быть «программно удаленной». Вам не нужно самостоятельно определять состояние `trashed`, так как оно автоматически доступно для всех фабрик:

    use App\Models\User;

    $user = User::factory()->trashed()->create();

<a name="factory-callbacks"></a>
### Хуки фабрик

Хуки фабрик регистрируются с использованием методов `afterMaking` и `afterCreating` и позволяют выполнять дополнительные задачи после инициализации или создания модели. Вы должны зарегистрировать эти хуки, переопределив метод `configure` в вашем классе фабрики. Этот метод будет автоматически вызываться Laravel при создании экземпляра фабрики:

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * Конфигурация фабрики модели.
         *
         * @return $this
         */
        public function configure()
        {
            return $this->afterMaking(function (User $user) {
                //
            })->afterCreating(function (User $user) {
                //
            });
        }

        // ...
    }

<a name="creating-models-using-factories"></a>
## Создание моделей с использованием фабрик

<a name="instantiating-models"></a>
### Инициализация экземпляров моделей

После того, как вы определили свои фабрики, вы можете использовать статический метод `factory`, предоставляемый вашим моделям с помощью трейта `Illuminate\Database\Eloquent\Factories\HasFactory`, чтобы инициализировать экземпляр фабрики для этой модели. Давайте посмотрим на несколько примеров создания моделей. Во-первых, мы воспользуемся методом `make` для создания моделей, но без их сохранения в базе данных:

    use App\Models\User;

    public function test_models_can_be_instantiated()
    {
        $user = User::factory()->make();

        // Использование модели в тестах ...
    }

Вы можете создать коллекцию из множества моделей, используя метод `count`:

    $users = User::factory()->count(3)->make();

<a name="applying-states"></a>
#### Применение состояний

Вы также можете применить к моделям любое из ваших [состояний](#factory-states). Если вы хотите применить к моделям несколько изменений состояния, то вы можете просто вызвать методы преобразования состояния напрямую:

    $users = User::factory()->count(5)->suspended()->make();

<a name="overriding-attributes"></a>
#### Переопределение атрибутов

Если вы хотите переопределить некоторые значения по умолчанию для ваших моделей, вы можете передать массив значений методу `make`. Будут заменены только указанные атрибуты, в то время как для остальных атрибутов сохранятся значения по умолчанию, указанные в фабрике:

    $user = User::factory()->make([
        'name' => 'Abigail Otwell',
    ]);

В качестве альтернативы, метод `state` может быть вызван непосредственно на экземпляре фабрики для выполнения быстрого преобразования состояния:

    $user = User::factory()->state([
        'name' => 'Abigail Otwell',
    ])->make();

> {tip} [Защита от массового присвоения](eloquent.md#mass-assignment) автоматически отключается при создании моделей с использованием фабрик.

<a name="persisting-models"></a>
### Сохранение моделей

Метод `create` инициализирует экземпляры модели и сохраняет их в базе данных с помощью метода `save` модели Eloquent:

    use App\Models\User;

    public function test_models_can_be_persisted()
    {
        // Создаем один экземпляр `App\Models\User` ...
        $user = User::factory()->create();

        // Создаем три экземпляра `App\Models\User` ...
        $users = User::factory()->count(3)->create();

        // Использование модели в тестах ...
    }

Вы можете переопределить атрибуты модели по умолчанию, передав массив атрибутов методу `create`:

    $user = User::factory()->create([
        'name' => 'Abigail',
    ]);

<a name="sequences"></a>
### Последовательность состояний

По желанию можно изменить значение конкретного атрибута модели для каждой вновь созданной модели. Вы можете добиться этого, определив преобразование состояния как последовательность. Например, вы можете изменять значение столбца `admin` между `Y` и `N` для каждого вновь созданного пользователя:

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Sequence;

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        ['admin' => 'Y'],
                        ['admin' => 'N'],
                    ))
                    ->create();

В этом примере пять пользователей будут созданы со значением `admin`, равным `Y`, и пять пользователей – со значением `admin`, равным `N`.

При необходимости вы можете внедрить замыкание в качестве значения последовательности. Замыкание будет вызываться каждый раз, когда последовательности потребуется новое значение:

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        fn ($sequence) => ['role' => UserRoles::all()->random()],
                    ))
                    ->create();

Внутри замыкания последовательности вы можете получить доступ к свойствам `$index` или `$count` экземпляра последовательности, переданной в замыкание. Свойство `$index` содержит количество выполненных итераций последовательности, а свойство `$count` содержит общее количество вызываемых итераций раз последовательности:

    $users = User::factory()
                    ->count(10)
                    ->sequence(fn ($sequence) => ['name' => 'Name '.$sequence->index])
                    ->create();

<a name="factory-relationships"></a>
## Отношения

<a name="has-many-relationships"></a>
### Отношения Has Many

Затем, давайте рассмотрим построение отношений модели Eloquent с использованием текучего интерфейса методов фабрик Laravel. Во-первых, предположим, что у нашего приложения есть модель `App\Models\User` и модель `App\Models\Post`. Также предположим, что модель `User` определяет отношения `hasMany` с `Post`. Мы можем создать пользователя с тремя постами, используя метод `has`, предоставляемый фабриками Laravel. Метод `has` принимает экземпляр фабрики:

    use App\Models\Post;
    use App\Models\User;

    $user = User::factory()
                ->has(Post::factory()->count(3))
                ->create();

По соглашению, при передаче модели `Post` методу `has`, Laravel будет предполагать, что модель `User` должна иметь метод `posts`, который определяет отношения. При необходимости вы можете явно указать имя отношения, которым вы хотите управлять:

    $user = User::factory()
                ->has(Post::factory()->count(3), 'posts')
                ->create();

Конечно, вы можете выполнять манипуляции с состоянием связанных моделей. Кроме того, вы можете передать преобразование состояния на основе замыкания, если для изменения вашего состояния требуется доступ к родительской модели:

    $user = User::factory()
                ->has(
                    Post::factory()
                            ->count(3)
                            ->state(function (array $attributes, User $user) {
                                return ['user_type' => $user->type];
                            })
                )
                ->create();

<a name="has-many-relationships-using-magic-methods"></a>
#### Использование магических методов Has Many

Для удобства вы можете использовать магические методы отношений фабрики Laravel для построения отношений. Например, в следующем примере будет использоваться соглашение, чтобы определить, что связанные модели должны быть созданы с помощью метода отношений `posts` модели `User`:

    $user = User::factory()
                ->hasPosts(3)
                ->create();

При использовании магических методов для создания отношений фабрики вы можете передать массив атрибутов для их переопределения в связанных моделях:

    $user = User::factory()
                ->hasPosts(3, [
                    'published' => false,
                ])
                ->create();

Вы можете предоставить преобразование состояния на основе замыкания, если для изменения состояния требуется доступ к родительской модели:

    $user = User::factory()
                ->hasPosts(3, function (array $attributes, User $user) {
                    return ['user_type' => $user->type];
                })
                ->create();

<a name="belongs-to-relationships"></a>
### Отношения Belongs To

Теперь, когда мы изучили, как построить отношения Has Many с помощью фабрик, давайте рассмотрим обратное отношение. Метод `for` используется для определения родительской модели, к которой принадлежат модели, созданные фабрикой. Например, мы можем создать три экземпляра модели `App\Models\Post`, которые принадлежат одному пользователю:

    use App\Models\Post;
    use App\Models\User;

    $posts = Post::factory()
                ->count(3)
                ->for(User::factory()->state([
                    'name' => 'Jessica Archer',
                ]))
                ->create();

Если у вас уже есть экземпляр родительской модели, который должен быть связан с моделями, которые вы создаете, вы можете передать экземпляр модели методу `for`:

    $user = User::factory()->create();

    $posts = Post::factory()
                ->count(3)
                ->for($user)
                ->create();

<a name="belongs-to-relationships-using-magic-methods"></a>
#### Использование магических методов Belongs To

Для удобства вы можете использовать магические методы отношений фабрики Laravel для построения отношений Belongs To. Например, в следующем примере будет использоваться соглашение, чтобы определить, что три поста должны принадлежать отношениям `user` в модели `Post`:

    $posts = Post::factory()
                ->count(3)
                ->forUser([
                    'name' => 'Jessica Archer',
                ])
                ->create();

<a name="many-to-many-relationships"></a>
### Отношения Many To Many

Как и [отношения Has Many](#has-many-relationships), отношения Many To Many могут быть созданы с использованием метода `has`:

    use App\Models\Role;
    use App\Models\User;

    $user = User::factory()
                ->has(Role::factory()->count(3))
                ->create();

<a name="pivot-table-attributes"></a>
#### Атрибуты сводной таблицы

Если вам нужно определить атрибуты, которые должны быть установлены в сводной / промежуточной таблице, связывающей модели, вы можете использовать метод `hasAttached`. Этот метод принимает в качестве второго аргумента массив имен и значений атрибутов сводной таблицы:

    use App\Models\Role;
    use App\Models\User;

    $user = User::factory()
                ->hasAttached(
                    Role::factory()->count(3),
                    ['active' => true]
                )
                ->create();

Вы можете предоставить преобразование состояния на основе замыкания, если для изменения состояния требуется доступ к связанной модели:

    $user = User::factory()
                ->hasAttached(
                    Role::factory()
                        ->count(3)
                        ->state(function (array $attributes, User $user) {
                            return ['name' => $user->name.' Role'];
                        }),
                    ['active' => true]
                )
                ->create();

Если у вас уже есть экземпляры модели, которые вы хотите прикрепить к создаваемым моделям, вы можете передать экземпляры модели методу `hasAttached`. В этом примере всем трем пользователям будут назначены одни и те же три роли:

    $roles = Role::factory()->count(3)->create();

    $user = User::factory()
                ->count(3)
                ->hasAttached($roles, ['active' => true])
                ->create();

<a name="many-to-many-relationships-using-magic-methods"></a>
#### Использование магических методов Many To Many

Для удобства вы можете использовать магические методы отношений фабрики Laravel для построения отношений Many To Many. Например, в следующем примере будет использоваться соглашение, чтобы определить, что связанные модели должны быть созданы с помощью метода отношений `roles` модели `User`:

    $user = User::factory()
                ->hasRoles(1, [
                    'name' => 'Editor'
                ])
                ->create();

<a name="polymorphic-relationships"></a>
### Полиморфные отношения

[Полиморфные отношения](eloquent-relationships.md#polymorphic-relationships) также могут быть созданы с использованием фабрик. Полиморфные отношения Morph Many создаются так же, как типичные отношения Has Many. Например, если модель `App\Models\Post` имеет отношение `morphMany` с моделью `App\Models\Comment`:

    use App\Models\Post;

    $post = Post::factory()->hasComments(3)->create();

<a name="morph-to-relationships"></a>
#### Отношения Morph To

Магические методы нельзя использовать для создания отношений `morphTo`. Вместо этого метод `for` должен использоваться напрямую, а имя отношения должно быть явно указано. Например, представьте, что модель `Comment` имеет метод `commentable`, который определяет отношение `morphTo`. В этой ситуации мы можем создать три комментария, относящиеся к одному посту, используя напрямую метод `for`:

    $comments = Comment::factory()->count(3)->for(
        Post::factory(), 'commentable'
    )->create();

<a name="polymorphic-many-to-many-relationships"></a>
#### Полиморфные отношения Many To Many

Полиморфные отношения Many To Many (`morphToMany` / `morphedByMany`) могут быть созданы точно так же, как неполиморфные отношения Many To Many:

    use App\Models\Tag;
    use App\Models\Video;

    $videos = Video::factory()
                ->hasAttached(
                    Tag::factory()->count(3),
                    ['public' => true]
                )
                ->create();

Конечно, магический метод `has` также используется для создания полиморфных отношений Many To Many:

    $videos = Video::factory()
                ->hasTags(3, ['public' => true])
                ->create();

<a name="defining-relationships-within-factories"></a>
### Определение отношений внутри фабрик

Чтобы определить отношение в рамках вашей фабрики модели, вы обычно назначаете новый экземпляр фабрики внешнему ключу отношения. Обычно это делается для «обратных» отношений, таких как `belongsTo` и `morphTo`. Например, если вы хотите создать нового пользователя при создании публикации, вы можете сделать следующее:

    use App\Models\User;

    /**
     * Определить состояние модели по умолчанию.
     *
     * @return array
     */
    public function definition()
    {
        return [
            'user_id' => User::factory(),
            'title' => $this->faker->title(),
            'content' => $this->faker->paragraph(),
        ];
    }

Если столбцы отношения зависят от фабрики, которая его определяет, вы можете назначить замыкание атрибуту. Замыкание получит массив проанализированных атрибутов фабрики:

    /**
     * Определить состояние модели по умолчанию.
     *
     * @return array
     */
    public function definition()
    {
        return [
            'user_id' => User::factory(),
            'user_type' => function (array $attributes) {
                return User::find($attributes['user_id'])->type;
            },
            'title' => $this->faker->title(),
            'content' => $this->faker->paragraph(),
        ];
    }

<a name="running-seeders"></a>
## Запуск наполнителей

Если вы хотите использовать [наполнители базы данных](seeding.md) для наполнения вашей базы данных во время функционального тестирования, то вы можете вызвать метод `seed`. По умолчанию метод `seed` будет запускать `DatabaseSeeder`, который должен запускать все другие ваши наполнители. Как вариант, вы можете передать конкретное имя класса-наполнителя методу `seed`:

    <?php

    namespace Tests\Feature;

    use Database\Seeders\OrderStatusSeeder;
    use Database\Seeders\TransactionStatusSeeder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * Тест создания нового заказа.
         *
         * @return void
         */
        public function test_orders_can_be_created()
        {
            // Запустить `DatabaseSeeder` ...
            $this->seed();

            // Запустить конкретный наполнитель ...
            $this->seed(OrderStatusSeeder::class);

            // ...

            // Запустить массив определенных наполнителей ...
            $this->seed([
                OrderStatusSeeder::class,
                TransactionStatusSeeder::class,
                // ...
            ]);
        }
    }

В качестве альтернативы вы можете указать Laravel автоматически заполнять базу данных перед каждым тестом, использующим трейт `RefreshDatabase`. Вы можете добиться этого, определив свойство `$seed` в вашем базовом тестовом классе:

    <?php

    namespace Tests;

    use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

    abstract class TestCase extends BaseTestCase
    {
        use CreatesApplication;

        /**
         * Указывает, следует ли запускать наполнитель по умолчанию перед каждым тестом.
         *
         * @var bool
         */
        protected $seed = true;
    }

Когда свойство `$seed` имеет значение `true`, тогда класс `Database\Seeders\DatabaseSeeder` будет запускаться перед каждым тестом, использующим трейт `RefreshDatabase`. Однако, вы можете указать конкретный наполнитель, который должен выполняться, определив свойство `$seeder` в вашем тестовом классе:

    use Database\Seeders\OrderStatusSeeder;

    /**
     * Запускать указанный наполнитель перед каждым тестом.
     *
     * @var string
     */
    protected $seeder = OrderStatusSeeder::class;

<a name="available-assertions"></a>
## Доступные утверждения

Laravel содержит несколько утверждений базы данных для ваших функциональных тестов [PHPUnit](https://phpunit.de/). Мы обсудим каждое из этих утверждений ниже.

<a name="assert-database-count"></a>
#### assertDatabaseCount

Утверждение о том, что таблица в базе данных содержит указанное количество записей:

    $this->assertDatabaseCount('users', 5);

<a name="assert-database-has"></a>
#### assertDatabaseHas

Утверждение о том, что таблица в базе данных содержит записи, соответствующие переданным ключ / значение ограничениям запроса:

    $this->assertDatabaseHas('users', [
        'email' => 'sally@example.com',
    ]);

<a name="assert-database-missing"></a>
#### assertDatabaseMissing

Утверждение о том, что таблица в базе данных не содержит записей, соответствующих переданным ключ / значение ограничениям запроса:

    $this->assertDatabaseMissing('users', [
        'email' => 'sally@example.com',
    ]);

<a name="assert-soft-deleted"></a>
#### assertSoftDeleted

Метод `assertSoftDeleted` используется для утверждения того, что переданная модель Eloquent была «программно удалена»:

    $this->assertSoftDeleted($user);

<a name="assert-model-exists"></a>
#### assertModelExists

Утверждение о том, что переданная модель существует в базе данных:

    use App\Models\User;

    $user = User::factory()->create();

    $this->assertModelExists($user);

<a name="assert-model-missing"></a>
#### assertModelMissing

Утверждение о том, что переданная модель не существует в базе данных:

    use App\Models\User;

    $user = User::factory()->create();

    $user->delete();

    $this->assertModelMissing($user);
