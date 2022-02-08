# Laravel 9 · База данных · Наполнение фиктивными данными

- [Введение](#introduction)
- [Написание наполнителей](#writing-seeders)
    - [Использование фабрик моделей](#using-model-factories)
    - [Вызов дополнительных наполнителей](#calling-additional-seeders)
    - [Подавление событий моделей](#muting-model-events)
- [Запуск наполнителей](#running-seeders)

<a name="introduction"></a>
## Введение

Laravel предлагает возможность наполнения базы данных с использованием классов-наполнителей. Все классы наполнителей хранятся в каталоге `database/seeders`. Класс `DatabaseSeeder` уже определен по умолчанию. В этом классе вы можете использовать метод `call` для запуска других наполнителей, что позволит вам контролировать порядок наполнения БД.

> {tip} При наполнении базы данных автоматически отключается защита [массового присвоения](eloquent.md#mass-assignment).

<a name="writing-seeders"></a>
## Написание наполнителей

Чтобы сгенерировать новый наполнитель, используйте команду `make:seeder` [Artisan](artisan.md). Эта команда поместит новый класс наполнителя в каталог `database/seeders` вашего приложения:

```shell
php artisan make:seeder UserSeeder
```

Класс наполнителя по умолчанию содержит только один метод: `run`. Этот метод вызывается при выполнении команды `db:seed` Artisan. В методе `run` вы можете вставлять данные в свою базу данных, как хотите. Вы можете использовать [построитель запросов](queries.md) для самостоятельной вставки данных или использовать [фабрики моделей Eloquent](database-testing.md#defining-model-factories).

В качестве примера давайте изменим класс `DatabaseSeeder`, созданный по умолчанию, и добавим выражение вставки фасада `DB` в методе `run`:

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Запустить наполнение базы данных.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => Str::random(10),
                'email' => Str::random(10).'@gmail.com',
                'password' => Hash::make('password'),
            ]);
        }
    }

> {tip} В методе `run` вы можете объявить любые необходимые типы зависимостей. Они будут автоматически извлечены и внедрены через [контейнер служб](container.md) Laravel.

<a name="using-model-factories"></a>
### Использование фабрик моделей

Конечно, ручное указание атрибутов для каждой модели наполнителя обременительно. Вместо этого вы можете использовать [фабрики моделей](database-testing.md#defining-model-factories) для удобного создания большого количества записей в БД. Сначала просмотрите [документацию фабрики моделей](database-testing.md#defining-model-factories), чтобы узнать, как определить свои фабрики.

Например, давайте создадим 50 пользователей, у каждого из которых будет по одному посту:

    use App\Models\User;

    /**
     * Запустить наполнение базы данных.
     *
     * @return void
     */
    public function run()
    {
        User::factory()
                ->count(50)
                ->hasPosts(1)
                ->create();
    }

<a name="calling-additional-seeders"></a>
### Вызов дополнительных наполнителей

Внутри класса `DatabaseSeeder` вы можете использовать метод `call` для запуска других наполнителей. Использование метода `call` позволяет вам разбить ваши наполнители БД на несколько файлов, так что ни один класс наполнителя не станет слишком большим. Метод `call` принимает массив классов, которые должны быть выполнены:

    /**
     * Запустить наполнение базы данных.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }

<a name="muting-model-events"></a>
### Подавление событий моделей

Во время наполнения вы можете запретить моделям отправлять события. Вы можете добиться этого, используя трейт `WithoutModelEvents`. При использовании трейта `WithoutModelEvents` гарантирует, что события модели не инициируются, даже если дополнительные классы наполнителей выполняются с помощью метода `call`:

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Console\Seeds\WithoutModelEvents;

    class DatabaseSeeder extends Seeder
    {
        use WithoutModelEvents;

        /**
         * Запустить наполнение базы данных.
         *
         * @return void
         */
        public function run()
        {
            $this->call([
                UserSeeder::class,
            ]);
        }
    }

<a name="running-seeders"></a>
## Запуск наполнителей

Вы можете выполнить команду `db:seed` Artisan для наполнения вашей базы данных. По умолчанию команда `db:seed` запускает класс `Database\Seeders\DatabaseSeeder`, который, в свою очередь, может вызывать другие классы. Однако вы можете использовать параметр `--class`, чтобы указать конкретный класс наполнителя для его индивидуального запуска:

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

Вы также можете заполнить свою базу данных с помощью команды `migrate:fresh` в сочетании с параметром `--seed`, которая удалит все таблицы и повторно запустит все ваши миграции. Эта команда полезна для полного перестроения вашей базы данных:

```shell
php artisan migrate:fresh --seed
```

<a name="forcing-seeding-production"></a>
#### Принудительное наполнение при эксплуатации приложения

Некоторые операции наполнения могут привести к изменению или потере данных. В окружении `production`, чтобы защитить вас от запуска команд наполнения эксплуатируемой базы данных, вам будет предложено подтвердить их запуск. Чтобы заставить наполнители запускаться без подтверждений, используйте флаг `--force`:

```shell
php artisan db:seed --force
```
