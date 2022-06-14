# Laravel 9 · Уведомления

- [Введение](#introduction)
- [Генерация уведомлений](#generating-notifications)
- [Отправка уведомлений](#sending-notifications)
    - [Использование трейта `Notifiable`](#using-the-notifiable-trait)
    - [Использование фасада `Notification`](#using-the-notification-facade)
    - [Определение каналов доставки](#specifying-delivery-channels)
    - [Очереди уведомлений](#queueing-notifications)
    - [Уведомления по запросу](#on-demand-notifications)
- [Почтовые уведомления](#mail-notifications)
    - [Формирование почтовых сообщений](#formatting-mail-messages)
    - [Изменение отправителя](#customizing-the-sender)
    - [Изменение получателя](#customizing-the-recipient)
    - [Изменение темы сообщения](#customizing-the-subject)
    - [Изменение почтового драйвера](#customizing-the-mailer)
    - [Изменение почтовых шаблонов](#customizing-the-templates)
    - [Почтовые вложения](#mail-attachments)
    - [Теги и метаданные](#adding-tags-metadata)
    - [Настройка сообщения Symfony](#customizing-the-symfony-message)
    - [Использование почтовых отправлений](#using-mailables)
    - [Предварительный просмотр почтовых уведомлений](#previewing-mail-notifications)
- [Почтовые уведомления с разметкой Markdown](#markdown-mail-notifications)
    - [Генерация сообщения](#generating-the-message)
    - [Написание сообщения](#writing-the-message)
    - [Изменение компонентов](#customizing-the-components)
- [Уведомления через канал `database`](#database-notifications)
    - [Предварительная подготовка канала `database`](#database-prerequisites)
    - [Формирование уведомлений канала `database`](#formatting-database-notifications)
    - [Доступ к уведомлениям](#accessing-the-notifications)
    - [Отметка прочитанных уведомлений](#marking-notifications-as-read)
- [Трансляция уведомлений](#broadcast-notifications)
    - [Предварительная подготовка канала `broadcast`](#broadcast-prerequisites)
    - [Формирование транслируемых уведомлений](#formatting-broadcast-notifications)
    - [Прослушивание транслируемых уведомлений](#listening-for-notifications)
- [Уведомления через SMS](#sms-notifications)
    - [Предварительная подготовка канала SMS](#sms-prerequisites)
    - [Формирование уведомлений через SMS](#formatting-sms-notifications)
    - [Использование шорткодов при формировании SMS-уведомлений](#formatting-shortcode-notifications)
    - [Изменение номера отправителя](#customizing-the-from-number)
    - [Добавление ссылки на клиента](#adding-a-client-reference)
    - [Маршрутизация SMS-уведомлений](#routing-sms-notifications)
- [Уведомления через Slack](#slack-notifications)
    - [Предварительная подготовка канала Slack](#slack-prerequisites)
    - [Формирование уведомления через Slack](#formatting-slack-notifications)
    - [Вложения Slack-уведомлений](#slack-attachments)
    - [Маршрутизация Slack-уведомлений](#routing-slack-notifications)
- [Локализация уведомлений](#localizing-notifications)
- [События уведомления](#notification-events)
- [Пользовательские каналы уведомлений](#custom-channels)

<a name="introduction"></a>
## Введение

В дополнение к поддержке [отправки электронной почты](mail.md), Laravel обеспечивает поддержку отправки уведомлений по различным каналам доставки, включая электронную почту, SMS (через [Vonage](https://www.vonage.com/communications-apis/), бывший Nexmo) и [Slack](https://slack.com). Кроме того, сообществом было создано множество [каналов уведомлений](https://laravel-notification-channels.com/about/#suggesting-a-new-channel) для отправки уведомлений по десяткам различных каналов! Уведомления также могут храниться в базе данных, поэтому они могут быть отображены в вашем веб-интерфейсе.

Как правило, уведомления должны быть короткими информационными сообщениями, которые уведомляют пользователей о том, что произошло в вашем приложении. Например, если вы пишете приложение для выставления счетов, то вы можете отправить своим пользователям уведомление «Счет оплачен» по каналам электронной почты и SMS.

<a name="generating-notifications"></a>
## Генерация уведомлений

В Laravel каждое уведомление представлено единым классом. Чтобы сгенерировать новое уведомление, используйте команду `make:notification` [Artisan](artisan.md). Эта команда поместит новый класс уведомления в каталог `app/Notifications` вашего приложения. Если этот каталог не существует в вашем приложении, то Laravel предварительно создаст его:

```shell
php artisan make:notification InvoicePaid
```

Каждый класс уведомления содержит метод `via` и переменное количество методов формирования сообщений, таких как `toMail` или `toDatabase`, которые преобразуют уведомление в сообщение, адаптированное для этого конкретного канала.

<a name="sending-notifications"></a>
## Отправка уведомлений

<a name="using-the-notifiable-trait"></a>
### Использование трейта `Notifiable`

Уведомления могут быть отправлены двумя способами: с использованием метода `notify` трейта `Notifiable` или с помощью [фасада](facades.md) `Notification`. Трейт `Notifiable` по умолчанию содержится в модели `App\Models\User` вашего приложения:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Метод `notify`, предоставляемый этим трейтом, ожидает получить экземпляр уведомления:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} Помните, что вы можете использовать трейт `Notifiable` в любой из ваших моделей. Вы не ограничены использованием его только в модели `User`.

<a name="using-the-notification-facade"></a>
### Использование фасада `Notification`

Как вариант, вы можете отправлять уведомления через [фасад](facades.md) `Notification`. Этот подход полезен при отправки уведомления нескольким уведомляемым объектам, например группе пользователей. Чтобы отправлять уведомления с помощью фасада, передайте все уведомляемые сущности и экземпляр уведомления методу `send`:

    use Illuminate\Support\Facades\Notification;

    Notification::send($users, new InvoicePaid($invoice));

Вы также можете немедленно отправлять уведомления, используя метод `sendNow`. Этот метод немедленно отправит уведомление, даже если оно реализует интерфейс `ShouldQueue`:

    Notification::sendNow($developers, new DeploymentCompleted($deployment));

<a name="specifying-delivery-channels"></a>
### Определение каналов доставки

Каждый класс уведомлений имеет метод `via`, который определяет, по каким каналам будет доставлено уведомление. Уведомления можно отправлять по каналам `mail`, `database`, `broadcast`, `vonage` и `slack`.

> {tip} Если вы хотите использовать другие каналы доставки, такие как Telegram или Pusher, то посетите веб-сайт сообщества [Laravel Notification Channels](http://laravel-notification-channels.com).

Метод `via` получает экземпляр `$notifiable`, который будет экземпляром класса, которому отправляется уведомление. Вы можете использовать `$notifiable`, чтобы определить, по каким каналам должно доставляться уведомление:

    /**
     * Получить каналы доставки уведомлений.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### Очереди уведомлений

> {note} Перед отправкой уведомлений в очередь вы должны настроить и запустить [обработчик очереди](queues.md).

Отправка уведомлений может занять время, особенно если каналу необходимо выполнить внешний вызов API для доставки уведомления. Чтобы ускорить время отклика вашего приложения, поместите ваше уведомление в очередь, добавив интерфейс `ShouldQueue` и трейт `Queueable` в ваш класс. Интерфейс и трейт уже импортированы для всех уведомлений, сгенерированных с помощью команды `make:notification`, поэтому вы можете сразу добавить их в свой класс уведомлений:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

После добавления интерфейса `ShouldQueue` к уведомлению вы можете отправить уведомление как обычно. Laravel обнаружит интерфейс `ShouldQueue` в классе и автоматически поставит в очередь доставку уведомления:

    $user->notify(new InvoicePaid($invoice));

При постановке уведомлений в очередь будет создано задание в очереди для каждой комбинации получателя и канала. Например, в очередь будут отправлены шесть заданий, если у вашего уведомления три получателя и два канала.

<a name="delaying-notifications"></a>
#### Задержка уведомлений

Если вы хотите отложить доставку уведомления, то вы можете вызвать метод `delay` экземпляра уведомления:

    $delay = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($delay));

<a name="delaying-notifications-per-channel"></a>
#### Задержка уведомлений для конкретного канала

Вы можете передать массив методу `delay`, чтобы указать величину задержки для определенных каналов:

    $user->notify((new InvoicePaid($invoice))->delay([
        'mail' => now()->addMinutes(5),
        'sms' => now()->addMinutes(10),
    ]));

В качестве альтернативы вы можете определить метод `withDelay` в самом классе уведомлений. Метод `withDelay` должен возвращать массив имен каналов и значений задержки:

    /**
     * Определить задержку доставки уведомлений.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function withDelay($notifiable)
    {
        return [
            'mail' => now()->addMinutes(5),
            'sms' => now()->addMinutes(10),
        ];
    }

<a name="customizing-the-notification-queue-connection"></a>
#### Изменение соединения отложенных уведомлений

По умолчанию отложенные уведомления будут помещены в очередь с использованием соединения очереди по умолчанию для вашего приложения. Если вы хотите указать другое соединение, которое должно использоваться для определенного уведомления, то вы можете определить свойство `$connection` в классе уведомления:

    /**
     * Имя соединения очереди, которое будет использоваться отложенным уведомлением.
     *
     * @var string
     */
    public $connection = 'redis';

<a name="customizing-notification-channel-queues"></a>
#### Изменение очереди канала уведомлений

Если вы хотите указать конкретную очередь, которая должна использоваться для каждого канала уведомления, поддерживаемого уведомлением, то вы можете определить метод `viaQueues` в своем уведомлении. Этот метод должен возвращать массив пар имя канала / имя очереди:

    /**
     * Определить, какие очереди следует использовать для каждого канала уведомления.
     *
     * @return array
     */
    public function viaQueues()
    {
        return [
            'mail' => 'mail-queue',
            'slack' => 'slack-queue',
        ];
    }

<a name="queued-notifications-and-database-transactions"></a>
#### Уведомления в очереди и транзакции в базе данных

Когда отложенные уведомления отправляются в транзакциях базы данных, они могут быть обработаны очередью до того, как транзакция базы данных будет зафиксирована. Когда это происходит, любые обновления, внесенные вами в модели или записи базы данных во время транзакции базы данных, могут еще не быть отражены в базе данных. Кроме того, любые модели или записи базы данных, созданные в рамках транзакции, могут не существовать в базе данных. Если ваше уведомление зависит от этих моделей, могут возникнуть непредвиденные ошибки при обработке задания, отправляющего уведомление.

Если для параметра `after_commit` конфигурации вашего соединения с очередью установлено значение `false`, то вы все равно можете указать, что конкретное отложенное уведомление должно быть отправлено после того, как все открытые транзакции базы данных были зафиксированы, вызвав метод `afterCommit` при отправке уведомления:

    use App\Notifications\InvoicePaid;

    $user->notify((new InvoicePaid($invoice))->afterCommit());

В качестве альтернативы вы можете вызвать метод `afterCommit` из конструктора вашего уведомления:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        /**
         * Создать новый экземпляр уведомления.
         *
         * @return void
         */
        public function __construct()
        {
            $this->afterCommit();
        }
    }

> {tip} Чтобы узнать больше о том, как обойти эти проблемы, просмотрите документацию, касающуюся [заданий в очереди и транзакций базы данных](queues.md#jobs-and-database-transactions).

<a name="determining-if-the-queued-notification-should-be-sent"></a>
#### Определение необходимости отправки уведомления в очереди

После отправки уведомления в очередь для фоновой обработки оно обычно принимается обработчиком очереди и далее отправляется предполагаемому получателю.

Однако, если вы хотите окончательно указать, следует ли отправлять запланированное уведомление после того, как оно будет обработано обработчиком очереди, то вы можете определить метод `shouldSend` в классе уведомлений. Если этот метод возвращает `false`, то уведомление не будет отправлено:

    /**
     * Определить, нужно ли отправлять уведомление.
     *
     * @param  mixed  $notifiable
     * @param  string  $channel
     * @return bool
     */
    public function shouldSend($notifiable, $channel)
    {
        return $this->invoice->isPaid();
    }

<a name="on-demand-notifications"></a>
### Уведомления по запросу

По желанию можно отправить уведомление кому-то, кто не сохранен как «пользователь» вашего приложения. Используя метод `route` фасада `Notification`, вы можете указать информацию о маршрутизации специального уведомления перед отправкой уведомления:

    Notification::route('mail', 'taylor@example.com')
                ->route('vonage', '5555555555')
                ->route('slack', 'https://hooks.slack.com/services/...')
                ->notify(new InvoicePaid($invoice));

Если вы хотите указать имя получателя при отправке такого уведомления на маршрут `mail`, вы можете предоставить массив, содержащий адрес электронной почты в качестве ключа и имя в качестве значения первого элемента в массиве:

    Notification::route('mail', [
        'barrett@example.com' => 'Barrett Blair',
    ])->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## Почтовые уведомления

<a name="formatting-mail-messages"></a>
### Формирование почтовых сообщений

Если уведомление поддерживает отправку по электронной почте, то вы должны определить метод `toMail` в классе уведомления. Этот метод получит объект `$notifiable` и должен вернуть экземпляр `Illuminate\Notifications\Messages\MailMessage`.

Класс `MailMessage` содержит несколько простых методов, которые помогут вам создавать транзакционные сообщения электронной почты. Почтовые сообщения могут содержать строки текста, а также «призыв к действию». Давайте посмотрим на пример метода `toMail`:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} Обратите внимание, что мы используем `$this->invoice->id` в нашем методе `toMail`. Вы можете передать любые данные, которые необходимы вашему уведомлению для генерации сообщения, в конструктор уведомления.

В этом примере мы регистрируем приветствие, строку текста, призыв к действию, а затем еще одну строку текста. Эти методы, предоставляемые объектом `MailMessage`, упрощают и ускоряют формирование небольших транзакционных электронных писем. Затем канал `mail` преобразует компоненты сообщения в красивый, отзывчивый HTML-шаблон сообщения электронной почты с аналогом в виде обычного текста. Вот пример электронного письма, созданного каналом `mail`:

<img src="./img/notification-example-2.png">

> {tip} При отправке почтовых уведомлений не забудьте установить параметр `name` в вашем конфигурационном файле `config/app.php`. Это значение будет использоваться в верхнем и нижнем колонтитулах ваших почтовых уведомлений.

<a name="other-mail-notification-formatting-options"></a>
#### Другие параметры формирования почтовых уведомлений

Вместо определения «строк» текста в классе уведомления, вы можете использовать метод `view`, чтобы указать собственный шаблон, который следует использовать для отображения почтового уведомления:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }

Вы можете определить текстовое содержимое для почтового сообщения, указав имя представления в качестве второго элемента массива, передаваемого методу `view`:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            ['emails.name.html', 'emails.name.plain'],
            ['invoice' => $this->invoice]
        );
    }

<a name="error-messages"></a>
#### Сообщения об ошибках

Некоторые уведомления информируют пользователей об ошибках, например о неудачной оплате счета. Вы можете указать, что почтовое сообщение касается ошибки, вызвав метод `error` при построении вашего сообщения. При использовании метода `error` в почтовом сообщении кнопка призыва к действию будет красной вместо черной:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-sender"></a>
### Изменение отправителя

По умолчанию адрес отправителя электронного письма определяется в конфигурационном файле `config/mail.php`. Однако вы можете указать адрес отправителя для конкретного уведомления с помощью метода `from`:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->from('barrett@example.com', 'Barrett Blair')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### Изменение получателя

При отправке уведомлений по каналу `mail` система уведомлений автоматически ищет свойство `email` уведомляемого объекта. Вы можете указать, какой адрес электронной почты будет использоваться для доставки уведомления, определив метод `routeNotificationForMail` объекта уведомления:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Маршрутизация уведомлений для почтового канала.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return array|string
         */
        public function routeNotificationForMail($notification)
        {
            // Вернуть только адрес электронной почты ...
            return $this->email_address;

            // Вернуть адрес электронной почты и имя ...
            return [$this->email_address => $this->name];
        }
    }

<a name="customizing-the-subject"></a>
### Изменение темы сообщения

По умолчанию темой электронного письма является название класса уведомления в регистре «Title Case». Итак, если ваш класс уведомлений называется `InvoicePaid`, то темой электронного письма будет «Invoice Paid». Если вы хотите указать другую тему для сообщения, то вы можете вызвать метод `subject` при создании своего сообщения:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-mailer"></a>
### Изменение почтового драйвера

По умолчанию уведомление по электронной почте будет отправлено с использованием почтового драйвера по умолчанию, определенной в конфигурационном файле `config/mail.php`. Однако вы можете указать другой почтовый драйвер во время выполнения, вызвав метод `mailer` при создании вашего сообщения:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->mailer('postmark')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### Изменение почтовых шаблонов

Вы можете изменить шаблон из HTML и обычного текста, используемый для почтовых уведомлений, опубликовав необходимые ресурсы уведомления. После выполнения этой команды шаблоны почтовых уведомлений будут расположены в каталоге `resources/views/vendor/notifications`:

```shell
php artisan vendor:publish --tag=laravel-notifications
```

<a name="mail-attachments"></a>
### Почтовые вложения

Чтобы добавить вложения к почтовому уведомлению, используйте метод `attach` при создании сообщения. Метод `attach` принимает абсолютный путь к файлу в качестве своего первого аргумента:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file');
    }

При прикреплении файлов к сообщению вы также можете указать отображаемое имя и / или MIME-тип, передав массив в качестве второго аргумента методу `attach`:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',
                        'mime' => 'application/pdf',
                    ]);
    }

В отличие от прикрепления файлов к почтовым отправлениям, вы не можете прикреплять файл непосредственно с диска файлового хранилища с помощью `attachFromStorage`. Лучше использовать метод `attach` с абсолютным путем к файлу на диске. В качестве альтернативы вы можете вернуть [отправление](mail.md#generating-mailables) из метода `toMail`:

    use App\Mail\InvoicePaid as InvoicePaidMailable;

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new InvoicePaidMailable($this->invoice))
                    ->to($notifiable->email)
                    ->attachFromStorage('/path/to/file');
    }

<a name="raw-data-attachments"></a>
#### Почтовые вложения необработанных данных

Метод `attachData` используется для присоединения необработанной строки в качестве вложения. При вызове метода `attachData` вы должны указать имя файла, которое должно быть присвоено вложению:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="adding-tags-metadata"></a>
### Теги и метаданные

Некоторые сторонние почтовые поставщики, такие как Mailgun и Postmark, поддерживают «теги» и «метаданные» сообщения, которые могут использоваться для группировки и отслеживания электронных писем, отправляемых вашим приложением. Вы можете добавить теги и метаданные к сообщению электронной почты с помощью методов `tag` и `metadata`:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->greeting('Comment Upvoted!')
                    ->tag('upvote')
                    ->metadata('comment_id', $this->comment->id);
    }

Если ваше приложение использует драйвер Mailgun, то вы можете обратиться к документации Mailgun для получения дополнительной информации о [тегах](https://documentation.mailgun.com/en/latest/user_manual.html#tagging-1) и [метаданных](https://documentation.mailgun.com/en/latest/user_manual.html#attaching-data-to-messages). Кроме того, можно также обратиться к документации Postmark для получения дополнительной информации об их поддержке [тегов](https://postmarkapp.com/blog/tags-support-for-smtp) и [метаданных](https://postmarkapp.com/support/article/1125-custom-metadata-faq).

Если ваше приложение использует Amazon SES для отправки электронных писем, то вам следует использовать метод `metadata` для прикрепления [тегов SES](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) к сообщению.
Теги и метаданные могут быть добавлены в `MailMessage` – они используются вашей почтовой службой для фильтрации/обработки.

<a name="customizing-the-symfony-message"></a>
### Настройка сообщения Symfony

Метод `withSymfonyMessage` базового класса `MailMessage` позволяет вам зарегистрировать замыкание, которое будет вызываться с экземпляром сообщения Symfony перед отправкой сообщения. Это дает вам возможность глубокой настройки сообщения перед его доставкой:

    use Symfony\Component\Mime\Email;

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->withSymfonyMessage(function (Email $message) {
                        $message->getHeaders()->addTextHeader(
                            'Custom-Header', 'Header Value'
                        );
                    });
    }

<a name="using-mailables"></a>
### Использование почтовых отправлений

При необходимости вы можете вернуть полностью [объект почтового отправления](mail.md) из метода `toMail` вашего уведомления. При возврате `Mailable` вместо `MailMessage` вам нужно будет указать получателя сообщения с помощью метода `to` объекта почтового отправления:

    use App\Mail\InvoicePaid as InvoicePaidMailable;

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new InvoicePaidMailable($this->invoice))
                    ->to($notifiable->email);
    }

<a name="mailables-and-on-demand-notifications"></a>
#### Почтовые отправления и уведомления по запросу

Если вы отправляете [уведомление по запросу](#on-demand-notifications), то экземпляр `$notifiable`, переданный методу `toMail`, будет экземпляром `Illuminate\Notifications\AnonymousNotifiable`, содержащий метод `routeNotificationFor`, который можно использовать для получения адреса электронной почты для отправления уведомления по запросу:

    use App\Mail\InvoicePaid as InvoicePaidMailable;
    use Illuminate\Notifications\AnonymousNotifiable;

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        $address = $notifiable instanceof AnonymousNotifiable
                ? $notifiable->routeNotificationFor('mail')
                : $notifiable->email;

        return (new InvoicePaidMailable($this->invoice))
                    ->to($address);
    }

<a name="previewing-mail-notifications"></a>
### Предварительный просмотр почтовых уведомлений

При разработке шаблона почтового уведомления удобно быстро просмотреть визуализированное почтовое сообщение в браузере, как типичный шаблон Blade. По этой причине Laravel позволяет вам возвращать любое почтовое сообщение непосредственно из замыкания маршрута или контроллера. При возврате `MailMessage`, оно будет обработано и отображено в браузере, что позволит вам быстро просмотреть его дизайн без необходимости отправлять его на реальный адрес электронной почты:

    use App\Models\Invoice;
    use App\Notifications\InvoicePaid;

    Route::get('/notification', function () {
        $invoice = Invoice::find(1);

        return (new InvoicePaid($invoice))
                    ->toMail($invoice->user);
    });

<a name="markdown-mail-notifications"></a>
## Почтовые уведомления с разметкой Markdown

Почтовые уведомления с разметкой Markdown позволяют вам воспользоваться преимуществами предварительно созданных шаблонов почтовых уведомлений. Поскольку сообщения написаны на Markdown, Laravel может отображать красивые, отзывчивые HTML-шаблоны для сообщений, а также автоматически генерировать аналог в виде простого текста.

<a name="generating-the-message"></a>
### Генерация сообщения

Чтобы сгенерировать уведомление с соответствующим шаблоном Markdown, вы можете использовать параметр `--markdown` команды `make:notification` Artisan:

```shell
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

Как и все другие почтовые уведомления, уведомления, использующие шаблоны Markdown, должны определять метод `toMail` в своем классе уведомлений. Однако вместо использования методов `line` и `action` для создания уведомления используйте метод `markdown`, чтобы указать имя шаблона Markdown, который следует использовать. Массив данных, который вы хотите сделать доступным для шаблона, может быть передан в качестве второго аргумента метода:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### Написание сообщения

Почтовые уведомления Markdown используют комбинацию компонентов Blade и синтаксиса Markdown, которые позволяют легко создавать почтовые уведомления, используя предварительно созданные компоненты уведомлений Laravel:

```blade
@component('mail::message')
# Invoice Paid

Your invoice has been paid!

@component('mail::button', ['url' => $url])
View Invoice
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

<a name="button-component"></a>
#### Компонент Button

Компонент кнопки отображает ссылку на кнопку по центру. Компонент принимает два аргумента: `url` и необязательный `color`. Поддерживаемые цвета: `primary`, `green` и `red`. Вы можете добавить к уведомлению столько компонентов кнопки, сколько захотите:

```blade
@component('mail::button', ['url' => $url, 'color' => 'green'])
View Invoice
@endcomponent
```

<a name="panel-component"></a>
#### Компонент Panel

Компонент панели отображает указанный блок текста на панели, цвет фона которой немного отличается от цвета остальной части сообщения. Это позволяет привлечь внимание к указанному блоку текста:

```blade
@component('mail::panel')
This is the panel content.
@endcomponent
```

<a name="table-component"></a>
#### Компонент Table

Компонент таблицы позволяет преобразовать таблицу Markdown в таблицу HTML. Компонент принимает в качестве содержимого таблицу Markdown. Выравнивание столбцов таблицы поддерживается с использованием синтаксиса выравнивания таблицы Markdown по умолчанию:

```blade
@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
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

Чтобы настроить тему для отдельного уведомления, вы можете вызвать метод `theme` при создании почтового сообщения уведомления. Метод `theme` принимает имя темы, которая должна использоваться при отправке уведомления:

    /**
     * Получить содержимое почтового уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->theme('invoice')
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="database-notifications"></a>
## Уведомления через канал `database`

<a name="database-prerequisites"></a>
### Предварительная подготовка базы данных

Канал уведомлений `database` хранит информацию уведомления в таблице базы данных. Эта таблица будет содержать такую информацию, как тип уведомления, а также JSON-структуру данных, которая описывает уведомление.

Вы можете запросить таблицу, чтобы отобразить уведомления в пользовательском интерфейсе вашего приложения. Но прежде чем вы сможете это сделать, вам нужно будет создать таблицу базы данных для хранения ваших уведомлений. Вы можете использовать команду `notifications:table` для создания [миграции](migrations.md) с необходимой схемой таблицы:

```shell
php artisan notifications:table

php artisan migrate
```

<a name="formatting-database-notifications"></a>
### Формирование уведомлений канала `database`

Чтобы уведомление было сохранено в таблице базы данных, вы должны определить метод `toDatabase` или `toArray` в классе уведомления. Каждый из этих методов получает объект `$notifiable` и должен возвращать простой массив PHP. Возвращенный массив будет закодирован как JSON и сохранен в столбце `data` вашей таблицы `notifications`. Давайте посмотрим на пример метода `toArray`:

    /**
     * Получить массив данных уведомления.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

<a name="todatabase-vs-toarray"></a>
#### Методы `toDatabase` и `toArray`

Метод `toArray` также используется каналом `broadcast`, чтобы определить, какие данные транслировать в JavaScript-приложение на клиентской стороне. Если вы хотите иметь два разных массива данных для каналов `database` и `broadcast`, то вы должны определить метод `toDatabase` вместо метода `toArray`.

<a name="accessing-the-notifications"></a>
### Доступ к уведомлениям

После сохранения уведомления в базу данных, вам понадобится удобный способ доступа к нему из уведомляемых объектов. Трейт `Illuminate\Notifications\Notifiable`, который по умолчанию расположен в модели `App\Models\User` Laravel, содержит [отношение](eloquent-relationships.md) `notifications` Eloquent, возвращающее уведомления для объекта. Вы можете обратиться к этому методу, как и к любому другому отношению Eloquent, чтобы получить уведомления. По умолчанию уведомления будут упорядочены по столбцу `created_at` временной метки, причем самые последние уведомления будут помещены в начало коллекции:

    $user = App\Models\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Для получения только «непрочитанных» уведомлений, используйте отношение `unreadNotifications`. Опять же, эти уведомления будут упорядочены по столбцу `created_at` временной метки с самыми последними уведомлениями в начале коллекции:

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} Чтобы получить доступ к уведомлениям в JavaScript-приложении на клиентской стороне, вы должны определить контроллер уведомлений для своего приложения, который возвращает уведомления для уведомляемого объекта, такого как текущий пользователь. Затем вы можете сделать HTTP-запрос к URL-адресу этого контроллера из своего JavaScript-приложения на клиентской стороне.

<a name="marking-notifications-as-read"></a>
### Отметка прочитанных уведомлений

По желанию можно пометить уведомление как «прочитанное», когда пользователь его просматривает. Трейт `Illuminate\Notifications\Notifiable` содержит метод `markAsRead`, который обновляет столбец `read_at` записи уведомления в базе данных:

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Однако вместо того, чтобы перебирать каждое уведомление, вы можете использовать метод `markAsRead` непосредственно для коллекции уведомлений:

    $user->unreadNotifications->markAsRead();

Вы также можете выполнить запрос массового обновления, чтобы пометить все уведомления как прочитанные, не извлекая их из базы данных:

    $user = App\Models\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

Вы можете полностью удалить уведомления из таблицы, используя метод `delete`:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Трансляция уведомлений

<a name="broadcast-prerequisites"></a>
### Предварительная подготовка трансляции

Перед трансляцией уведомлений вы должны настроить и ознакомиться со службами [трансляции событий](broadcasting.md) Laravel. Трансляция событий – способ реагирования на серверные события Laravel из своего JavaScript-приложения на клиентской стороне.

<a name="formatting-broadcast-notifications"></a>
### Формирование транслируемых уведомлений

Канал `broadcast` транслирует уведомления с использованием служб [трансляции событий](broadcasting.md) Laravel, что позволяет вашему JavaScript-приложению на клиентской стороне улавливать уведомления в режиме реального времени. Если уведомление поддерживает трансляцию, то вы должны определить метод `toBroadcast` в классе уведомления. Этот метод получит объект `$notifiable` и должен вернуть экземпляр `BroadcastMessage`. Если метод `toBroadcast` не существует, то метод `toArray` будет использоваться для сбора данных, которые следует транслировать. Возвращенные данные будут закодированы как JSON и переданы вашему JavaScript-приложению на клиентской стороне. Давайте посмотрим на пример метода `toBroadcast`:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Получить содержимое транслируемого уведомления.
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

<a name="broadcast-queue-configuration"></a>
#### Конфигурирование очереди трансляции

Все транслируемые уведомления ставятся в очередь для трансляции. Если вы хотите изменить соединение очереди или имя очереди, которое используется для постановки в очередь трансляции, то вы можете использовать методы `onConnection` и `onQueue` экземпляра `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

<a name="customizing-the-notification-type"></a>
#### Изменение типа транслируемого уведомления

В дополнение к указанным вами данным все транслируемые уведомления также имеют поле `type`, содержащее полное имя класса уведомления. Если вы хотите изменить `type` уведомления, то вы можете определить метод `broadcastType` в классе уведомления:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Получить тип транслируемого уведомления.
     *
     * @return string
     */
    public function broadcastType()
    {
        return 'broadcast.message';
    }

<a name="listening-for-notifications"></a>
### Прослушивание транслируемых уведомлений

Уведомления будут транслироваться по частному каналу, в формате с использованием соглашения `{notifiable}.{id}`. Итак, если вы отправляете уведомление экземпляру `App\Models\User` с идентификатором `1`, то уведомление будет транслироваться по частному каналу `App.Models.User.1`. При использовании [Laravel Echo](broadcasting.md#client-side-installation) вы можете легко прослушивать уведомления канала, используя метод `notification`:

    Echo.private('App.Models.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

<a name="customizing-the-notification-channel"></a>
#### Изменение канала транслируемого уведомления

Если вы хотите изменить канал, на котором транслируются уведомления объекта, то вы можете определить метод `receivesBroadcastNotificationsOn` объекта уведомления:

    <?php

    namespace App\Models;

    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Каналы, по которым пользователь получает рассылку уведомлений.
         *
         * @return string
         */
        public function receivesBroadcastNotificationsOn()
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## Уведомления через SMS

<a name="sms-prerequisites"></a>
### Предварительная подготовка канала SMS

Отправка SMS-уведомлений в Laravel обеспечивается [Vonage](https://www.vonage.com/) (бывший Nexmo). Прежде чем вы сможете отправлять уведомления через Vonage, вам необходимо установить пакеты Composer:

    composer require laravel/vonage-notification-channel guzzlehttp/guzzle

Пакет содержит [конфигурационный файл](https://github.com/laravel/vonage-notification-channel/blob/3.x/config/vonage.php). Однако вам не обязательно экспортировать этот файл конфигурации в собственное приложение. Вы можете просто использовать переменные окружения `VONAGE_KEY` и `VONAGE_SECRET` для определения публичного и секретного ключей Vonage.

После определения ключей вы можете установить переменную окружения `VONAGE_SMS_FROM`, которая определяет номер телефона, с которого по умолчанию должны отправляться ваши SMS-сообщения. Вы можете сгенерировать этот номер телефона в панели управления Vonage:

    VONAGE_SMS_FROM=15556666666

<a name="formatting-sms-notifications"></a>
### Формирование уведомлений через SMS

Если уведомление поддерживает отправку в виде SMS, то вы должны определить метод `toVonage` в классе уведомлений. Этот метод получит объект `$notifiable` и должен вернуть экземпляр `Illuminate\Notifications\Messages\VonageMessage`:

    /**
     * Получить SMS-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\VonageMessage
     */
    public function toVonage($notifiable)
    {
        return (new VonageMessage)
                    ->content('Your SMS message content');
    }

<a name="unicode-content"></a>
#### Содержимое Unicode

Если ваше SMS-сообщение будет содержать символы Unicode, то вы должны вызвать метод `unicode` при создании экземпляра `VonageMessage`:

    /**
     * Получить SMS-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\VonageMessage
     */
    public function toVonage($notifiable)
    {
        return (new VonageMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### Изменение номера отправителя

Если вы хотите отправить уведомление с номера телефона, который отличается от номера телефона, указанного вашей переменной окружения `VONAGE_SMS_FROM`, вы можете вызвать метод `from` экземпляра `VonageMessage`:

    /**
     * Получить SMS-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\VonageMessage
     */
    public function toVonage($notifiable)
    {
        return (new VonageMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="adding-a-client-reference"></a>
### Добавление ссылки на клиента

Если вы хотите отслеживать затраты каждого пользователя, команды или клиента, то вы можете добавить в уведомление «ссылку на клиента». Vonage позволит вам создавать отчеты, используя эту ссылку, чтобы вы могли лучше понять использование SMS конкретным клиентом. Ссылка на клиента может быть любой строкой, имеющей длину до 40 символов:

    /**
     * Получить SMS-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\VonageMessage
     */
    public function toVonage($notifiable)
    {
        return (new VonageMessage)
                    ->clientReference((string) $notifiable->id)
                    ->content('Your SMS message content');
    }

<a name="routing-sms-notifications"></a>
### Маршрутизация SMS-уведомлений

Для отправки уведомления с использованием Vonage на необходимый номер телефона, определите метод `routeNotificationForVonage` вашего уведомляемого объекта:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Маршрутизация уведомлений для канала Vonage.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForVonage($notification)
        {
            return $this->phone_number;
        }
    }

<a name="slack-notifications"></a>
## Уведомления через Slack

<a name="slack-prerequisites"></a>
### Предварительная подготовка канала Slack

Прежде чем вы сможете отправлять уведомления через Slack, вы должны установить канал уведомлений Slack через Composer:

```shell
composer require laravel/slack-notification-channel
```

Вам также потребуется создать [приложение Slack](https://api.slack.com/apps?new_app=1) для своей команды. После создания приложения вы должны настроить «Incoming Webhook» для рабочей области. Затем Slack предоставит вам URL-адрес веб-хука, который вы можете использовать при [маршрутизации Slack-уведомлений](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>
### Формирование уведомления через Slack

Если уведомление поддерживает отправку в виде Slack-сообщения, то вы должны определить метод `toSlack` класса уведомления. Этот метод получит объект `$notifiable` и должен вернуть экземпляр `Illuminate\Notifications\Messages\SlackMessage`. Сообщения Slack могут содержать текстовое содержимое, а также «вложение», которое формирует дополнительный текст или массив полей. Давайте посмотрим на пример `toSlack`:

    /**
     * Получить представление Slack-уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

<a name="slack-attachments"></a>
### Вложения Slack-уведомлений

Вы также можете добавлять «вложения» к сообщениям Slack. Вложения предоставляют более широкие возможности форматирования, чем простые текстовые сообщения. В этом примере мы отправим уведомление об ошибке об исключении, которое произошло в приложении, включая ссылку для просмотра дополнительных сведений об исключении:

    /**
     * Получить представление Slack-уведомления.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

Вложения также позволяют указать массив данных, которые должны быть представлены пользователю. Указанные данные будут представлены в формате таблицы для удобства при чтении:

    /**
     * Получить представление Slack-уведомления.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

<a name="markdown-attachment-content"></a>
#### Содержимое вложений с разметкой Markdown

Если некоторые из ваших полей вложения содержат разметку Markdown, то вы можете использовать метод `markdown`, чтобы указать Slack на синтаксический анализ и отображение указанных полей вложения как текста в формате Markdown. Значения, принимаемые этим методом: `pretext`, `text` и / или `fields`. Дополнительные сведения о форматировании вложений Slack см. в [документации API Slack](https://api.slack.com/docs/message-formatting#message_formatting):

    /**
     * Получить представление Slack-уведомления.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was *not found*.')
                                   ->markdown(['text']);
                    });
    }

<a name="routing-slack-notifications"></a>
### Маршрутизация Slack-уведомлений

Для отправки уведомления с использованием Slack в соответствующую команду и канал, определите метод `routeNotificationForSlack` вашего уведомляемого объекта. Этот метод должен вернуть URL-адрес веб-хука, на который должно быть доставлено уведомление. URL-адреса веб-хуков могут быть сгенерированы путем добавления сервиса «Incoming Webhook» в вашу команду Slack:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Маршрутизация уведомлений для канала Slack.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForSlack($notification)
        {
            return 'https://hooks.slack.com/services/...';
        }
    }

<a name="localizing-notifications"></a>
## Локализация уведомлений

Laravel позволяет отправлять уведомления, используя язык, отличный от текущего языка запроса, и даже будет помнить его, если уведомление находится в очереди.

Для этого класс `Illuminate\Notifications\Notification` содержит метод `locale` для установки желаемого языка. Приложение изменит язык при анализе уведомления, а затем вернется к предыдущему языку, когда анализ будет завершен:

    $user->notify((new InvoicePaid($invoice))->locale('es'));

Локализация нескольких уведомляемых записей также доступна через фасад `Notification`:

    Notification::locale('es')->send(
        $users, new InvoicePaid($invoice)
    );

<a name="user-preferred-locales"></a>
### Предпочитаемые пользователем локализации

Иногда приложения хранят предпочтительный язык каждого пользователя. Реализуя контракт `HasLocalePreference` в вашей уведомляемой модели, вы можете указать Laravel использовать этот сохраненный язык при отправке уведомления:

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

После того, как вы реализовали интерфейс, Laravel будет автоматически использовать предпочтительный язык при отправке уведомлений и почтовых сообщений модели. Следовательно, при использовании этого интерфейса нет необходимости вызывать метод `locale`:

    $user->notify(new InvoicePaid($invoice));

<a name="notification-events"></a>
## События уведомления

<a name="notification-sending-event"></a>
#### Событие отправки уведомления

При отправке уведомления система уведомлений запускает [событие](events.md) `Illuminate\Notifications\Events\NotificationSending`. Событие содержит «уведомляемую» сущность и сам экземпляр уведомления. Как правило, регистрация слушателей этого события осуществляется в поставщике `App\Providers\EventServiceProvider`:

    /**
     * Карта слушателей событий приложения.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSending' => [
            'App\Listeners\CheckNotificationStatus',
        ],
    ];

Уведомление не будет отправлено, если слушатель события `NotificationSending` вернет `false` из своего метода `handle`:

    use Illuminate\Notifications\Events\NotificationSending;

    /**
     * Обработать переданное событие.
     *
     * @param  \Illuminate\Notifications\Events\NotificationSending  $event
     * @return void
     */
    public function handle(NotificationSending $event)
    {
        return false;
    }

В слушателе события вы можете получить доступ к свойствам `notifiable`, `notification` и `channel` события для получения сведений о получателе уведомления или о самом уведомлении:

    /**
     * Обработать переданное событие.
     *
     * @param  \Illuminate\Notifications\Events\NotificationSending  $event
     * @return void
     */
    public function handle(NotificationSending $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="notification-sent-event"></a>
#### Событие отправленного уведомления

После отправки уведомления система уведомлений запускает [событие](events.md) `Illuminate\Notifications\Events\NotificationSent`. Событие содержит «уведомляемую» сущность и сам экземпляр уведомления. Как правило, регистрация слушателей этого события осуществляется в поставщике `App\Providers\EventServiceProvider`:

    /**
     * Карта слушателей событий приложения.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} После регистрации слушателей в вашем `EventServiceProvider` используйте команду `event:generate` Artisan, чтобы быстро сгенерировать классы слушателей.

В слушателе события вы можете получить доступ к свойствам `notifiable`, `notification`, `channel` и `response` события для получения сведений о получателе уведомления или о самом уведомлении:

    /**
     * Обработать переданное событие.
     *
     * @param  \Illuminate\Notifications\Events\NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
        // $event->response
    }

<a name="custom-channels"></a>
## Пользовательские каналы уведомлений

Laravel предлагает несколько каналов уведомлений, но вы можете написать свои собственные драйверы для доставки уведомлений по другим каналам. С Laravel это сделать просто. Для начала определите класс, содержащий метод `send`. Этот метод должен получать два аргумента: `$notifiable` и `$notification`.

В методе `send` вы можете вызывать методы уведомления, чтобы получить объект сообщения, понятный вашему каналу, а затем отправить уведомление необходимому экземпляру `$notifiable`:

    <?php

    namespace App\Notifications;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Отправить переданное уведомление.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // Отправка уведомления экземпляру `$notifiable` ...
        }
    }

Как только ваш класс канала уведомления был определен, вы можете вернуть имя класса из метода `via` любого из ваших уведомлений. В этом примере метод вашего уведомления `toVoice` может возвращать любой объект для формирования голосовых сообщений. Например, вы можете определить свой собственный класс `VoiceMessage` для формирования таких сообщений:

    <?php

    namespace App\Notifications;

    use App\Notifications\Messages\VoiceMessage;
    use App\Notifications\VoiceChannel;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Получить каналы доставки уведомлений.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * Получить содержимое голосового сообщения.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
