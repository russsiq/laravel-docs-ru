# Laravel 9 · База данных · Использование Redis

- [Введение](#introduction)
- [Конфигурирование](#configuration)
    - [Кластеры](#clusters)
    - [Predis](#predis)
    - [phpredis](#phpredis)
- [Взаимодействие с Redis](#interacting-with-redis)
    - [Транзакции](#transactions)
    - [Конвейерное выполнение команд](#pipelining-commands)
- [Публикация / подписка](#pubsub)

<a name="introduction"></a>
## Введение

[Redis](https://redis.io) – это расширенное хранилище ключ-значение с открытым исходным кодом. Его часто называют сервером структуры данных, поскольку ключи могут содержать [строки](https://redis.io/topics/data-types#strings), [хеши](https://redis.io/topics/data-types#hashes), [списки](https://redis.io/topics/data-types#lists), [наборы](https://redis.io/topics/data-types#sets) и [отсортированные наборы](https://redis.io/topics/data-types#sorted-sets).

Перед использованием Redis с Laravel мы рекомендуем вам установить и использовать расширение [phpredis](https://github.com/phpredis/phpredis) PHP через PECL. Расширение сложнее установить по сравнению с пакетами PHP пользовательского слоя, но оно может обеспечить лучшую производительность для приложений, интенсивно использующих Redis. Если вы используете [Laravel Sail](sail.md), то это расширение уже установлено в контейнере Docker вашего приложения.

Если вы не можете установить расширение phpredis, то установите пакет `predis/predis` через Composer. Predis – это клиент Redis, полностью написанный на PHP и не требующий дополнительных расширений:

```shell
composer require predis/predis
```

<a name="configuration"></a>
## Конфигурирование

Вы можете настроить параметры Redis для своего приложения с помощью конфигурационного файла `config/database.php`. В этом файле вы увидите массив `redis`, содержащий серверы Redis, используемые вашим приложением:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_CACHE_DB', 1),
        ],

    ],

Каждый сервер Redis, определенный в вашем конфигурационном файле, должен иметь имя, хост и порт, либо единый URL соединения Redis:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'url' => 'tcp://127.0.0.1:6379?database=0',
        ],

        'cache' => [
            'url' => 'tls://user:password@127.0.0.1:6380?database=1',
        ],

    ],

<a name="configuring-the-connection-scheme"></a>
#### Настройка схемы подключения

По умолчанию клиенты Redis будут использовать схему `tcp` при подключении к вашим серверам Redis; однако вы можете использовать шифрование TLS / SSL, указав параметр `scheme` конфигурации в массиве конфигурации вашего сервера Redis:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'scheme' => 'tls',
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

    ],

<a name="clusters"></a>
### Кластеры

Если ваше приложение использует кластер серверов Redis, то вы должны определить эти кластеры в ключе `clusters` вашей конфигурации Redis. Этот ключ конфигурации не существует по умолчанию, поэтому вам нужно будет создать его в конфигурационном файле `config/database.php` вашего приложения:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD'),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

По умолчанию кластеры будут выполнять сегментирование на стороне клиента между вашими узлами, что позволяет объединять узлы в пул и создавать большой объем доступной оперативной памяти. Однако сегментирование на стороне клиента не обрабатывает отказоустойчивость; поэтому оно в первую очередь подходит для переходных кешированных данных, доступных из другого первичного хранилища данных.

Если вы хотите использовать собственную кластеризацию Redis вместо сегментирования на стороне клиента, то вы можете указать это, установив значение конфигурации `options.cluster` на `redis` в конфигурационном файле `config/database.php` вашего приложения:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

Если вы хотите, чтобы ваше приложение взаимодействовало с Redis через пакет Predis, то вы должны убедиться, что значение переменной окружения `REDIS_CLIENT` установлено как `predis`:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'predis'),

        //
    ],

Помимо основных параметров конфигурации сервера `host`, `port`, `database` и `password`, Predis поддерживает дополнительные [параметры подключения](https://github.com/nrk/predis/wiki/Connection-Parameters), которые могут быть определены для каждого из ваших серверов Redis. Чтобы использовать эти дополнительные параметры конфигурации, добавьте их в конфигурацию сервера Redis в конфигурационном файле `config/database.php` вашего приложения:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="the-redis-facade-alias"></a>
#### Псевдоним фасада Redis

Конфигурационный файл `config/app.php` содержит массив `aliases`, который определяет все псевдонимы классов, которые будут зарегистрированы фреймворком Laravel. По умолчанию псевдоним `Redis` не включен, поскольку он конфликтует с именем класса `Redis` расширения `phpredis`. Если вы используете клиент Predis и хотите добавить псевдоним `Redis`, то вы можете добавить его в массив `aliases` в конфигурационного файла `config/app.php` вашего приложения:

    'aliases' => Facade::defaultAliases()->merge([
        'Redis' => Illuminate\Support\Facades\Redis::class,
    ])->toArray(),

<a name="phpredis"></a>
### phpredis

По умолчанию Laravel будет использовать расширение phpredis для соединения с Redis. Клиент, который Laravel будет использовать для соединения с Redis, определяется значением параметра `redis.client` конфигурации, который обычно проксирует значение переменной `REDIS_CLIENT` окружения:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        // Остальная часть конфигурации Redis ...
    ],

Помимо основных параметров конфигурации сервера `scheme`, `host`, `port`, `database` и `password`, phpredis поддерживает дополнительные параметры подключения: `name`, `persistent`, `persistent_id`, `prefix`, `read_timeout`, `retry_interval`, `timeout` и `context`. Чтобы использовать эти дополнительные параметры конфигурации, добавьте их в конфигурацию сервера Redis в конфигурационном файле `config/database.php` вашего приложения:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
        'context' => [
            // 'auth' => ['username', 'secret'],
            // 'stream' => ['verify_peer' => false],
        ],
    ],

<a name="phpredis-serialization"></a>
#### Сериализация и сжатие phpredis

Расширение phpredis также можно настроить для использования различных алгоритмов сериализации и сжатия. Эти алгоритмы можно настроить с помощью массива параметров вашей конфигурации Redis:

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'serializer' => Redis::SERIALIZER_MSGPACK,
            'compression' => Redis::COMPRESSION_LZ4,
        ],

        // Остальные настройки Redis...
    ],

В настоящее время поддерживаются следующие алгоритмы сериализации: `Redis::SERIALIZER_NONE` (по умолчанию), `Redis::SERIALIZER_PHP`, `Redis::SERIALIZER_JSON`, `Redis::SERIALIZER_IGBINARY` и `Redis::SERIALIZER_MSGPACK`.

Поддерживаемые алгоритмы сжатия включают в себя: `Redis::COMPRESSION_NONE` (по умолчанию), `Redis::COMPRESSION_LZF`, `Redis::COMPRESSION_ZSTD` и `Redis::COMPRESSION_LZ4`.

<a name="interacting-with-redis"></a>
## Взаимодействие с Redis

Вы можете взаимодействовать с Redis, вызывая различные методы [фасада](facades.md) `Redis`. Фасад `Redis` поддерживает динамические методы, то есть вы можете вызвать любую [команду Redis](https://redis.io/commands), используя фасад, и команда будет передана непосредственно в Redis. В этом примере мы вызовем команду Redis `GET`, вызвав метод `get` фасада `Redis`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;

    class UserController extends Controller
    {
        /**
         * Показать профиль конкретного пользователя.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */
        public function show($id)
        {
            return view('user.profile', [
                'user' => Redis::get('user:profile:'.$id)
            ]);
        }
    }

Как упоминалось выше, вы можете вызывать любую из команд Redis, используя фасад `Redis`. Laravel использует магические методы для передачи команд на сервер Redis. Если команда Redis ожидает аргументов, то вы должны передать их соответствующему методу фасада:

    use Illuminate\Support\Facades\Redis;

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

В качестве альтернативы вы можете передавать команды серверу, используя метод `command` фасада `Redis`, который принимает имя команды в качестве первого аргумента и массив значений в качестве второго аргумента:

    $values = Redis::command('lrange', ['name', 5, 10]);

<a name="using-multiple-redis-connections"></a>
#### Использование нескольких подключений Redis

Конфигурационный файл `config/database.php` вашего приложения позволяет вам определять несколько соединений / серверов Redis. Вы можете получить соединение с конкретным соединением Redis, используя метод `connection` фасада `Redis`:

    $redis = Redis::connection('connection-name');

Чтобы получить экземпляр соединения Redis по умолчанию, вы можете вызвать метод `connection` без каких-либо дополнительных аргументов:

    $redis = Redis::connection();

<a name="transactions"></a>
### Транзакции

Метод `transaction` фасада `Redis` обеспечивает удобную обертку для собственных команд `MULTI` и `EXEC` Redis. Метод `transaction` принимает замыкание как единственный аргумент. Это замыкание получит экземпляр подключения Redis и может использовать любые необходимые вам команды,  отправляемые на сервер Redis. Все команды Redis в рамках замыкания будут выполняться в одной атомарной транзакции:

    use Illuminate\Support\Facades\Redis;

    Redis::transaction(function ($redis) {
        $redis->incr('user_visits', 1);
        $redis->incr('total_visits', 1);
    });

> {note} При определении транзакции Redis вы не можете получать какие-либо значения из соединения Redis. Помните, ваша транзакция выполняется как одна атомарная операция, и эта операция не выполнится, пока не завершится выполнение всех команд замыкания.

#### Скрипты Lua

Метод `eval` обеспечивает другой метод выполнения нескольких команд Redis за одну атомарную операцию. Однако преимущество метода `eval` состоит в том, что он может взаимодействовать со значениями ключей Redis и использовать их во время этой операции. Скрипты Redis написаны на [языке программирования Lua](https://www.lua.org).

Поначалу метод `eval` может показаться немного пугающим, но мы рассмотрим пример. Метод `eval` ожидает несколько аргументов. Во-первых, вы должны передать сценарий Lua (в виде строки) в метод. Во-вторых, вы должны передать количество ключей (в виде целого числа), с которыми скрипт взаимодействует. В-третьих, вы должны передать имена этих ключей. Наконец, вы можете передать любые другие дополнительные аргументы, к которым вам нужно получить доступ в вашем скрипте.

В этом примере мы увеличим счетчик, проверим его новое значение и увеличим второй счетчик, если значение первого счетчика больше пяти. Наконец, мы вернем значение первого счетчика:

    $value = Redis::eval(<<<'LUA'
        local counter = redis.call("incr", KEYS[1])

        if counter > 5 then
            redis.call("incr", KEYS[2])
        end

        return counter
    LUA, 2, 'first-counter', 'second-counter');

> {note} Пожалуйста, обратитесь к [документации Redis](https://redis.io/commands/eval) для получения дополнительных сведений о сценариях Redis.

<a name="pipelining-commands"></a>
### Конвейерное выполнение команд

По желанию можно выполнить десятки команд Redis. Вместо того, чтобы совершать сетевое обращение к вашему серверу Redis для каждой команды, вы можете использовать метод `pipeline`. Метод `pipeline` принимает один аргумент: замыкание, которое получает экземпляр Redis. Вы можете передать все свои команды этому экземпляру Redis, и все они будут отправлены на сервер Redis одновременно, чтобы уменьшить количество сетевых обращений к серверу. Команды по-прежнему будут выполняться в том порядке, в котором они были отправлены:

    use Illuminate\Support\Facades\Redis;

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Публикация / подписка

Laravel предлагает удобный интерфейс для команд `publish` и `subscribe` Redis. Эти команды Redis позволяют вам прослушивать сообщения на указанном «канале». Вы можете публиковать сообщения в канал из другого приложения или даже с использованием другого языка программирования, что позволяет легко взаимодействовать между приложениями и процессами.

Во-первых, давайте настроим слушатель каналов с помощью метода `subscribe`. Мы поместим вызов этого метода в [команду Artisan](artisan.md), поскольку вызов метода `subscribe` запускает длительный процесс:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * Имя и сигнатура консольной команды.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * Описание консольной команды.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Выполнить консольную команду.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

Теперь мы можем публиковать сообщения в канале с помощью метода `publish`:

    use Illuminate\Support\Facades\Redis;

    Route::get('/publish', function () {
        // ...

        Redis::publish('test-channel', json_encode([
            'name' => 'Adam Wathan'
        ]));
    });

<a name="wildcard-subscriptions"></a>
#### Групповые подписки

Допускается использование метасимвола подстановки `*` при использовании метода `psubscribe`, что позволит вам перехватывать все сообщения на нескольких каналах. Имя канала будет передано вторым аргументом в указанное замыкание:

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
