# Laravel 9 · Почтовые отправления

- [Введение](#introduction)
    - [Конфигурирование](#configuration)
    - [Предварительная подготовка драйверов](#driver-prerequisites)
    - [Аварийное конфигурирование](#failover-configuration)
- [Генерация отправлений](#generating-mailables)
- [Написание отправлений](#writing-mailables)
    - [Конфигурирование отправителя](#configuring-the-sender)
    - [Конфигурирование шаблона](#configuring-the-view)
    - [Данные шаблона](#view-data)
    - [Вложения](#attachments)
    - [Встраиваемые вложения](#inline-attachments)
    - [Прикрепляемые объекты](#attachable-objects)
    - [Заголовки](#headers)
    - [Теги и метаданные](#tags-and-metadata)
    - [Настройка сообщения Symfony](#customizing-the-symfony-message)
- [Отправления с разметкой Markdown](#markdown-mailables)
    - [Генерация отправлений с разметкой Markdown](#generating-markdown-mailables)
    - [Написание сообщений с разметкой Markdown](#writing-markdown-messages)
    - [Изменение компонентов](#customizing-the-components)
- [Отправка почты](#sending-mail)
    - [Очередь почты](#queueing-mail)
- [Отрисовка отправлений](#rendering-mailables)
    - [Предварительный просмотр отправлений в браузере](#previewing-mailables-in-the-browser)
- [Локализация отправлений](#localizing-mailables)
- [Тестирование отправлений](#testing-mailables)
- [Почта и локальная разработка](#mail-and-local-development)
- [События](#events)
- [Пользовательские драйвера](#custom-transports)
    - [Дополнительные драйвера Symfony](#additional-symfony-transports)

<a name="introduction"></a>
## Введение

Отправка электронной почты не должна быть сложной. Laravel предлагает чистый и простой почтовый API на базе популярного компонента [Symfony Mailer](https://symfony.com/doc/6.0/mailer.html). Laravel и Symfony Mailer обеспечены драйверами для отправки электронной почты через SMTP, Mailgun, Postmark, Amazon SES и `sendmail`, что позволяет быстро начать отправку почты через локальный или облачный сервис по вашему выбору.

<a name="configuration"></a>
### Конфигурирование

Почтовые службы Laravel могут быть настроены через конфигурационный файл `config/mail.php` вашего приложения. Каждая почтовая программа, настроенная в этом файле, может иметь свою собственную уникальную конфигурацию и даже свой собственный уникальный «транспорт», что позволяет вашему приложению использовать различные почтовые службы для отправки определенных сообщений электронной почты. Например, ваше приложение может использовать Postmark для отправки транзакционных писем, а Amazon SES – для массовых рассылок.

В конфигурационном файле `config/mail.php` вы найдете массив `mailers`. Этот массив содержит образец записи конфигурации для каждого из основных почтовых драйверов / транспортов, поддерживаемых Laravel, в то время как значение конфигурации `default` определяет, какая почтовая программа будет использоваться по умолчанию, когда ваше приложение должно отправить сообщение электронной почты.

<a name="driver-prerequisites"></a>
### Предварительная подготовка драйверов

Драйверы на основе API, такие как Mailgun и Postmark, часто проще в использовании и быстрее, чем отправка почты через SMTP-серверы. По возможности мы рекомендуем использовать один из этих драйверов.

<a name="mailgun-driver"></a>
#### Драйвер Mailgun

Чтобы использовать драйвер Mailgun, сначала установите Symfony Mailgun Mailer с помощью менеджера пакетов Composer:

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

Затем установите параметру `default` в вашем конфигурационном файле `config/mail.php` значение `mailgun`. Затем убедитесь, что ваш конфигурационный файл `config/services.php` содержит следующие параметры:

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
],
```

Если вы не используете [регион Mailgun](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions) США, то вы можете определить конечную точку своего региона в конфигурации файла `services`:

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
],
```

<a name="postmark-driver"></a>
#### Драйвер Postmark

Чтобы использовать драйвер Postmark, сначала установите Symfony Postmark Mailer с помощью менеджера пакетов Composer:

```shell
composer require symfony/postmark-mailer symfony/http-client
```

Затем установите параметру `default` в вашем конфигурационном файле `config/mail.php` значение `postmark`. Затем убедитесь, что ваш конфигурационный файл `config/services.php` содержит следующие параметры:

```php
'postmark' => [
    'token' => env('POSTMARK_TOKEN'),
],
```

Если вы хотите указать поток сообщений Postmark, который должен использоваться данной почтовой программой, вы можете добавить параметр конфигурации `message_stream_id` в массив конфигурации почтовой программы. Этот массив конфигурации можно найти в файле конфигурации вашего приложения `config/mail.php`:

```php
'postmark' => [
    'transport' => 'postmark',
    'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
],
```

Таким образом, вы также можете настроить несколько почтовых программ Postmark с разными потоками сообщений.

<a name="ses-driver"></a>
#### Драйвер SES

Чтобы использовать драйвер Amazon SES, сначала необходимо установить Amazon AWS SDK для PHP. Вы можете установить эту библиотеку через менеджер пакетов Composer:

```shell
composer require aws/aws-sdk-php
```

Затем установите для параметра `default` в вашем файле конфигурации `config/mail.php` значение `ses` и убедитесь, что конфигурационный файл `config/services.php` содержит следующие параметры:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
```

Чтобы использовать [временные учетные данные](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) AWS через токен сессии, вы можете добавить ключ `token` в конфигурацию SES вашего приложения:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'token' => env('AWS_SESSION_TOKEN'),
],
```

Если вы хотите определить [дополнительные параметры](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sesv2-2019-09-27.html#sendemail), которые Laravel должен передать методу `SendEmail` AWS SDK при отправке сообщения электронной почты, вы можете определить массив `options` в конфигурации `ses`:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'options' => [
        'ConfigurationSetName' => 'MyConfigurationSet',
        'EmailTags' => [
            ['Name' => 'foo', 'Value' => 'bar'],
        ],
    ],
],
```

<a name="failover-configuration"></a>
### Аварийное конфигурирование

Иногда внешняя служба, которую вы настроили для отправки почты своего приложения, может не работать. В этих случаях может быть полезно определить одну или несколько резервных конфигураций доставки почты, которые будут использоваться в случае, если ваш основной драйвер доставки не работает.

Для этого вы должны определить почтовую программу в конфигурационном файле `config/mail.php` вашего приложения, который использует транспорт `failover`. Массив конфигурации для почтовой программы `failover` вашего приложения должен содержать массив `mailers` в порядке, в котором почтовые драйверы должны быть выбраны для доставки:

```php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'postmark',
            'mailgun',
            'sendmail',
        ],
    ],

    // ...
],
```

После того, как ваш аварийный почтовый агент был определен, вы должны установить его как почтовую программу по умолчанию, используемую вашим приложением, указав его имя как значение конфигурационного ключа `default` в конфигурационном файле `config/mail.php` вашего приложения:

```php
'default' => env('MAIL_MAILER', 'failover'),
```

<a name="generating-mailables"></a>
## Генерация отправлений

При создании приложений Laravel каждый тип электронной почты, отправляемой вашим приложением, представляется экземпляром класса `Illuminate\Mail\Mailable`. Эти классы хранятся в каталоге `app/Mail`. Не беспокойтесь, если вы не видите этот каталог в своем приложении, поскольку он будет сгенерирован для вас, когда вы создадите свой первый почтовый класс с помощью команды `make:mail` [Artisan](artisan.md):

```shell
php artisan make:mail OrderShipped
```

<a name="writing-mailables"></a>
## Написание отправлений

После того, как вы создали почтовый класс, откройте его, чтобы мы могли изучить его содержимое. Конфигурация почтового класса `Mailable` выполняется несколькими методами: `envelope`, `content` и `attachments`.

Метод `envelope` возвращает объект `Illuminate\Mail\Mailables\Envelope`, который определяет тему и, иногда, получателей сообщения. Метод `content` возвращает объект `Illuminate\Mail\Mailables\Content`, который определяет [шаблон Blade](blade.md), используемый для создания содержимого сообщения.

<a name="configuring-the-sender"></a>
### Конфигурирование отправителя

<a name="using-the-envelope"></a>
#### Использование конверта

Во-первых, давайте рассмотрим настройку отправителя электронного письма. Или, другими словами, от кого будет электронное письмо. Настроить отправителя можно двумя способами. Во-первых, вы можете указать адрес `from` на конверте вашего сообщения:

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * Получить конверт сообщения.
 *
 * @return \Illuminate\Mail\Mailables\Envelope
 */
public function envelope()
{
    return new Envelope(
        from: new Address('jeffrey@example.com', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}
```

Если хотите, вы также можете указать адрес `replyTo`:

```php
return new Envelope(
    from: new Address('jeffrey@example.com', 'Jeffrey Way'),
    replyTo: [
        new Address('taylor@example.com', 'Taylor Otwell'),
    ],
    subject: 'Order Shipped',
);
```

<a name="using-a-global-from-address"></a>
#### Использование глобального адреса `from`

Однако, если ваше приложение использует один и тот же адрес `from` для всех своих электронных писем, вызов метода `from` в каждом создаваемом вами классе рассылки может стать громоздким. Вместо этого вы можете указать глобальный адрес отправителя в файле конфигурации `config/mail.php`. Этот адрес будет использоваться, если в почтовом классе не указан другой адрес в методе `from`:

```php
'from' => ['address' => 'example@example.com', 'name' => 'App Name'],
```

Кроме того, вы можете определить глобальный адрес `reply_to` в конфигурационном файле `config/mail.php`:

```php
'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],
```

<a name="configuring-the-view"></a>
### Конфигурирование шаблона

В методе `content` почтового класса вы можете определить `view` или какой шаблон следует использовать при отображении содержимого электронного письма. Поскольку каждое электронное письмо обычно использует [шаблон Blade](blade.md) для визуализации своего содержимого, вы получаете всю мощь и удобство механизма шаблонов Blade при создании HTML-кода электронной письма:

```php
/**
 * Получить определение содержимого сообщения.
 *
 * @return \Illuminate\Mail\Mailables\Content
 */
public function content()
{
    return new Content(
        view: 'emails.orders.shipped',
    );
}
```

> **Примечание**\
> Вы можете создать каталог `resources/views/emails` для размещения всех ваших шаблонов электронной почты; но вы можете размещать их где угодно в каталоге `resources/views`.

<a name="plain-text-emails"></a>
#### Письма с обычным текстом

Если вы хотите определить текстовую версию вашего электронного письма, вы можете указать шаблон простого текста при создании определения содержания сообщения `Content`. Как и параметр `view`, параметр `text` должен быть именем шаблона, который будет использоваться для отображения содержимого электронного письма. Вы можете определить как HTML, так и текстовую версию вашего сообщения:

```php
/**
 * Получить определение содержимого сообщения.
 *
 * @return \Illuminate\Mail\Mailables\Content
 */
public function content()
{
    return new Content(
        view: 'emails.orders.shipped',
        text: 'emails.orders.shipped-text'
    );
}
```

Для ясности параметр `html` можно использовать как псевдоним параметра `view`:

```php
return new Content(
    html: 'emails.orders.shipped',
    text: 'emails.orders.shipped-text'
);
```

<a name="view-data"></a>
### Данные шаблона

<a name="via-public-properties"></a>
#### Передача данных шаблону через публичные свойства

Как правило, вам нужно передать в шаблон некоторые данные, которые можно использовать при отрисовки HTML-кода электронного письма. Есть два способа сделать данные доступными для вашего шаблона. Во-первых, любое публичное свойство, определенное в вашем почтовом классе, будет автоматически доступно для шаблона. Так, например, можно передать данные в конструктор почтового класса и присвоить этим данные публичным свойствам, определенным в классе:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Экземпляр заказа.
     *
     * @var \App\Models\Order
     */
    public $order;

    /**
     * Создать экземпляр нового сообщения.
     *
     * @param  \App\Models\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    /**
     * Получить определение содержимого сообщения.
     *
     * @return \Illuminate\Mail\Mailables\Content
     */
    public function content()
    {
        return new Content(
            view: 'emails.orders.shipped',
        );
    }
}
```

После того, как данные были заданы как публичные свойства, они будут автоматически доступны в вашем шаблоне, поэтому вы можете получить к ним доступ так же, как и к любым другим данным в ваших шаблонах Blade:

```blade
<div>
    Price: {{ $order->price }}
</div>
```

<a name="via-the-with-parameter"></a>
#### Передача данных шаблону через параметр `with`

Если вы хотите настроить формат данных вашего электронного письма перед их отправкой в шаблон, то вы можете вручную передать свои данные в шаблон с помощью параметра `with` определения `Content`. Как правило, вы по-прежнему будете передавать данные через конструктор почтового класса; однако вы должны установить для этих данных свойства `protected` или `private`, чтобы данные не стали автоматически доступными для шаблона:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Экземпляр заказа.
     *
     * @var \App\Models\Order
     */
    protected $order;

    /**
     * Создать экземпляр нового сообщения.
     *
     * @param  \App\Models\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    /**
     * Получить определение содержимого сообщения.
     *
     * @return \Illuminate\Mail\Mailables\Content
     */
    public function content()
    {
        return new Content(
            view: 'emails.orders.shipped',
            with: [
                'orderName' => $this->order->name,
                'orderPrice' => $this->order->price,
            ],
        );
    }
}
```

После того, как данные были переданы методу `with`, они автоматически станут доступны в вашем шаблоне, поэтому вы можете получить к ним доступ так же, как и к любым другим данным в ваших шаблонах Blade:

```blade
<div>
    Price: {{ $orderPrice }}
</div>
```

<a name="attachments"></a>
### Вложения

Чтобы добавить вложения в электронное письмо, вы должны добавить вложения в массив, возвращаемый методом `attachments` почтового отправления. Во-первых, вы можете добавить вложение, указав путь к файлу в методе `fromPath` класса `Attachment`:

```php
use Illuminate\Mail\Mailables\Attachment;

/**
 * Получить вложения к сообщению.
 *
 * @return \Illuminate\Mail\Mailables\Attachment[]
 */
public function attachments()
{
    return [
        Attachment::fromPath('/path/to/file'),
    ];
}
```

При прикреплении файлов к сообщению вы также можете указать отображаемое имя и MIME-тип для вложения, используя методы `as` и `withMime`:

```php
/**
 * Получить вложения к сообщению.
 *
 * @return \Illuminate\Mail\Mailables\Attachment[]
 */
public function attachments()
{
    return [
        Attachment::fromPath('/path/to/file')
                ->as('name.pdf')
                ->withMime('application/pdf'),
    ];
}
```

<a name="attaching-files-from-disk"></a>
#### Прикрепление файлов с диска

Если вы сохранили файл на одном из [дисков файлового хранилища](filesystem.md), то вы можете прикрепить его к электронному письму с помощью метода `fromStorage` класса `Attachment`:

```php
/**
 * Получить вложения к сообщению.
 *
 * @return \Illuminate\Mail\Mailables\Attachment[]
 */
public function attachments()
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

Конечно, вы также можете указать имя вложения и MIME-тип:

```php
/**
 * Получить вложения к сообщению.
 *
 * @return \Illuminate\Mail\Mailables\Attachment[]
 */
public function attachments()
{
    return [
        Attachment::fromStorage('/path/to/file')
                ->as('name.pdf')
                ->withMime('application/pdf'),
    ];
}
```

Метод `fromStorage` используется, если вам нужно указать диск хранения, отличный от вашего диска по умолчанию:

```php
/**
 * Получить вложения к сообщению.
 *
 * @return \Illuminate\Mail\Mailables\Attachment[]
 */
public function attachments()
{
    return [
        Attachment::fromStorageDisk('s3', '/path/to/file')
                ->as('name.pdf')
                ->withMime('application/pdf'),
    ];
}
```

<a name="raw-data-attachments"></a>
#### Вложения необработанных данных

Метод `fromData` класса `Attachment` используется для прикрепления необработанной строки в качестве вложения. Например, вы можете использовать этот метод, если вы создали PDF-файл в памяти и хотите прикрепить его к электронному письму, не записывая его на диск. Метод `fromData` принимает замыкание, которое определяет байты необработанных данных, а также имя, которое должно быть присвоено вложению:

```php
/**
 * Получить вложения к сообщению.
 *
 * @return \Illuminate\Mail\Mailables\Attachment[]
 */
public function attachments()
{
    return [
        Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
                ->withMime('application/pdf'),
    ];
}
```

<a name="inline-attachments"></a>
### Встраиваемые вложения

Встраивание изображений в ваши электронные письма, как правило, обременительно; однако Laravel предлагает удобный способ прикреплять изображения к вашим письмам. Чтобы встроить изображение, используйте метод `embed` для переменной `$message` в вашем шаблоне электронной почты. Laravel автоматически делает переменную `$message` доступной для всех ваших шаблонов электронной почты, поэтому вам не нужно беспокоиться о ее передаче вручную:

```blade
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

> **Предупреждение**\
> Переменная `$message` недоступна в шаблонах текстовых сообщений, так как в текстовых сообщениях не используются встраиваемые вложения.

<a name="embedding-raw-data-attachments"></a>
#### Встраиваемые вложения необработанных данных

Если у вас уже есть строка необработанных данных изображения, которую вы хотите встроить в шаблон электронной почты, то вы можете вызвать метод `embedData` для переменной `$message`. При вызове метода `embedData` вам необходимо указать имя файла, которое должно быть присвоено встраиваемому изображению:

```blade
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

<a name="attachable-objects"></a>
### Прикрепляемые объекты

Хотя присоединения файлов к сообщениям с помощью простых строковых путей часто бывает достаточно, во многих случаях присоединяемые сущности в вашем приложении представлены классами. Например, если ваше приложение прикрепляет фотографию к сообщению, то ваше приложение также может иметь модель `Photo`, представляющую эту фотографию. Не было бы удобно просто передать модель `Photo` методу `attach`? Присоединяемые объекты позволяют сделать именно это.

Для начала реализуйте интерфейс `Illuminate\Contracts\Mail\Attachable` для объекта, который будет прикрепляться к сообщениям. Этот интерфейс требует, чтобы ваш класс определял метод `toMailAttachment`, который возвращает экземпляр `Illuminate\Mail\Attachment`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Mail\Attachable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Mail\Attachment;

class Photo extends Model implements Attachable
{
    /**
     * Получить представление модели для прикрепления к письму.
     *
     * @return \Illuminate\Mail\Attachment
     */
    public function toMailAttachment()
    {
        return Attachment::fromPath('/path/to/file');
    }
}
```

После того, как вы определили свой присоединяемый объект, вы можете вернуть экземпляр этого объекта из метода `attachments` при создании сообщения электронного сообщения:

```php
/**
 * Получить вложения к сообщению.
 *
 * @return array
 */
public function attachments()
{
    return [$this->photo];
}
```

Конечно, данные вложений могут храниться в удаленном файловом хранилище, таком как Amazon S3. Laravel также позволяет создавать экземпляры вложений из данных, которые хранятся на одном из [дисков файловой системы](filesystem.md) вашего приложения:

```php
// Создать вложение из файла на диске по умолчанию ...
return Attachment::fromStorage($this->path);

// Создать вложение из файла на указанном диске...
return Attachment::fromStorageDisk('backblaze', $this->path);
```

Кроме того, вы можете создавать экземпляры вложений с помощью данных, которые хранятся в памяти. Для этого передайте замыкание методу `fromData`. Замыкание должно возвращать необработанные данные, представляющие вложение:

```php
return Attachment::fromData(fn () => $this->content, 'Photo Name');
```

Laravel также предлагает дополнительные методы, которые вы можете использовать для настройки вложений. Например, вы можете использовать методы `as` и `withMime` для указания имени файла и MIME-типа:

```php
return Attachment::fromPath('/path/to/file')
        ->as('Photo Name')
        ->withMime('image/jpeg');
```

<a name="headers"></a>
### Заголовки

Иногда требуется добавить к исходящему письму дополнительные заголовки. Например, вам может потребоваться установить собственный `Message-Id` или другие произвольные текстовые заголовки.

Для этого определите метод `headers` для вашего почтового отправления. Метод `headers` должен возвращать экземпляр `Illuminate\Mail\Mailables\Headers`. Этот класс принимает параметры `messageId`, `references` и `text`. Конечно, вы можете указать только те параметры, которые вам нужны для вашего конкретного письма:

```php
use Illuminate\Mail\Mailables\Headers;

/**
 * Получить заголовки отправления.
 *
 * @return \Illuminate\Mail\Mailables\Headers
 */
public function headers()
{
    return new Headers(
        messageId: 'custom-message-id@example.com',
        references: ['previous-message@example.com'],
        text: [
            'X-Custom-Header' => 'Custom Value',
        ],
    );
}
```

<a name="tags-and-metadata"></a>
### Теги и метаданные

Некоторые сторонние почтовые поставщики, такие как Mailgun и Postmark, поддерживают «теги» и «метаданные» сообщения, которые могут использоваться для группировки и отслеживания электронных писем, отправляемых вашим приложением. Вы можете добавить теги и метаданные к сообщению электронной почты с помощью вашего определения `Envelope`:

```php
use Illuminate\Mail\Mailables\Envelope;

/**
 * Получить конверт сообщения.
 *
 * @return \Illuminate\Mail\Mailables\Envelope
 */
public function envelope()
{
    return new Envelope(
        subject: 'Order Shipped',
        tags: ['shipment'],
        metadata: [
            'order_id' => $this->order->id,
        ],
    );
}
```

Если ваше приложение использует драйвер Mailgun, то вы можете обратиться к документации Mailgun для получения дополнительной информации о [тегах](https://documentation.mailgun.com/en/latest/user_manual.html#tagging-1) и [метаданных](https://documentation.mailgun.com/en/latest/user_manual.html#attaching-data-to-messages). Кроме того, можно также обратиться к документации Postmark для получения дополнительной информации об их поддержке [тегов](https://postmarkapp.com/blog/tags-support-for-smtp) и [метаданных](https://postmarkapp.com/support/article/1125-custom-metadata-faq).

Если ваше приложение использует Amazon SES для отправки электронных писем, то вам следует использовать метод `metadata` для прикрепления [тегов SES](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) к сообщению.

<a name="customizing-the-symfony-message"></a>
### Настройка сообщения Symfony

Почтовые возможности Laravel основаны на Symfony Mailer. Laravel позволяет вам регистрировать замыкания, которые будут вызываться экземпляром Symfony Message перед отправкой сообщения. Это дает вам возможность глубоко настроить сообщение перед его отправкой. Для этого определите параметр `using` в вашем определении `Envelope`:

```php
use Illuminate\Mail\Mailables\Envelope;
use Symfony\Component\Mime\Email;

/**
 * Получить конверт сообщения.
 *
 * @return \Illuminate\Mail\Mailables\Envelope
 */
public function envelope()
{
    return new Envelope(
        subject: 'Order Shipped',
        using: [
            function (Email $message) {
                // ...
            },
        ]
    );
}
```

<a name="markdown-mailables"></a>
## Отправления с разметкой Markdown

Почтовые сообщения с разметкой Markdown позволяют вам воспользоваться преимуществами предварительно созданных шаблонов и компонентов [почтовых уведомлений](notifications.md#mail-notifications) в ваших почтовых рассылках. Поскольку сообщения написаны на Markdown, Laravel может отображать красивые, отзывчивые HTML-шаблоны для сообщений, а также автоматически генерировать аналог в виде простого текста.

<a name="generating-markdown-mailables"></a>
### Генерация отправлений с разметкой Markdown

Чтобы сгенерировать почтовый класс с соответствующим шаблоном Markdown, вы можете использовать параметр `--markdown` в команде `make:mail` Artisan:

```shell
php artisan make:mail OrderShipped --markdown=emails.orders.shipped
```

Затем, в методе `content` при задании определения `Content` используйте параметр `markdown` вместо параметра `view`:

```php
use Illuminate\Mail\Mailables\Content;

/**
 * Получить определение содержимого сообщения.
 *
 * @return \Illuminate\Mail\Mailables\Content
 */
public function content()
{
    return new Content(
        markdown: 'emails.orders.shipped',
        with: [
            'url' => $this->orderUrl,
        ],
    );
}
```

<a name="writing-markdown-messages"></a>
### Написание сообщений с разметкой Markdown

Почтовые сообщения Markdown используют комбинацию компонентов Blade и синтаксиса Markdown, которые позволяют легко создавать почтовые сообщения, используя предварительно созданные компоненты пользовательского интерфейса электронной почты Laravel:

```blade
<x-mail::message>
# Order Shipped

Your order has been shipped!

<x-mail::button :url="$url">
View Order
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

> **Примечание**\
> Не используйте лишние отступы при написании писем Markdown. По стандартам Markdown парсеры будут отображать контент с отступом в виде блоков кода.

<a name="button-component"></a>
#### Компонент Button

Компонент кнопки отображает ссылку на кнопку по центру. Компонент принимает два аргумента: `url` и необязательный `color`. Поддерживаемые цвета: `primary`, `success` и `error`. Вы можете добавить к сообщению столько компонентов кнопки, сколько захотите:

```blade
<x-mail::button :url="$url" color="success">
View Order
</x-mail::button>
```

<a name="panel-component"></a>
#### Компонент Panel

Компонент панели отображает указанный блок текста на панели, цвет фона которой немного отличается от цвета остальной части сообщения. Это позволяет привлечь внимание к указанному блоку текста:

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>
#### Компонент Table

Компонент таблицы позволяет преобразовать таблицу Markdown в таблицу HTML. Компонент принимает в качестве содержимого таблицу Markdown. Выравнивание столбцов таблицы поддерживается с использованием синтаксиса выравнивания таблицы Markdown по умолчанию:

```blade
<x-mail::table>
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### Изменение компонентов

Вы можете экспортировать все почтовые компоненты Markdown в собственное приложение для их дальнейшего изменения. Чтобы экспортировать компоненты, используйте команду `vendor:publish` Artisan с параметром `--tag=laravel-mail`:

```shell
php artisan vendor:publish --tag=laravel-mail
```

Эта команда опубликует почтовые компоненты Markdown в каталоге `resources/views/vendor/mail`. Каталог `mail` будет содержать каталог `html` и `text`, каждый из которых содержит соответствующие представления каждого доступного компонента. Вы можете настроить эти компоненты по своему усмотрению.

<a name="customizing-the-css"></a>
#### Редактирование файла CSS

После экспорта компонентов в каталоге `resources/views/vendor/mail/html/themes` будет содержаться файл `default.css`. Вы можете отредактировать CSS в этом файле, и ваши стили будут автоматически преобразованы во встроенные стили CSS в HTML-представлениях ваших почтовых сообщений Markdown.

Если вы хотите создать совершенно новую тему для компонентов Laravel Markdown, вы можете поместить файл CSS в каталог `html/themes`. После присвоения имени и сохранения файла CSS обновите параметр `theme` в файле конфигурации вашего приложения `config/mail.php`, чтобы он соответствовал имени вашей новой темы.

Чтобы настроить тему для отдельного почтового сообщения, вы можете установить в свойстве `$theme` почтового класса имя темы, которое следует использовать при отправке этого почтового сообщения.

<a name="sending-mail"></a>
## Отправка почты

Чтобы отправить сообщение, используйте метод `to` [фасада](facades.md) `Mail`. Метод `to` принимает адрес электронной почты, экземпляр пользователя или коллекцию пользователей. Если вы передаете объект или коллекцию объектов, почтовая программа будет автоматически использовать их свойства `email` и `name` при определении получателей электронной почты, поэтому убедитесь, что эти атрибуты доступны для ваших объектов. После того, как вы указали своих получателей, вы можете передать экземпляр вашего почтового класса методу `send`:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Mail\OrderShipped;
use App\Models\Order;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderShipmentController extends Controller
{
    /**
     * Отправить заказ.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $order = Order::findOrFail($request->order_id);

        // Отправляем заказ ...

        Mail::to($request->user())->send(new OrderShipped($order));
    }
}
```

Вы не ограничены простым указанием получателей при отправке сообщения. Вы можете указать получателей to, `cc` и `bcc`, связав их соответствующие методы вместе:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

<a name="looping-over-recipients"></a>
#### Итерация списка получателей

Иногда требуется отправить почтовое сообщение списку получателей, перебирая массив получателей / адресов электронной почты. Однако, поскольку метод `to` добавляет адреса электронной почты к списку получателей почтового сообщения, каждая итерация цикла будет отправлять другое электронное письмо каждому предыдущему получателю. Следовательно, вы всегда должны повторно создавать почтовый экземпляр для каждого получателя:

```php
foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
    Mail::to($recipient)->send(new OrderShipped($order));
}
```

<a name="sending-mail-via-a-specific-mailer"></a>
#### Указание драйвера при отправки почты

По умолчанию Laravel будет отправлять электронную почту, используя почтовую программу, настроенную как почтовую программу `default` в файле конфигурации вашего приложения `mail`. Однако вы можете использовать метод `mailer` для отправки сообщения с использованием определенной конфигурации почтовой программы:

```php
Mail::mailer('postmark')
        ->to($request->user())
        ->send(new OrderShipped($order));
```

<a name="queueing-mail"></a>
### Очередь почты

<a name="queueing-a-mail-message"></a>
#### Постановка сообщения в очередь почты

Поскольку отправка сообщений электронной почты может негативно повлиять на время отклика вашего приложения, многие разработчики ставят сообщения электронной почты в очередь для фоновой отправки. Laravel упрощает это с помощью встроенного [API унифицированной очереди](queues.md). Чтобы поставить почтовое сообщение в очередь, используйте метод `queue` фасада `Mail` после указания получателей сообщения:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```

Этот метод автоматически помещает задание в очередь, чтобы сообщение отправлялось в фоновом режиме. Перед использованием этого функционала вам необходимо [настроить очереди](queues.md).

<a name="delayed-message-queueing"></a>
#### Очередь отложенных сообщений

Если вы хотите отложить доставку электронного сообщения в очереди, вы можете использовать метод `later`. В качестве первого аргумента метод `later` принимает экземпляр `DateTime`, указывающий, когда сообщение должно быть отправлено:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later(now()->addMinutes(10), new OrderShipped($order));
```

<a name="pushing-to-specific-queues"></a>
#### Постановка сообщения в конкретную очередь почты

Поскольку все почтовые классы, сгенерированные с помощью команды `make:mail`, используют трейт `Illuminate\Bus\Queueable`, вы можете вызвать методы `onQueue` и `onConnection` для любого экземпляра почтового класса, что позволит вам указать соединение и имя очереди для сообщения:

```php
$message = (new OrderShipped($order))
                ->onConnection('sqs')
                ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

<a name="queueing-by-default"></a>
#### Очередь почты, используемая по умолчанию

Если у вас есть почтовые классы, которые вы хотите всегда ставить в очередь, то вы можете реализовать контракт `ShouldQueue` для этого класса. Теперь, даже если вы вызовете метод `send` для отправки, почтовый класс все равно будет помещен в очередь, поскольку он содержит контракт:

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    //
}
```

<a name="queued-mailables-and-database-transactions"></a>
#### Почтовые сообщения в очереди и транзакции в базе данных

Когда помещенные в очередь почтовые сообщения отправляются в рамках транзакций базы данных, они могут быть обработаны очередью до того, как транзакция базы данных будет зафиксирована. Когда это происходит, любые обновления, внесенные вами в модели или записи базы данных во время транзакции базы данных, могут еще не быть отражены в базе данных. Кроме того, любые модели или записи базы данных, созданные в рамках транзакции, могут не существовать в базе данных. Если ваше почтовое сообщение зависит от этих моделей, при обработке задания, отправляющего почтовое сообщение в очереди, могут возникнуть непредвиденные ошибки.

Если для параметра `after_commit` конфигурации вашего соединения с очередью задано значение `false`, то вы все равно можете указать, что конкретное почтовое сообщение в очереди должно быть отправлено после того, как все открытые транзакции базы данных были зафиксированы, вызвав метод `afterCommit` при отправке почтового сообщения:

```php
Mail::to($request->user())->send(
    (new OrderShipped($order))->afterCommit()
);
```

В качестве альтернативы вы можете вызвать метод `afterCommit` из конструктора вашего почтового сообщения:

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    /**
     * Создать новый экземпляр сообщения.
     *
     * @return void
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> **Примечание**\
> Чтобы узнать больше о том, как обойти эти проблемы, просмотрите документацию, касающуюся [заданий в очереди и транзакций базы данных](queues.md#jobs-and-database-transactions).

<a name="rendering-mailables"></a>
## Отрисовка отправлений

Иногда требуется получить HTML-содержимое почтового сообщения, не отправляя его. Для этого вы можете вызвать метод `render` почтового сообщения. Этот метод вернет проанализированное HTML-содержимое почтового сообщения в виде строки:

```php
use App\Mail\InvoicePaid;
use App\Models\Invoice;

$invoice = Invoice::find(1);

return (new InvoicePaid($invoice))->render();
```

<a name="previewing-mailables-in-the-browser"></a>
### Предварительный просмотр отправлений в браузере

При разработке шаблона почтового сообщения удобно быстро просмотреть визуализированное почтовое сообщение в браузере, как типичный шаблон Blade. По этой причине Laravel позволяет вам возвращать любое почтовое сообщение непосредственно из замыкания маршрута или контроллера. Когда почтовое сообщение возвращается, оно будет обработано и отображено в браузере, что позволит вам быстро просмотреть его дизайн без необходимости отправлять его на реальный адрес электронной почты:

```php
Route::get('/mailable', function () {
    $invoice = App\Models\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```

> **Предупреждение**\
> [Встраиваемые вложения](#inline-attachments) не будут отображаться при предварительном просмотре почтового сообщения в вашем браузере. Чтобы просмотреть эти почтовые сообщения, вы должны отправить их в приложение для тестирования электронной почты, например, [MailHog](https://github.com/mailhog/MailHog) или [HELO](https://usehelo.com).

<a name="localizing-mailables"></a>
## Локализация отправлений

Laravel позволяет отправлять почтовые сообщения, используя язык, отличный от текущего языка запроса, и даже будет помнить его, если почта находится в очереди.

Для этого фасад `Mail` содержит метод `locale` для установки желаемого языка. Приложение изменит язык при анализе шаблона почтового сообщения, а затем вернется к предыдущему языку, когда анализ будет завершен:

```php
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);
```

<a name="user-preferred-locales"></a>
### Предпочитаемые пользователем локализации

Иногда приложения хранят предпочтительный язык каждого пользователя. Реализуя контракт `HasLocalePreference` в ваших моделях, вы можете указать Laravel использовать этот сохраненный язык при отправке почты:

```php
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    /**
     * Получить предпочитаемую пользователем локализацию.
     *
     * @return string
     */
    public function preferredLocale()
    {
        return $this->locale;
    }
}
```

После того, как вы реализовали интерфейс, Laravel будет автоматически использовать предпочтительный язык при отправке уведомлений и почтовых сообщений модели. Следовательно, при использовании этого интерфейса нет необходимости вызывать метод `locale`:

```php
Mail::to($request->user())->send(new OrderShipped($order));
```

<a name="testing-mailables"></a>
## Тестирование отправлений

Laravel предлагает множество методов для проверки структуры вашего почтового отправления. Кроме того, Laravel предлагает несколько удобных методов для проверки того, содержит ли ваше почтовое отправление ожидаемый контент. Этими методами являются: `assertSeeInHtml`, `assertDontSeeInHtml`, `assertSeeInOrderInHtml`, `assertSeeInText`, `assertDontSeeInText`, `assertSeeInOrderInText`, `assertHasAttachment`, `assertHasAttachedData`, `assertHasAttachmentFromStorage` и `assertHasAttachmentFromStorageDisk`.

Как и следовало ожидать, «HTML утверждения» необходимы для проверки того, что HTML-версия вашего почтового отправления содержит переданную строку, в то время как «текстовые утверждения» служат для проверки того, что текстовая версия вашего почтового отправления содержит переданную строку:

```php
use App\Mail\InvoicePaid;
use App\Models\User;

public function test_mailable_content()
{
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertFrom('jeffrey@example.com');
    $mailable->assertTo('taylor@example.com');
    $mailable->assertHasCc('abigail@example.com');
    $mailable->assertHasBcc('victoria@example.com');
    $mailable->assertHasReplyTo('tyler@example.com');
    $mailable->assertHasSubject('Invoice Paid');
    $mailable->assertHasTag('example-tag');
    $mailable->assertHasMetadata('key', 'value');

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertSeeInHtml('Invoice Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
}
```

<a name="testing-mailable-sending"></a>
#### Тестирование отправки почтовых сообщений

Мы предлагаем тестировать содержимое ваших почтовых сообщений отдельно от тестов, которые подтверждают, что данное почтовое сообщение было «отправлено» определенному пользователю. Чтобы узнать, как протестировать, что почтовые отправления были отправлены, ознакомьтесь с нашей документацией по [фальсификации Mail](mocking.md#mail-fake).

<a name="mail-and-local-development"></a>
## Почта и локальная разработка

При разработке приложения для отправки электронной почты вы, вероятно, не захотите отправлять электронные письма на реальные адреса электронной почты. Laravel предлагает несколько способов «отключить» фактическую отправку электронных писем во время локальной разработки.

<a name="log-driver"></a>
#### Драйвер Log

Вместо того, чтобы отправлять ваши электронные письма, почтовый драйвер `log` будет записывать все сообщения электронной почты в ваши файлы журналов для проверки. Обычно этот драйвер используется только во время локальной разработки. Для получения дополнительной информации о настройке вашего приложения для каждой среды ознакомьтесь с [документацией по конфигурации](configuration.md#environment-configuration).

<a name="mailtrap"></a>
#### HELO / Mailtrap / MailHog

Альтернативно, вы можете использовать такую службу, как [HELO](https://usehelo.com) или [Mailtrap](https://mailtrap.io) и драйвер `smtp`, чтобы отправлять сообщения электронной почты в «фиктивный» почтовый ящик, где вы можете просмотреть их в настоящем почтовом клиенте. Этот подход имеет то преимущество, что позволяет вам фактически проверять окончательные электронные письма, непосредственно в почтовых службах.

Если вы используете [Laravel Sail](sail.md), то вы можете предварительно просмотреть свои сообщения с помощью [MailHog](https://github.com/mailhog/MailHog). Когда Sail запущен, вы можете получить доступ к интерфейсу MailHog по адресу: `http://localhost:8025`.

<a name="using-a-global-to-address"></a>
#### Использование единого адреса получателя

Наконец, вы можете указать глобальный адрес «получателя», используя метод `alwaysTo` фасада `Mail`. Как правило, вызов этого метода осуществляется в методе `boot` [поставщика служб](providers.md):

```php
use Illuminate\Support\Facades\Mail;

/**
 * Загрузка любых служб приложения.
 *
 * @return void
 */
public function boot()
{
    if ($this->app->environment('local')) {
        Mail::alwaysTo('taylor@example.com');
    }
}
```

<a name="events"></a>
## События

Laravel запускает два события в процессе отправки почтовых сообщений. Событие `MessageSending` запускается перед отправкой сообщения, а событие `MessageSent` запускается после того, как сообщение было отправлено. Помните, что эти события запускаются, когда почта *отправляется*, а не когда она ставится в очередь. Как правило, регистрация слушателей этих событий осуществляется в поставщике `App\Providers\EventServiceProvider`:

```php
use App\Listeners\LogSendingMessage;
use App\Listeners\LogSentMessage;
use Illuminate\Mail\Events\MessageSending;
use Illuminate\Mail\Events\MessageSent;

/**
 * Карта слушателей событий приложения.
 *
 * @var array
 */
protected $listen = [
    MessageSending::class => [
        LogSendingMessage::class,
    ],

    MessageSent::class => [
        LogSentMessage::class,
    ],
];
```

<a name="custom-transports"></a>
## Пользовательские драйвера

Laravel включает множество почтовых транспортов; однако вы можете написать свои собственные транспорты для доставки электронной почты через другие сервисы, которые Laravel не поддерживает по умолчанию. Для начала определите класс, который расширяет класс `Symfony\Component\Mailer\Transport\AbstractTransport`. Затем реализуйте методы `doSend` и `__toString()` в вашем классе транспорта:

```php
use MailchimpTransactional\ApiClient;
use Symfony\Component\Mailer\SentMessage;
use Symfony\Component\Mailer\Transport\AbstractTransport;
use Symfony\Component\Mime\MessageConverter;

class MailchimpTransport extends AbstractTransport
{
    /**
     * Клиент Mailchimp API.
     *
     * @var \MailchimpTransactional\ApiClient
     */
    protected $client;

    /**
     * Создать новый экземпляр транспорта Mailchimp.
     *
     * @param  \MailchimpTransactional\ApiClient  $client
     * @return void
     */
    public function __construct(ApiClient $client)
    {
        parent::__construct();

        $this->client = $client;
    }

    /**
     * {@inheritDoc}
     */
    protected function doSend(SentMessage $message): void
    {
        $email = MessageConverter::toEmail($message->getOriginalMessage());

        $this->client->messages->send(['message' => [
            'from_email' => $email->getFrom(),
            'to' => collect($email->getTo())->map(function ($email) {
                return ['email' => $email->getAddress(), 'type' => 'to'];
            })->all(),
            'subject' => $email->getSubject(),
            'text' => $email->getTextBody(),
        ]]);
    }

    /**
     * Получить строковое представление транспорта.
     *
     * @return string
     */
    public function __toString(): string
    {
        return 'mailchimp';
    }
}
```

После того, как вы определили свой собственный транспорт, вы можете зарегистрировать его с помощью метода `extend` фасада `Mail`. Как правило, это должно быть сделано в методе `boot` поставщика `AppServiceProvider` вашего приложения. Аргумент `$config` будет передан замыканию метода `extend`. Этот аргумент будет содержать массив конфигурации, определенный для почтовой программы в конфигурационном файле `config/mail.php` приложения:

```php
use App\Mail\MailchimpTransport;
use Illuminate\Support\Facades\Mail;

/**
 * Загрузка любых служб приложения.
 *
 * @return void
 */
public function boot()
{
    Mail::extend('mailchimp', function (array $config = []) {
        return new MailchimpTransport(/* ... */);
    })
}
```

После того, как ваш пользовательский транспорт был определен и зарегистрирован, для использования нового транспорта вы можете создать определение почтовой программы в конфигурационном файле `config/mail.php` вашего приложения:

```php
'mailchimp' => [
    'transport' => 'mailchimp',
    // ...
],
```

<a name="additional-symfony-transports"></a>
### Дополнительные драйвера Symfony

Laravel включает поддержку некоторых существующих почтовых транспортов, поддерживаемых Symfony, таких как Mailgun и Postmark. Однако вы можете захотеть расширить Laravel поддержкой дополнительных транспортов, поддерживаемых Symfony. Вы можете сделать это, потребовав необходимую почтовую программу Symfony через Composer и зарегистрировав транспорт в Laravel. Например, вы можете установить и зарегистрировать почтовую программу Symfony Sendinblue:

```none
composer require symfony/sendinblue-mailer
```

После установки пакета Sendinblue вы можете добавить запись для своих учетных данных в конфигурационном файле `services` вашего приложения:

```php
'sendinblue' => [
    'key' => 'your-api-key',
],
```

Наконец, вы можете использовать метод `extend` фасада `Mail` для регистрации транспорта в Laravel. Как правило, это должно быть сделано в методе `boot` поставщика `AppServiceProvider` вашего приложения:

```php
use Illuminate\Support\Facades\Mail;
use Symfony\Component\Mailer\Bridge\Sendinblue\Transport\SendinblueTransportFactory;
use Symfony\Component\Mailer\Transport\Dsn;

/**
 * Загрузка любых служб приложения.
 *
 * @return void
 */
public function boot()
{
    Mail::extend('sendinblue', function () {
        return (new SendinblueTransportFactory)->create(
            new Dsn(
                'sendinblue+api',
                'default',
                config('services.sendinblue.key')
            )
        );
    });
}
```
