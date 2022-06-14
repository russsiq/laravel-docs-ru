# Laravel 9 · Файловое хранилище

- [Введение](#introduction)
- [Конфигурирование](#configuration)
    - [Локальный драйвер](#the-local-driver)
    - [Публичный диск](#the-public-disk)
    - [Предварительная подготовка драйверов](#driver-prerequisites)
    - [Файловые системы, совместимые с Amazon S3](#amazon-s3-compatible-filesystems)
- [Доступ к экземплярам дисков](#obtaining-disk-instances)
    - [Диски по запросу](#on-demand-disks)
- [Получение файлов](#retrieving-files)
    - [Скачивание файлов](#downloading-files)
    - [URL-адреса файлов](#file-urls)
    - [Метаданные файла](#file-metadata)
- [Хранение файлов](#storing-files)
    - [Добавление информации к файлам](#prepending-appending-to-files)
    - [Копирование и перемещение файлов](#copying-moving-files)
    - [Автоматическая потоковая передача](#automatic-streaming)
    - [Загрузка файлов](#file-uploads)
    - [Видимость файла](#file-visibility)
- [Удаление файлов](#deleting-files)
- [Каталоги](#directories)
- [Пользовательские файловые системы](#custom-filesystems)

<a name="introduction"></a>
## Введение

Laravel обеспечивает мощную абстракцию файловой системы благодаря замечательному пакету [Flysystem](https://github.com/thephpleague/flysystem) PHP от Фрэнка де Йонга. Интеграция Laravel с Flysystem содержит простые драйверы для работы с локальными файловыми системами, SFTP и Amazon S3. Более того, удивительно просто переключаться между этими вариантами хранения: как локального, так и производственного серверов – поскольку API остается одинаковым для каждой системы.

<a name="configuration"></a>
## Конфигурирование

Файл конфигурации файловой системы Laravel находится в `config/filesystems.php`. В этом файле вы можете настроить все «диски» файловой системы. Каждый диск представляет собой определенный драйвер хранилища и место хранения. Примеры конфигураций для каждого поддерживаемого драйвера включены в конфигурационный файл, так что вы можете изменить конфигурацию, отражающую ваши предпочтения хранения и учетные данные.

Драйвер `local` взаимодействует с файлами, хранящимися локально на сервере, на котором запущено приложение Laravel, в то время как драйвер `s3` используется для записи в службу облачного хранилища Amazon S3.

> {tip} Вы можете настроить столько дисков, сколько захотите, и даже иметь несколько дисков, использующих один и тот же драйвер.

<a name="the-local-driver"></a>
### Локальный драйвер

При использовании драйвера `local` все операции с файлами выполняются относительно корневого каталога, определенного в файле конфигурации `filesystems`. По умолчанию это значение задано каталогом `storage/app`. Следовательно, следующий метод запишет файл в `storage/app/example.txt`:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('local')->put('example.txt', 'Contents');

<a name="the-public-disk"></a>
### Публичный диск

Диск `public`, определенный в файле конфигурации `filesystems` вашего приложения, предназначен для файлов, которые будут общедоступными. По умолчанию публичный диск использует драйвер `local` и хранит свои файлы в `storage/app/public`.

Чтобы сделать эти файлы доступными из интернета, вы должны создать символическую ссылку на `storage/app/public` в `public/storage`. Использование этого соглашения о папках позволит хранить ваши публичные файлы в одном каталоге, который может быть легко доступен между развертываниями при использовании систем развертывания с нулевым временем простоя, таких как [Envoyer](https://envoyer.io).

Чтобы создать символическую ссылку, вы можете использовать команду `storage:link` Artisan:

```shell
php artisan storage:link
```

После того, как была создана символическая ссылка, вы можете создавать URL-адреса для сохраненных файлов, используя помощник `asset`:

    echo asset('storage/file.txt');

Вы можете настроить дополнительные символические ссылки в файле конфигурации `filesystems`. Каждая из настроенных ссылок будет создана, когда вы запустите команду `storage:link`:

    'links' => [
        public_path('storage') => storage_path('app/public'),
        public_path('images') => storage_path('app/images'),
    ],

<a name="driver-prerequisites"></a>
### Предварительная подготовка драйверов

<a name="s3-driver-configuration"></a>
#### Конфигурирование драйвера S3

Перед использованием драйвера S3 вам необходимо установить пакет Flysystem S3 с помощью менеджера пакетов Composer:

```shell
composer require league/flysystem-aws-s3-v3 "^3.0"
```

Информация о конфигурации драйвера S3 находится в вашем файле конфигурации `config/filesystems.php`. Этот файл содержит пример массива конфигурации для драйвера S3. Вы можете изменить этот массив своей собственной конфигурацией S3 и учетными данными. Для удобства эти переменные среды соответствуют соглашению об именах, используемому в интерфейсе командной строки AWS.

<a name="ftp-driver-configuration"></a>
#### Конфигурирование драйвера FTP

Перед использованием драйвера FTP вам необходимо установить пакет Flysystem FTP с помощью менеджера пакетов Composer:

```shell
composer require league/flysystem-ftp "^3.0"
```

Интеграция Laravel с Flysystem отлично работает с FTP; однако, пример конфигурации по умолчанию не включен в конфигурационный файл `config/filesystems.php` фреймворка. Если вам нужно настроить файловую систему FTP, вы можете использовать пример конфигурации ниже:

    'ftp' => [
        'driver' => 'ftp',
        'host' => env('FTP_HOST'),
        'username' => env('FTP_USERNAME'),
        'password' => env('FTP_PASSWORD'),

        // Optional FTP Settings...
        // 'port' => env('FTP_PORT', 21),
        // 'root' => env('FTP_ROOT'),
        // 'passive' => true,
        // 'ssl' => true,
        // 'timeout' => 30,
    ],

<a name="sftp-driver-configuration"></a>
#### Конфигурирование драйвера SFTP

Перед использованием драйвера SFTP вам необходимо установить пакет Flysystem SFTP с помощью менеджера пакетов Composer:

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

Интеграция Laravel с Flysystem отлично работает с SFTP; однако, пример конфигурации по умолчанию не включен в конфигурационный файл `config/filesystems.php` фреймворка. Если вам нужно настроить файловую систему SFTP, вы можете использовать пример конфигурации ниже:

    'sftp' => [
        'driver' => 'sftp',
        'host' => env('SFTP_HOST'),

        // Settings for basic authentication...
        'username' => env('SFTP_USERNAME'),
        'password' => env('SFTP_PASSWORD'),

        // Settings for SSH key based authentication with encryption password...
        'privateKey' => env('SFTP_PRIVATE_KEY'),
        'password' => env('SFTP_PASSWORD'),

        // Optional SFTP Settings...
        // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
        // 'maxTries' => 4,
        // 'passphrase' => env('SFTP_PASSPHRASE'),
        // 'port' => env('SFTP_PORT', 22),
        // 'root' => env('SFTP_ROOT', ''),
        // 'timeout' => 30,
        // 'useAgent' => true,
    ],

<a name="amazon-s3-compatible-filesystems"></a>
### Файловые системы, совместимые с Amazon S3

По умолчанию конфигурационный файл `filesystems` вашего приложения содержит конфигурацию диска `s3`. Помимо использования этого диска для взаимодействия с Amazon S3, вы можете использовать его для взаимодействия с любой совместимой с S3 службой хранения файлов, такой как [MinIO](https://github.com/minio/minio) или [DigitalOcean Spaces](https://www.digitalocean.com/products/spaces/).

Обычно после обновления учетных данных диска в соответствии с учетными данными службы, которую вы планируете использовать, вам нужно обновить только значение параметра конфигурации `url`. Значение этого параметра обычно определяется через переменную окружения `AWS_ENDPOINT`:

    'endpoint' => env('AWS_ENDPOINT', 'https://minio:9000'),

<a name="obtaining-disk-instances"></a>
## Доступ к экземплярам дисков

Фасад `Storage` используется для взаимодействия с любым из ваших сконфигурированных дисков. Например, вы можете использовать метод `put` фасада, чтобы сохранить аватар на диске по умолчанию. Если вы вызываете методы фасада `Storage` без предварительного вызова метода `disk`, то метод будет проксирован на диск по умолчанию:

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $content);

Если ваше приложение взаимодействует с несколькими дисками, то вы можете использовать метод `disk` фасада `Storage` для работы с файлами на указанном диске:

    Storage::disk('s3')->put('avatars/1', $content);

<a name="on-demand-disks"></a>
### Диски по запросу

По желанию можно создать диск на лету, используя конкретную конфигурацию, без ее фактического присутствия в конфигурационном файле `filesystems` вашего приложения. Для этого вы можете передать массив конфигурации методу `build` фасада `Storage`:

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

<a name="retrieving-files"></a>
## Получение файлов

Метод `get` используется для получения содержимого файла. Необработанное строковое содержимое файла будет возвращено методом. Помните, что все пути к файлам должны быть указаны относительно «корня» диска:

    $contents = Storage::get('file.jpg');

Метод `exists` используется для определения, существует ли файл на диске:

    if (Storage::disk('s3')->exists('file.jpg')) {
        // ...
    }

Метод `missing` используется, чтобы определить, отсутствует ли файл на диске:

    if (Storage::disk('s3')->missing('file.jpg')) {
        // ...
    }

<a name="downloading-files"></a>
### Скачивание файлов

Метод `download` используется для генерации ответа, который заставляет браузер пользователя загружать файл по указанному пути. Метод `download` принимает имя файла в качестве второго аргумента метода, определяющий имя файла, которое видит пользователь, скачивающий этот файл. Наконец, вы можете передать массив заголовков HTTP в качестве третьего аргумента метода:

    return Storage::download('file.jpg');

    return Storage::download('file.jpg', $name, $headers);

<a name="file-urls"></a>
### URL-адреса файлов

Вы можете использовать метод `url`, чтобы получить URL для указанного файла. Если вы используете драйвер `local`, он обычно просто добавляет `/storage` к указанному пути и возвращает относительный URL-адрес файла. Если вы используете драйвер `s3`, будет возвращен абсолютный внешний URL-адрес:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file.jpg');

При использовании драйвера `local` все файлы, которые должны быть общедоступными, должны быть помещены в каталог `storage/app/public`. Кроме того, вы должны [создать символическую ссылку](#the-public-disk) в `public/storage`, которая указывает на каталог `storage/app/public`.

> {note} При использовании драйвера `local` возвращаемое значение `url` не является URL-кодированным. По этой причине мы рекомендуем всегда хранить ваши файлы, используя имена, которые будут создавать допустимые URL-адреса.

<a name="temporary-urls"></a>
#### Временные URL

Используя метод `temporaryUrl`, вы можете создавать временные URL-адреса для файлов, хранящихся с помощью драйвера `s3`. Этот метод принимает путь и экземпляр `DateTime`, указывающий, когда должен истечь доступ к файлу по URL:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

Если вам нужно указать дополнительные [параметры запроса S3](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests), то вы можете передать массив параметров запроса в качестве третьего аргумент методу `temporaryUrl`:

    $url = Storage::temporaryUrl(
        'file.jpg',
        now()->addMinutes(5),
        [
            'ResponseContentType' => 'application/octet-stream',
            'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
        ]
    );

Если вам нужно изменить способ создания временных URL-адресов для определенного диска хранилища, то вы можете использовать метод `buildTemporaryUrlsUsing`. Например, это может быть полезно, если у вас есть контроллер, позволяющий загружать файлы, хранящиеся на диске, который обычно не поддерживает временные URL-адреса. Как правило, вызов этого метода осуществляется в методе `boot` [поставщика](providers.md):

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Storage;
    use Illuminate\Support\Facades\URL;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Загрузка любых служб приложения.
         *
         * @return void
         */
        public function boot()
        {
            Storage::disk('local')->buildTemporaryUrlsUsing(function ($path, $expiration, $options) {
                return URL::temporarySignedRoute(
                    'files.download',
                    $expiration,
                    array_merge($options, ['path' => $path])
                );
            });
        }
    }

<a name="url-host-customization"></a>
#### Настройка хоста URL

Если вы хотите заранее определить хост для URL-адресов, сгенерированных с помощью фасада `Storage`, то вы можете добавить параметр `url` в массив конфигурации диска:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### Метаданные файла

Помимо чтения и записи файлов, Laravel также может предоставлять информацию о самих файлах. Например, метод `size` используется для получения размера файла в байтах:

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

Метод `lastModified` возвращает временную метку UNIX последнего изменения файла:

    $time = Storage::lastModified('file.jpg');

<a name="file-paths"></a>
#### Пути к файлам

Вы можете использовать метод `path`, чтобы получить путь к указанному файлу. Если вы используете драйвер `local`, он вернет абсолютный путь к файлу. Если вы используете драйвер `s3`, этот метод вернет относительный путь к файлу в корзине `S3`:

    use Illuminate\Support\Facades\Storage;

    $path = Storage::path('file.jpg');

<a name="storing-files"></a>
## Хранение файлов

Метод `put` используется для сохранения содержимого файла на диске. Вы также можете передать `resource` PHP методу `put`, который будет использовать поддержку базового потока Flysystem. Помните, что все пути к файлам должны быть указаны относительно «корневого» расположения, настроенного для диска:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

<a name="failed-writes"></a>
#### Неудавшиеся операции записи

Если метод `put` (или другие операции «записи») не может записать файл на диск, то будет возвращено `false`:

    if (! Storage::put('file.jpg', $contents)) {
        // Не удалось записать файл на диск...
    }

Если хотите, то вы можете определить параметр `throw` в конфигурационном массиве файловой системы вашего диска. Когда этот параметр определен как `true`, то методы «записи», такие как `put`, будут генерировать экземпляр `League\Flysystem\UnableToWriteFile` при неудавшихся операциях записи:

    'public' => [
        'driver' => 'local',
        // ...
        'throw' => true,
    ],

<a name="prepending-appending-to-files"></a>
### Добавление информации к файлам

Методы `prepend` и `append` позволяют записывать в начало или конец файла, соответственно:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

<a name="copying-moving-files"></a>
### Копирование и перемещение файлов

Метод `copy` используется для копирования существующего файла в новое место на диске, а метод `move` используется для переименования или перемещения существующего файла в новое место:

    Storage::copy('old/file.jpg', 'new/file.jpg');

    Storage::move('old/file.jpg', 'new/file.jpg');

<a name="automatic-streaming"></a>
### Автоматическая потоковая передача

Потоковая передача файлов в хранилище позволяет значительно сократить использование памяти. Если вы хотите, чтобы Laravel автоматически управлял потоковой передачей переданного файла в ваше хранилище, вы можете использовать методы `putFile` или `putFileAs`. Эти методы принимают экземпляр `Illuminate\Http\File` или `Illuminate\Http\UploadedFile` и автоматически передают файл в нужное место:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // Автоматически генерировать уникальный идентификатор для имени файла ...
    $path = Storage::putFile('photos', new File('/path/to/photo'));

    // Явно указать имя файла ...
    $path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

Следует отметить несколько важных моментов, касающихся метода `putFile`. Обратите внимание, что мы указали только имя каталога, а не имя файла. По умолчанию метод `putFile` генерирует уникальный идентификатор, который будет служить именем файла. Расширение файла будет определено путем проверки MIME-типа файла. Путь к файлу будет возвращен методом `putFile`, так что вы можете сохранить путь, включая сгенерированное имя файла, в вашей базе данных.

Методы `putFile` и `putFileAs` также принимают аргумент для определения «видимости» сохраненного файла. Это особенно полезно, если вы храните файл на облачном диске, таком как Amazon S3, и хотите, чтобы файл был общедоступным через сгенерированные URL:

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

<a name="file-uploads"></a>
### Загрузка файлов

В веб-приложениях одним из наиболее распространенных вариантов хранения файлов является хранение загруженных пользователем файлов, таких как фотографии и документы. Laravel упрощает хранение загруженных файлов с помощью метода `store` экземпляра загружаемого файла. Вызовите метод `store`, указав путь, по которому вы хотите сохранить загруженный файл:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UserAvatarController extends Controller
    {
        /**
         * Обновить аватар пользователя.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

В этом примере следует отметить несколько важных моментов. Обратите внимание, что мы указали только имя каталога, а не имя файла. По умолчанию метод `store` генерирует уникальный идентификатор, который будет служить именем файла. Расширение файла будет определено путем проверки MIME-типа файла. Путь к файлу будет возвращен методом `store`, поэтому вы можете сохранить путь, включая сгенерированное имя файла, в своей базе данных.

Вы также можете вызвать метод `putFile` фасада `Storage`, чтобы выполнить ту же операцию сохранения файлов, что и в примере выше:

    $path = Storage::putFile('avatars', $request->file('avatar'));

<a name="specifying-a-file-name"></a>
#### Указание имени файла

Если вы не хотите, чтобы имя файла автоматически присваивалось вашему сохраненному файлу, вы можете использовать метод `storeAs`, который получает путь, имя файла и (необязательный) диск в качестве аргументов:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

Вы также можете использовать метод `putFileAs` фасада `Storage`, который будет выполнять ту же операцию сохранения файлов, что и в примере выше:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

> {note} Непечатаемые и недопустимые символы Unicode будут автоматически удалены из путей к файлам. По этой причине, вы _по желанию_ можете очистить пути к файлам перед их передачей в методы хранения файлов Laravel. Пути к файлам нормализуются с помощью метода `League\Flysystem\WhitespacePathNormalizer::normalizePath`.

<a name="specifying-a-disk"></a>
#### Указание диска

По умолчанию метод `store` загружаемого файла будет использовать ваш диск по умолчанию. Если вы хотите указать другой диск, передайте имя диска в качестве второго аргумента методу `store`:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

Если вы используете метод `storeAs`, вы можете передать имя диска в качестве третьего аргумента метода:

    $path = $request->file('avatar')->storeAs(
        'avatars',
        $request->user()->id,
        's3'
    );

<a name="other-uploaded-file-information"></a>
#### Другая информация о загружаемом файле

Если вы хотите получить оригинальное имя и расширение загружаемого файла, то вы можете сделать это с помощью методов `getClientOriginalName` и `getClientOriginalExtension`:

    $file = $request->file('avatar');

    $name = $file->getClientOriginalName();
    $extension = $file->getClientOriginalExtension();

Однако имейте в виду, что методы `getClientOriginalName` и `getClientOriginalExtension` считаются небезопасными, так как имя и расширение файла могут быть подделаны злоумышленником. По этой причине вы должны предпочесть методы `hashName` и `extension`, чтобы получить имя и расширение загружаемого файла:

    $file = $request->file('avatar');

    $name = $file->hashName(); // Создание уникального случайного имени ...
    $extension = $file->extension(); // Определение расширения файла на основе MIME-типа файла ...

<a name="file-visibility"></a>
### Видимость файла

В интеграции Laravel Flysystem «видимость» – это абстракция прав доступа к файлам на нескольких платформах. Файлы могут быть объявлены `public` или `private`. Когда файл объявляется `public`, вы указываете, что файл обычно должен быть доступен для других. Например, при использовании драйвера `s3` вы можете получить URL-адреса для `public` файлов.

Вы можете задать видимость при записи файла с помощью метода `put`:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

Если файл уже был сохранен, его видимость может быть получена и задана с помощью методов `getVisibility` и `setVisibility`, соответственно:

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public');

При взаимодействии с загружаемыми файлами, вы можете использовать методы `storePublicly` и `storePubliclyAs` для сохранения загружаемого файла с видимостью `public`:

    $path = $request->file('avatar')->storePublicly('avatars', 's3');

    $path = $request->file('avatar')->storePubliclyAs(
        'avatars',
        $request->user()->id,
        's3'
    );

<a name="local-files-and-visibility"></a>
#### Локальные файлы и видимость

При использовании драйвера `local`, [видимость](#file-visibility) `public` интерпретируется в право доступа `0755` для каталогов и право доступа `0644` для файлов. Вы можете изменить сопоставление прав доступа в файле конфигурации `filesystems` вашего приложения:

    'local' => [
        'driver' => 'local',
        'root' => storage_path('app'),
        'permissions' => [
            'file' => [
                'public' => 0644,
                'private' => 0600,
            ],
            'dir' => [
                'public' => 0755,
                'private' => 0700,
            ],
        ],
    ],

<a name="deleting-files"></a>
## Удаление файлов

Метод `delete` принимает имя одного файла или массив имен файлов для удаления:

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file.jpg', 'file2.jpg']);

При необходимости вы можете указать диск, с которого следует удалить файл:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('path/file.jpg');

<a name="directories"></a>
## Каталоги

<a name="get-all-files-within-a-directory"></a>
#### Получение всех файлов каталога

Метод `files` возвращает массив всех файлов указанного каталога. Если вы хотите получить список всех файлов каталога, включая все подкаталоги, вы можете использовать метод `allFiles`:

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

<a name="get-all-directories-within-a-directory"></a>
#### Получение всех каталогов из каталога

Метод `directories` возвращает массив всех каталогов указанного каталога. Кроме того, вы можете использовать метод `allDirectories`, чтобы получить список всех каталогов внутри указанного каталога и всех его подкаталогов:

    $directories = Storage::directories($directory);

    $directories = Storage::allDirectories($directory);

<a name="create-a-directory"></a>
#### Создание каталога

Метод `makeDirectory` создаст указанный каталог, включая все необходимые подкаталоги:

    Storage::makeDirectory($directory);

<a name="delete-a-directory"></a>
#### Удаление каталога

Наконец, для удаления каталога и всех его файлов можно использовать метод `deleteDirectory`:

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## Пользовательские файловые системы

Интеграция Laravel с Flysystem обеспечивает поддержку нескольких «драйверов» из коробки; однако, Flysystem этим не ограничивается и имеет адаптеры для многих других систем хранения. Вы можете создать собственный драйвер, если хотите использовать один из этих дополнительных адаптеров в своем приложении Laravel.

Чтобы определить собственную файловую систему, вам понадобится адаптер Flysystem. Давайте добавим в наш проект адаптер Dropbox, поддерживаемый сообществом:

```shell
composer require spatie/flysystem-dropbox
```

Затем вы можете зарегистрировать драйвер в методе `boot` одного из [поставщиков служб](providers.md) вашего приложения. Для этого вы должны использовать метод `extend` фасада `Storage`:

    <?php

    namespace App\Providers;

    use Illuminate\Filesystem\FilesystemAdapter;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Filesystem;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

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
            Storage::extend('dropbox', function ($app, $config) {
                $adapter = new DropboxAdapter(new DropboxClient(
                    $config['authorization_token']
                ));

                return new FilesystemAdapter(
                    new Filesystem($adapter, $config),
                    $adapter,
                    $config
                );
            });
        }
    }

Первый аргумент метода `extend` – это имя драйвера, а второй – замыкание, которое получает переменные `$app` и `$config`. Замыкание должно возвращать экземпляр `Illuminate\Filesystem\FilesystemAdapter`. Переменная `$config` содержит значения, определенные в `config/filesystems.php` для указанного диска.

После того, как вы создали и зарегистрировали расширение поставщика службы, вы можете использовать драйвер `dropbox` в вашем файле конфигурации `config/filesystems.php`.
