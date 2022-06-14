# Laravel 9 · Руководство по обновлению

- [Обновление с 8.x версии до 9.0](#upgrade-9.0)

<a name="high-impact-changes"></a>
## Изменения, оказывающие большое влияние

<!-- <div class="content-list" markdown="1"> -->

- [Обновление зависимостей](#updating-dependencies)
- [Flysystem 3.x](#flysystem-3)
- [Symfony Mailer](#symfony-mailer)

<!-- </div> -->

<a name="medium-impact-changes"></a>
## Изменения со средней степенью воздействия

<!-- <div class="content-list" markdown="1"> -->

- [Методы `firstOrNew`, `firstOrCreate` и `updateOrCreate` отношений «Belongs To Many»](#belongs-to-many-first-or-new)
- [Пользовательская типизация и `null`](#custom-casts-and-null)
- [Время ожидания HTTP-клиента по умолчанию](#http-client-default-timeout)
- [Возвращаемые типы PHP](#php-return-types)
- [Изменение параметра `schema` для Postgres](#postgres-schema-configuration)
- [Метод `assertDeleted`](#the-assert-deleted-method)
- [Каталог `lang`](#the-lang-directory)
- [Правило `password`](#the-password-rule)
- [Методы `when` / `unless`](#when-and-unless-methods)
- [Непровалидированные ключи массива](#unvalidated-array-keys)

<!-- </div> -->

<a name="upgrade-9.0"></a>
## Обновление с 8.x версии до 9.0

<a name="estimated-upgrade-time-30-minutes"></a>
#### Приблизительное время обновления: 30 минут

> {tip} Мы стараемся задокументировать все возможные критические изменения. Поскольку некоторые из этих критических изменений находятся в малоизвестных частях фреймворка, только часть этих изменений может повлиять на ваше приложение. Хотите сэкономить время? Вы можете использовать [Laravel Shift](https://laravelshift.com/), чтобы автоматизировать обновления приложений.

<a name="updating-dependencies"></a>
### Обновление зависимостей

**Вероятность воздействия: высокая**

#### Требование PHP 8.0.2

Laravel теперь требует PHP 8.0.2 или выше.

#### Зависимости Composer

Вы должны обновить следующие зависимости в файле `composer.json` вашего приложения:

<!-- <div class="content-list" markdown="1"> -->

- `laravel/framework` до `^9.0`
- `nunomaduro/collision` до `^6.1`

<!-- </div> -->

Кроме того, замените `facade/ignition` на `"spatie/laravel-ignition": "^1.0"` в файле `composer.json` вашего приложения.

Кроме того, следующие пакеты получили новые релизы для поддержки Laravel 9.x. Если применимо, то вы должны прочитать их отдельные руководства перед обновлением:

<!-- <div class="content-list" markdown="1"> -->

- [Vonage Notification Channel (v3.0)](https://github.com/laravel/vonage-notification-channel/blob/3.x/UPGRADE.md) (заменяет Nexmo)

<!-- </div> -->

Наконец, проверьте любые другие сторонние пакеты, используемые вашим приложением, и убедитесь, что вы используете корректную версию для поддержки Laravel 9.

<a name="php-return-types"></a>
#### Возвращаемые типы PHP

PHP начинает переходить к требованию определения типа возвращаемого значения в методах PHP, таких как `offsetGet`, `offsetSet` и т. д. В свете этого Laravel 9 реализовал эти возвращаемые типы в своей кодовой базе. Как правило, это не должно влиять на написанный пользователем код; однако, если вы переопределяете один из этих методов, расширяя базовые классы Laravel, то вам нужно будет добавить эти возвращаемые типы в код вашего собственного приложения или пакета:

<!-- <div class="content-list" markdown="1"> -->

- `count(): int`
- `getIterator(): Traversable`
- `getSize(): int`
- `jsonSerialize(): array`
- `offsetExists($key): bool`
- `offsetGet($key): mixed`
- `offsetSet($key, $value): void`
- `offsetUnset($key): void`

<!-- </div> -->

Кроме того, в методы, реализующие `SessionHandlerInterface` PHP, были добавлены возвращаемые типы. Опять же, маловероятно, что это изменение повлияет на ваше собственное приложение или код пакета:

<!-- <div class="content-list" markdown="1"> -->

- `open($savePath, $sessionName): bool`
- `close(): bool`
- `read($sessionId): string|false`
- `write($sessionId, $data): bool`
- `destroy($sessionId): bool`
- `gc($lifetime): int`

<!-- </div> -->

<a name="application"></a>
### Приложение

<a name="the-application-contract"></a>
#### Контракт `Application`

**Вероятность воздействия: низкая**

Метод `storagePath` интерфейса `Illuminate\Contracts\Foundation\Application` был обновлен, чтобы принимать аргумент `$path`. Если вы реализуете этот интерфейс, то вы должны соответствующим образом обновить свою реализацию:

    public function storagePath($path = '');

Точно так же метод `langPath` класса `Illuminate\Foundation\Application` был обновлен, чтобы принимать аргумент `$path`:

    public function langPath($path = '');

#### Метод `ignore` обработчика исключений

**Вероятность воздействия: низкая**

Метод `ignore` обработчика исключений теперь является `public`, а не `protected`. Этот метод не включен в приложение по умолчанию; однако, если вы определили этот метод самостоятельно, то вы должны обновить его до `public`:

```php
public function ignore(string $class);
```

### Шаблонизатор Blade

#### Отложенные коллекции и переменная `$loop`

**Вероятность воздействия: низкая**

При итерации экземпляра `LazyCollection` в шаблоне Blade переменная `$loop` больше недоступна, так как доступ к этой переменной приводит к загрузке всей коллекции в память, что делает использование отложенных коллекций бессмысленным в этом сценарии.

#### Директивы элементов интерфейса Blade: отмеченные, выделенные и отключенные

**Вероятность воздействия: низкая**

Новые директивы `@checked`, `@disabled` и `@selected` Blade могут конфликтовать с одноименными событиями Vue. Вы можете использовать `@@`, чтобы экранировать директивы и избежать подобного конфликта; пример использования экранированной директивы: `@@selected`.

### Коллекции

#### Контракт `Enumerable`

**Вероятность воздействия: низкая**

Контракт `Illuminate\Support\Enumerable` теперь определяет метод `sole`. Если вы реализуете этот контракт самостоятельно, то вам следует обновить свою реализацию, чтобы отразить этот новый метод:

```php
public function sole($key = null, $operator = null, $value = null);
```

#### Метод `reduceWithKeys`

Метод `reduceWithKeys` был удален, так как метод `reduce` обеспечивает ту же функциональность. Вы можете просто обновить свой код, чтобы он вызывал `reduce` вместо `reduceWithKeys`.

#### Метод `reduceMany`

Метод `reduceMany` был переименован в `reduceSpread` для согласованности именования с другими подобными методами.

### Контейнер служб

#### Контракт `Container`

**Вероятность воздействия: очень низкая**

Контракт `Illuminate\Contracts\Container\Container` теперь определяет два метода: `scoped` и `scopedIf`. Если вы реализуете этот контракт самостоятельно, то вам следует обновить свою реализацию, чтобы отразить эти новые методы.

#### Контракт `ContextualBindingBuilder`

**Вероятность воздействия: очень низкая**

Контракт `Illuminate\Contracts\Container\ContextualBindingBuilder` теперь определяет метод `giveConfig`. Если вы реализуете этот контракт самостоятельно, то вам следует обновить свою реализацию, чтобы отразить этот новый метод:

```php
public function giveConfig($key, $default = null);
```

### База данных

<a name="postgres-schema-configuration"></a>
#### Изменение параметра `schema` для Postgres

**Вероятность воздействия: средняя**

Параметр конфигурации `schema`, используемый для настройки путей поиска соединений Postgres в конфигурационном файле `config/database.php` вашего приложения, должен быть переименован в `search_path`.

<a name="schema-builder-doctrine-method"></a>
#### Метод `registerCustomDoctrineType` построителя схемы

**Вероятность воздействия: низкая**

Метод `registerCustomDoctrineType` был удален из класса `Illuminate\Database\Schema\Builder`. Вместо этого вы можете использовать метод `registerDoctrineType` фасада `DB` или зарегистрировать пользовательские типы Doctrine в конфигурационном файле `config/database.php`.

### Модели Eloquent

<a name="custom-casts-and-null"></a>
#### Пользовательская типизация и `null`

**Вероятность воздействия: средняя**

В предыдущих релизах Laravel метод `set` классов пользовательской типизации не вызывался, если для типизируемого атрибута было установлено значение `null`. Однако такое поведение не соответствовало документации Laravel. В Laravel 9.x метод `set` класса типизации будет вызываться с `null` в качестве предоставленного аргумента `$value`. Поэтому вы должны убедиться, что ваши пользовательские типизации способны в достаточной степени справиться с этим сценарием:

```php
/**
 * Подготовить переданное значение к сохранению.
 *
 * @param  \Illuminate\Database\Eloquent\Model  $model
 * @param  string  $key
 * @param  AddressModel  $value
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
```

<a name="belongs-to-many-first-or-new"></a>
#### Методы `firstOrNew`, `firstOrCreate` и `updateOrCreate` отношений «Belongs To Many»

**Вероятность воздействия: средняя**

Методы `firstOrNew`, `firstOrCreate` и `updateOrCreate` отношений «Belongs To Many» принимают массив атрибутов в качестве первого аргумента. В предыдущих релизах Laravel этот массив атрибутов сравнивался со «сводной» / промежуточной таблицей для существующих записей.

Однако такое поведение было неожиданным и, как правило, нежелательным. Вместо этого эти методы теперь сравнивают массив атрибутов с таблицей связанной модели:

```php
$user->roles()->updateOrCreate([
    'name' => 'Administrator',
]);
```

Кроме того, метод `firstOrCreate` теперь принимает массив `$values` ​​в качестве второго аргумента. Этот массив будет объединен с первым аргументом метода (`$attributes`) при создании связанной модели, если она еще не существует. Это изменение делает этот метод совместимым с методами `firstOrCreate` других типов отношений:

```php
$user->roles()->firstOrCreate([
    'name' => 'Administrator',
], [
    'created_by' => $user->id,
]);
```

#### Метод `touch`

**Вероятность воздействия: низкая**

Метод `touch` теперь принимает имя затрагиваемого атрибута. Если вы ранее перезаписывали этот метод, то вам следует обновить сигнатуру метода, чтобы отразить этот новый аргумент:

```php
public function touch($attribute = null);
```

### Шифрование

#### Контракт `Encrypter`

**Вероятность воздействия: низкая**

Контракт `Illuminate\Contracts\Encryption\Encrypter` теперь определяет метод `getKey`. Если вы реализуете этот интерфейс самостоятельно, то вам следует соответствующим образом обновить свою реализацию:

```php
public function getKey();
```

### Фасады

#### Метод `getFacadeAccessor`

**Вероятность воздействия: низкая**

Метод `getFacadeAccessor` всегда должен возвращать ключ привязки контейнера. В предыдущих релизах Laravel этот метод мог возвращать экземпляр объекта; однако это поведение больше не поддерживается. Если вы написали свои собственные фасады, то вы должны убедиться, что этот метод возвращает строку привязки контейнера:

```php
/**
 * Получить зарегистрированное имя компонента.
 *
 * @return string
 */
protected static function getFacadeAccessor()
{
    return Example::class;
}
```

### Файловое хранилище

#### Переменная окружения `FILESYSTEM_DRIVER`

**Вероятность воздействия: низкая**

Переменная окружения `FILESYSTEM_DRIVER` была переименована в `FILESYSTEM_DISK` для более точного отражения ее использования. Это изменение затрагивает только скелет приложения; однако вы можете обновить переменные окружения своего собственного приложения, чтобы отразить это изменение, если хотите.

#### Диск `cloud`

**Вероятность воздействия: низкая**

Параметр конфигурации `cloud` диска был удален из скелета приложения по умолчанию в ноябре 2020 года. Это изменение влияет только на скелет приложения. Если вы используете диск `cloud` в своем приложении, то вы должны оставить это значение конфигурации в скелете вашего собственного приложения.

<a name="flysystem-3"></a>
### Flysystem 3.x

**Вероятность воздействия: высокая**

Laravel 9.x мигрировал с [Flysystem](https://flysystem.thephpleague.com/v2/docs/) версии 1.x на версию 3.x. Под капотом Flysystem используются все методы манипулирования файлами, предоставляемые фасадом `Storage`. В связи с этим в вашем приложении могут потребоваться некоторые изменения; однако мы постарались сделать этот переход максимально плавным.

#### Требования к драйверу

Перед использованием драйверов S3, FTP или SFTP вам необходимо установить соответствующий пакет с помощью менеджера пакетов Composer:

- Amazon S3: `composer require -W league/flysystem-aws-s3-v3 "^3.0"`
- FTP: `composer require league/flysystem-ftp "^3.0"`
- SFTP: `composer require league/flysystem-sftp-v3 "^3.0"`

#### Перезапись существующих файлов

Операции записи, такие как `put`, `write` и `writeStream`, теперь по умолчанию перезаписывают существующие файлы. Если вы не хотите перезаписывать существующие файлы, то вам следует предварительно проверить существование файла перед выполнением операции записи.

#### Выброс исключений при выполнении записи

Операции записи, такие как `put`, `write` и `writeStream`, больше не вызывают исключение при неудавшихся операциях записи. Вместо этого возвращается `false`. Если вы хотите сохранить предыдущее поведение, которое вызывало исключения, то вы можете определить параметр `throw` в конфигурационном массиве файловой системы вашего диска:

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],
```

#### Чтение отсутствующих файлов

Попытка чтения из несуществующего файла теперь возвращает `null`. В предыдущих релизах Laravel возникало исключение `Illuminate\Contracts\Filesystem\FileNotFoundException`.

#### Удаление отсутствующих файлов

Попытка «удалить» несуществующий файл с помощью метода `delete` теперь возвращает `true`.

#### Адаптеры с кешем

Flysystem больше не поддерживает адаптеры с кешем. Таким образом, они были удалены из Laravel, и любая соответствующая конфигурация (например, ключ `cache` в конфигурациях диска) может быть удалена.

#### Пользовательские файловые системы

Небольшие изменения были внесены в шаги, необходимые для регистрации пользовательских драйверов файловой системы. Поэтому, если вы определяли свои собственные пользовательские драйверы файловой системы или использовали пакеты, определяющие пользовательские драйверы, то вам следует обновить свой код и зависимости.

Например, в Laravel 8.x пользовательский драйвер файловой системы может быть зарегистрирован следующим образом:

```php
use Illuminate\Support\Facades\Storage;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

Storage::extend('dropbox', function ($app, $config) {
    $client = new DropboxClient(
        $config['authorization_token']
    );

    return new Filesystem(new DropboxAdapter($client));
});
```

Однако в Laravel 9.x замыкание метода `Storage::extend` должно возвращать экземпляр `Illuminate\Filesystem\FilesystemAdapter` напрямую:

```php
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

Storage::extend('dropbox', function ($app, $config) {
    $adapter = new DropboxAdapter(
        new DropboxClient($config['authorization_token'])
    );

    return new FilesystemAdapter(
        new Filesystem($adapter, $config),
        $adapter,
        $config
    );
});
```

### Глобальные помощники

<a name="data-get-function"></a>
#### Функция `data_get` и итерируемые объекты

**Вероятность воздействия: очень низкая**

Раньше помощник `data_get` можно было использовать для извлечения вложенных данных в массивах и экземплярах класса `Collection`; однако теперь этот помощник может извлекать вложенные данные во всех итерируемых объектах.

<a name="str-function"></a>
#### Функция `str`

**Вероятность воздействия: очень низкая**

Laravel 9.x теперь включает [глобальный помощник `str`](helpers.md#method-str). Если вы самостоятельно определили глобальный помощник `str` в своем приложении, то вы должны переименовать или удалить его, чтобы он не конфликтовал с помощником `str` Laravel.

<a name="when-and-unless-methods"></a>
#### Методы `when` / `unless`

**Вероятность воздействия: средняя**

Как вы, возможно, знаете, методы `when` и `unless` предлагаются различными классами фреймворка. Эти методы можно использовать для условного выполнения действия, если логическое значение первого аргумента метода оценивается как «истина» или «ложь», соответственно:

```php
$collection->when(true, function ($collection) {
    $collection->merge([1, 2, 3]);
});
```

Поэтому в предыдущих релизах Laravel передача замыкания методам `when` или `unless` означала, что условная операция всегда будет выполняться, поскольку гибкое сравнение с объектом замыкания (или любым другим объектом) всегда оценивается как `true`. Это часто приводило к неожиданным результатам, поскольку разработчики ожидали, что **результат** замыкания будет использоваться как логическое значение, определяющее, выполняется ли условное действие.

Таким образом, в Laravel 9.x любые замыкания, переданные методам `when` или `unless`, будут выполнены, а значение, возвращаемое замыканием, будет считаться логическим значением, используемым методами `when` и `unless`:

```php
$collection->when(function ($collection) {
    // Это замыкание будет выполнено ...
    return false;
}, function ($collection) {
    // Не будет выполнено, так как первое замыкание вернуло `false` ...
    $collection->merge([1, 2, 3]);
});
```

### HTTP-клиент

<a name="http-client-default-timeout"></a>
#### Время ожидания по умолчанию для HTTP-клиента

**Вероятность воздействия: средняя**

[HTTP-клиент](http-client.md) теперь имеет время ожидания по умолчанию, равное 30 секундам. Другими словами, если сервер не отвечает в течение 30 секунд, то будет выброшено исключение. Раньше для HTTP-клиента не настраивалось время ожидания по умолчанию, из-за чего запросы иногда «зависали» на неопределенный срок.

Если вы хотите указать более длительное время ожидания для конкретного запроса, то вы можете сделать это с помощью метода `timeout`:

    $response = Http::timeout(120)->get(/* ... */);

#### Имитация HTTP и посредник

**Вероятность воздействия: низкая**

Ранее Laravel не запускал посредников Guzzle HTTP, если [HTTP-клиент](http-client.md) являлся имитацией. Однако в Laravel 9.x посредник Guzzle HTTP будет выполняться даже при имитации HTTP-клиента.

#### Имитация HTTP и внедрение зависимостей

**Вероятность воздействия: низкая**

В предыдущих релизах Laravel вызов метода `Http::fake()` не влиял на экземпляры `Illuminate\Http\Client\Factory`, которые были внедрены в конструкторы классов. Однако в Laravel 9.x `Http::fake()` гарантирует, что фейковые ответы будут возвращены HTTP-клиентами, внедряемыми в другие службы посредством внедрения зависимостей. Такое поведение больше соответствует поведению других фасадов и их имитаций.

<a name="symfony-mailer"></a>
### Symfony Mailer

**Вероятность воздействия: высокая**

Одним из самых больших изменений в Laravel 9.x является переход к Symfony Mailer от SwiftMailer, который с декабря 2021 года больше не поддерживается. Однако мы постарались сделать этот переход как можно более плавным для ваших приложений. При этом внимательно ознакомьтесь со списком изменений ниже, чтобы убедиться, что ваше приложение полностью совместимо.

#### Предварительная подготовка драйверов

Чтобы продолжить использование драйвера Mailgun, вашему приложению требуется пакеты `symfony/mailgun-mailer` и `symfony/http-client` Composer:

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

Пакет `wildbit/swiftmailer-postmark` Composer должен быть удален из вашего приложения. Вместо этого вашему приложению должен потребоваться пакеты `symfony/postmark-mailer` и `symfony/http-client` Composer::

```shell
composer require symfony/postmark-mailer symfony/http-client
```

#### Обновлены возвращаемые типы

Методы `send`, `html`, `raw` и `plain`, принадлежащие `Illuminate\Mail\Mailer` больше не возвращают `void`. Вместо этого возвращается экземпляр `Illuminate\Mail\SentMessage`. Этот объект содержит экземпляр `Symfony\Component\Mailer\SentMessage`, доступный через метод `getSymfonySentMessage` или путем динамического вызова методов объекта.

#### Переименованы методы Swift

Различные методы, связанные со SwiftMailer, некоторые из которых не были задокументированы, были переименованы в их аналоги Symfony Mailer. Например, метод `withSwiftMessage` был переименован в `withSymfonyMessage`:

    // Laravel 8.x...
    $this->withSwiftMessage(function ($message) {
        $message->getHeaders()->addTextHeader(
            'Custom-Header', 'Header Value'
        );
    });

    // Laravel 9.x...
    use Symfony\Component\Mime\Email;

    $this->withSymfonyMessage(function (Email $message) {
        $message->getHeaders()->addTextHeader(
            'Custom-Header', 'Header Value'
        );
    });

> {note} Пожалуйста, внимательно изучите [документацию Symfony Mailer](https://symfony.com/doc/6.0/mailer.html#creating-sending-messages) для взаимодействий с объектом `Symfony\Component\Mime\Email`.

Список ниже содержит более подробный обзор переименованных методов. Многие из этих методов являются низкоуровневыми методами, используемыми для прямого взаимодействия со SwiftMailer / Symfony Mailer, поэтому могут не использоваться в большинстве приложений Laravel:

    Message::getSwiftMessage();
    Message::getSymfonyMessage();

    Mailable::withSwiftMessage($callback);
    Mailable::withSymfonyMessage($callback);

    MailMessage::withSwiftMessage($callback);
    MailMessage::withSymfonyMessage($callback);

    Mailer::getSwiftMailer();
    Mailer::getSymfonyTransport();

    Mailer::setSwiftMailer($swift);
    Mailer::setSymfonyTransport(TransportInterface $transport);

    MailManager::createTransport($config);
    MailManager::createSymfonyTransport($config);

#### Прокси-методы `Illuminate\Mail\Message`

`Illuminate\Mail\Message` обычно проксирует отсутствующие методы базовому экземпляру `Swift_Message`. Однако отсутствующие методы теперь вместо этого проксируются экземпляру `Symfony\Component\Mime\Email`. Таким образом, любой код, который ранее полагался на отсутствующие методы для проксирования SwiftMailer, должен быть обновлен до соответствующих аналогов Symfony Mailer.

Опять же, многие приложения могут не взаимодействовать с этими методами, поскольку они не описаны в документации Laravel:

    // Laravel 8.x...
    $message
        ->setFrom('taylor@laravel.com')
        ->setTo('example@example.org')
        ->setSubject('Order Shipped')
        ->setBody('<h1>HTML</h1>', 'text/html')
        ->addPart('Plain Text', 'text/plain');

    // Laravel 9.x...
    $message
        ->from('taylor@laravel.com')
        ->to('example@example.org')
        ->subject('Order Shipped')
        ->html('<h1>HTML</h1>')
        ->text('Plain Text');

#### Генерация идентификаторов сообщений

SwiftMailer предлагал возможность определить собственный домен для включения в сгенерированные идентификаторы сообщений с помощью параметра конфигурации `mime.idgenerator.idright`. Это не поддерживается в Symfony Mailer. Вместо этого Symfony Mailer автоматически сгенерирует идентификатор сообщения на основе отправителя.

<!-- #### Изменения события `MessageSent`

Свойство `message` события `Illuminate\Mail\Events\MessageSent` теперь представляет собой экземпляр `Symfony\Component\Mime\Email` вместо экземпляра `Swift_Message`. Такое сообщение представляет собой образ электронного письма **до** его отправки.

Кроме того, событие `MessageSent` теперь имеет новое свойство `sent`. Это свойство представляет собой экземпляр `Illuminate\Mail\SentMessage` и информацию об отправленном электронном письме, к примеру, идентификатор сообщения. -->

#### Принудительное переподключение

Больше невозможно принудительно переподключиться к транспорту (например, когда почтовая программа работает через процесс-демон). Вместо этого Symfony Mailer попытается автоматически переподключиться к транспорту и выдаст исключение, если переподключение не удастся.

#### Параметры потока SMTP

Определение параметров потока для транспорта SMTP больше не поддерживается. Вместо этого вы должны определить соответствующие параметры непосредственно в конфигурации, если они поддерживаются. Например, чтобы отключить одноранговую проверку TLS:

    'smtp' => [
        // Laravel 8.x...
        'stream' => [
            'ssl' => [
                'verify_peer' => false,
            ],
        ],

        // Laravel 9.x...
        'verify_peer' => false,
    ],

Чтобы узнать больше о доступных параметрах конфигурации, ознакомьтесь с [документацией Symfony Mailer](https://symfony.com/doc/6.0/mailer.html#transport-setup).

> {note} Несмотря на приведенный выше пример, обычно не рекомендуется отключать проверку SSL, поскольку это создает возможность атак типа [«man-in-the-middle»](https://ru.wikipedia.org/wiki/Атака_посредника).

#### Режим аутентификации SMTP

Определение параметра `auth_mode` SMTP в конфигурационном файле `mail` больше не требуется. Режим аутентификации будет автоматически согласован между Symfony Mailer и SMTP-сервером.

#### Несостоявшиеся получатели

Больше невозможно получить список неуспешных получателей после отправки сообщения. Вместо этого будет выброшено исключение `Symfony\Component\Mailer\Exception\TransportExceptionInterface`, если сообщение не будет отправлено. Вместо того, чтобы полагаться на получение недействительных адресов электронной почты после отправки сообщения, мы рекомендуем вам проверять адреса электронной почты перед отправкой сообщения.

### Пакеты

<a name="the-lang-directory"></a>
#### Каталог `lang`

**Вероятность воздействия: средняя**

В новых приложениях Laravel каталог `resources/lang` теперь находится в корневом каталоге проекта (`lang`). Если ваш пакет публикует языковые файлы в этом каталоге, то вы должны убедиться, что ваш пакет вместо жестко заданного пути публикует файлы в `app()->langPath()`.

<a name="queue"></a>
### Очереди

<a name="the-opis-closure-library"></a>
#### Библиотека `opis/closure`

**Вероятность воздействия: низкая**

Зависимость Laravel от `opis/closure` была заменена на `laravel/serializable-closure`. Это не должно привести к каким-либо критическим изменениям в вашем приложении, если только вы не взаимодействуете с библиотекой `opis/closure` напрямую. Кроме того, были удалены ранее объявленные устаревшими классы `Illuminate\Queue\SerializableClosureFactory` и `Illuminate\Queue\SerializableClosure`. Если вы взаимодействуете с библиотекой `opis/closure` напрямую или используете какой-либо из удаленных классов, вместо этого вы можете использовать [Laravel Serializable Closure](https://github.com/laravel/serializable-closure).

#### Метод `flush` поставщика неудачных заданий

**Вероятность воздействия: низкая**

Метод `flush`, определенный интерфейсом `Illuminate\Queue\Failed\FailedJobProviderInterface`, теперь принимает аргумент `$hours`, который определяет, насколько старым должно быть неудачное задание (в часах), прежде чем оно будет удалено командой `queue:flush`. Если вы самостоятельно реализуете `FailedJobProviderInterface`, то вы должны убедиться, что ваша реализация обновлена, чтобы отразить этот новый аргумент:

```php
public function flush($hours = null);
```

### Сессия

#### Метод `getSession`

**Вероятность воздействия: низкая**

Класс `Symfony\Component\HttpFoundaton\Request`, расширенный классом `Illuminate\Http\Request` Laravel, предлагает метод `getSession` для получения текущего обработчика хранилища сессии. Этот метод не задокументирован Laravel, так как большинство приложений Laravel взаимодействуют с сессией через метод `session` Laravel.

Метод `getSession` ранее возвращал экземпляр `Illuminate\Session\Store` или `null`; однако из-за того, что в версии Symfony 6.x принудительно используется тип возвращаемого значения `Symfony\Component\HttpFoundation\Session\SessionInterface`, то `getSession` теперь корректно возвращает реализацию `SessionInterface` или выбрасывает `Symfony\Component\HttpFoundation\Exception\SessionNotFoundException`, когда сессия недоступна.

### Тестирование

<a name="the-assert-deleted-method"></a>
#### Метод `assertDeleted`

**Вероятность воздействия: средняя**

Все вызовы метода `assertDeleted` должны быть заменена на `assertModelMissing`.

### Доверенные прокси

**Вероятность воздействия: низкая**

Если вы обновляете свой проект Laravel 8 до Laravel 9, импортируя существующий код приложения в совершенно новый скелет приложения Laravel 9, вам может потребоваться обновить посредник «доверенного прокси» вашего приложения.

В вашем файле `app/Http/Middleware/TrustProxies.php` измените `use Fideloper\Proxy\TrustProxies as Middleware` на `use Illuminate\Http\Middleware\TrustProxies as Middleware`.

Там же вы должны обновить определение свойства `$headers`:

```php
// До ...
protected $headers = Request::HEADER_X_FORWARDED_ALL;

// После ...
protected $headers =
    Request::HEADER_X_FORWARDED_FOR |
    Request::HEADER_X_FORWARDED_HOST |
    Request::HEADER_X_FORWARDED_PORT |
    Request::HEADER_X_FORWARDED_PROTO |
    Request::HEADER_X_FORWARDED_AWS_ELB;
```

Наконец, вы можете удалить зависимость `fideloper/proxy` Composer из своего приложения:

```shell
composer remove fideloper/proxy
```

### Валидация

#### Метод `validated` запроса формы

**Вероятность воздействия: низкая**

Метод `validated`, предлагаемый запросами формы, теперь принимает аргументы `$key` и `$default`. Если вы самостоятельно определяете этот метод, то вам следует обновить сигнатуру вашего метода, чтобы отразить эти новые аргументы:

```php
public function validated($key = null, $default = null)
```

<a name="the-password-rule"></a>
#### Правило `password`

**Вероятность воздействия: средняя**

Правило `password`, которое проверяет, соответствует ли заданное входное значение текущему паролю аутентифицированного пользователя, было переименовано в `current_password`.

<a name="unvalidated-array-keys"></a>
#### Непровалидированные ключи массива

**Вероятность воздействия: средняя**

В предыдущих релизах Laravel вам приходилось вручную указывать валидатору Laravel исключать непроверенные ключи массива из «провалидированных» данных, которые он возвращает, особенно в сочетании с правилом `array`, которое не указывает список разрешенных ключей.

Однако в Laravel 9.x непроверенные ключи массива всегда исключаются из «провалидированных» данных, даже если в правиле `array` не указаны разрешенные ключи. Как правило, такое поведение является наиболее ожидаемым поведением, и предыдущий метод `excludeUnvalidatedArrayKeys` был добавлен в Laravel 8.x только как временная мера для сохранения обратной совместимости.

Хотя это не рекомендуется, но вы можете отказаться от предыдущего поведения Laravel 8.x, вызвав новый метод `includeUnvalidatedArrayKeys` в методе `boot` одного из поставщика служб вашего приложения:

```php
use Illuminate\Support\Facades\Validator;

/**
 * Загрузка любых служб приложения.
 *
 * @return void
 */
public function boot()
{
    Validator::includeUnvalidatedArrayKeys();
}
```

<a name="miscellaneous"></a>
### Разное

Мы также рекомендуем вам просматривать изменения в GitHub-репозитории [`laravel/laravel`](https://github.com/laravel/laravel). Хотя многие из этих изменений могут быть неважны, но вы можете синхронизировать эти файлы с вашим приложением. Некоторые из этих изменений будут рассмотрены в данном руководстве по обновлению, но другие, такие как изменения файлов конфигурации или комментарии, не будут. Вы можете легко просмотреть изменения с помощью [инструмента сравнения GitHub](https://github.com/laravel/laravel/compare/8.x...9.x) и выбрать, какие обновления важны для вас.
