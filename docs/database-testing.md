# Laravel 9 · Тестирование · База данных

- [Введение](#introduction)
    - [Сброс базы данных после каждого теста](#resetting-the-database-after-each-test)
- [Фабрики моделей](#model-factories)
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

<a name="model-factories"></a>
## Фабрики моделей

При тестировании вам может потребоваться вставить несколько записей в вашу базу данных перед выполнением теста. Вместо того, чтобы вручную указывать значение каждого столбца при создании этих тестовых данных, Laravel позволяет вам определять набор атрибутов по умолчанию для каждой из ваших [моделей Eloquent](eloquent.md), используя [фабрики моделей](eloquent-factories.md).

Чтобы узнать больше о создании и использовании фабрик моделей, обратитесь к полной [документации фабрики моделей](eloquent-factories.md). После того, как вы определили фабрику модели, вы можете использовать фабрику в своем тесте для создания моделей:

    use App\Models\User;

    public function test_models_can_be_instantiated()
    {
        $user = User::factory()->create();

        // ...
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

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

Метод `assertSoftDeleted` используется для утверждения того, что переданная модель Eloquent не была «программно удалена»:

    $this->assertNotSoftDeleted($user);

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
