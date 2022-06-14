# Laravel 9 · Трансляция событий

- [Введение](#introduction)
- [Установка на стороне сервера](#server-side-installation)
    - [Конфигурирование](#configuration)
    - [Pusher Channels](#pusher-channels)
    - [Ably](#ably)
    - [Альтернативы с открытым исходным кодом](#open-source-alternatives)
- [Установка на стороне клиента](#client-side-installation)
    - [Pusher Channels](#client-pusher-channels)
    - [Ably](#client-ably)
- [Обзор концепции](#concept-overview)
    - [Пример использования](#using-example-application)
- [Определение транслируемых событий](#defining-broadcast-events)
    - [Имя транслируемого события](#broadcast-name)
    - [Данные трансляции](#broadcast-data)
    - [Очередь трансляции](#broadcast-queue)
    - [Условия трансляции](#broadcast-conditions)
    - [Трансляция и транзакции базы данных](#broadcasting-and-database-transactions)
- [Авторизация каналов](#authorizing-channels)
    - [Определение маршрутов авторизации](#defining-authorization-routes)
    - [Определение авторизации канала](#defining-authorization-callbacks)
    - [Определение класса канала](#defining-channel-classes)
- [Трансляция событий](#broadcasting-events)
    - [Трансляция событий только остальным пользователям](#only-to-others)
    - [Настройка соединения](#customizing-the-connection)
- [Прием трансляций](#receiving-broadcasts)
    - [Прослушивание событий](#listening-for-events)
    - [Покидание канала](#leaving-a-channel)
    - [Пространства имён](#namespaces)
- [Каналы присутствия](#presence-channels)
    - [Авторизация каналов присутствия](#authorizing-presence-channels)
    - [Присоединение к каналам присутствия](#joining-presence-channels)
    - [Трансляция на каналы присутствия](#broadcasting-to-presence-channels)
- [Трансляция событий модели](#model-broadcasting)
    - [Соглашения трансляции событий модели](#model-broadcasting-conventions)
    - [Прослушивание транслируемых событий модели](#listening-for-model-broadcasts)
- [Клиентские события](#client-events)
- [Уведомления](#notifications)

<a name="introduction"></a>
## Введение

Во многих современных веб-приложениях веб-сокеты используются для реализации пользовательских интерфейсов, обновляемых в реальном времени. Когда некоторые данные обновляются на сервере, тогда обычно отправляется сообщение через соединение WebSocket для обработки клиентом. Веб-сокеты предоставляют более эффективную альтернативу постоянному опросу сервера вашего приложения на предмет изменений данных, которые должны быть отражены в вашем пользовательском интерфейсе.

Например, представьте, что ваше приложение может экспортировать данные пользователя в файл CSV и отправлять этот файл ему по электронной почте. Однако создание этого CSV-файла занимает несколько минут, поэтому вы можете создать и отправить CSV-файл по почте, поместив [задание в очередь](queues.md). Когда файл CSV будет создан и отправлен пользователю, тогда мы можем использовать **широковещание** для отправки события `App\Events\UserDataExported`, которое будет получено в JavaScript нашего приложения. Как только событие будет получено, мы можем отобразить сообщение пользователю о том, что его файл CSV был отправлен ему по электронной почте без необходимости в обновлении страницы.

Чтобы помочь вам в создании подобного рода функционала, Laravel упрощает «вещание» серверных [событий](events.md) Laravel через соединение WebSocket. Трансляция ваших событий Laravel позволяет вам использовать одни и те же имена событий и данные между серверным приложением Laravel и клиентским JavaScript-приложением.

Основные концепции трансляции просты: клиенты подключаются к именованным каналам во внешнем интерфейсе, в то время как ваше приложение Laravel транслирует события на эти каналы во внутреннем интерфейсе. Эти события могут содержать любые дополнительные данные, которые вы хотите сделать доступными для внешнего интерфейса.

<a name="supported-drivers"></a>
#### Поддерживаемые драйверы

По умолчанию Laravel содержит два серверных драйвера трансляции на выбор: [Pusher Channels](https://pusher.com/channels) и [Ably](https://ably.io). Однако пакеты сообщества, например, [laravel-websockets](https://beyondco.de/docs/laravel-websockets/getting-started/introduction) и [soketi](https://docs.soketi.app/) предлагают дополнительные драйверы трансляции без использования платных провайдеров.

> {tip} Прежде чем ближе ознакомиться с трансляцией событий, убедитесь, что вы прочитали документацию Laravel о [событиях и слушателях](events.md).

<a name="server-side-installation"></a>
## Установка на стороне сервера

Чтобы начать использовать трансляцию событий Laravel, нам нужно выполнить некоторую настройку в приложении Laravel, а также установить некоторые пакеты.

Трансляция событий осуществляется серверным драйвером трансляции, который транслирует ваши события Laravel, получаемые браузером клиента через Laravel Echo (библиотека JavaScript). Не волнуйтесь – мы рассмотрим каждую часть процесса установки шаг за шагом.

<a name="configuration"></a>
### Конфигурирование

Вся конфигурация трансляций событий вашего приложения хранится в конфигурационном файле `config/broadcasting.php`. Laravel из коробки поддерживает несколько драйверов трансляции: [Pusher Channels](https://pusher.com/channels), [Redis](redis.md) и драйвер `log` для локальной разработки и отладки. Кроме того, поддерживается драйвер `null`, который позволяет полностью отключить трансляцию во время тестирования. В конфигурационном файле `config/broadcasting.php` содержится пример конфигурации для каждого из этих драйверов.

<a name="broadcast-service-provider"></a>
#### Поставщик службы трансляции

Перед трансляцией каких-либо событий, вам сначала необходимо зарегистрировать поставщика `App\Providers\BroadcastServiceProvider`. В новых приложениях Laravel вам нужно только раскомментировать этого поставщика в массиве `providers` конфигурационного файла `config/app.php`. Поставщик `BroadcastServiceProvider` содержит код, необходимый для регистрации маршрутов авторизации трансляции.

<a name="queue-configuration"></a>
#### Конфигурирование очереди

Вам также потребуется настроить и запустить [обработчик очереди](queues.md). Все трансляции событий выполняются через задания в очереди, так что время отклика вашего приложения не сильно зависит от транслируемых событий.

<a name="pusher-channels"></a>
### Pusher Channels

Если вы планируете транслировать свои события с помощью [Pusher Channels](https://pusher.com/channels), то вам следует установить PHP SDK Pusher Channels с помощью менеджера пакетов Composer:

```shell
composer require pusher/pusher-php-server
```

Далее, вы должны настроить свои учетные данные Pusher Channels в конфигурационном файле `config/broadcasting.php`. Пример конфигурации Pusher Channels уже содержится в этом файле, что позволяет быстро указать параметры `key`, `secret` и `app_id`. Как правило, эти значения должны быть установлены через [переменные окружения](configuration.md#environment-configuration) `PUSHER_APP_KEY`, `PUSHER_APP_SECRET` и `PUSHER_APP_ID`:

```ini
PUSHER_APP_ID=your-pusher-app-id
PUSHER_APP_KEY=your-pusher-key
PUSHER_APP_SECRET=your-pusher-secret
PUSHER_APP_CLUSTER=mt1
```

Конфигурация `pusher` в файле `config/broadcasting.php` также позволяет вам указывать дополнительные параметры, которые поддерживаются Pusher, например, `cluster`.

Далее, вам нужно будет изменить драйвер трансляции на `pusher` в файле `.env`:

```ini
BROADCAST_DRIVER=pusher
```

И, наконец, вы готовы установить и настроить [Laravel Echo](#client-side-installation), который будет получать транслируемые события на клиентской стороне.

<a name="pusher-compatible-open-source-alternatives"></a>
#### Альтернативы Pusher с открытым исходным кодом

Пакеты [laravel-websockets](https://github.com/beyondcode/laravel-websockets) и [soketi](https://docs.soketi.app/) предоставляют совместимые с Pusher серверы WebSocket для Laravel. Эти пакеты позволяют вам использовать всю мощь трансляции Laravel без использования коммерческого провайдера WebSocket. Для получения дополнительной информации об установке и использовании этих пакетов обратитесь к нашей документации по [альтернативам с открытым исходным кодом](#open-source-alternatives).

<a name="ably"></a>
### Ably

Если вы планируете транслировать свои события с помощью [Ably](https://ably.io), то вам следует установить PHP SDK Ably с помощью менеджера пакетов Composer:

```shell
composer require ably/ably-php
```

Далее, вы должны настроить свои учетные данные Ably в конфигурационном файле `config/broadcasting.php`. Пример конфигурации Ably уже содержится в этом файле, что позволяет быстро указать параметр `key`. Как правило, это значение должно быть установлено через [переменную окружения](configuration.md#environment-configuration) `ABLY_KEY`:

```ini
ABLY_KEY=your-ably-key
```

Далее, вам нужно будет изменить драйвер трансляции на `ably` в файле `.env`:

```ini
BROADCAST_DRIVER=ably
```

И, наконец, вы готовы установить и настроить [Laravel Echo](#client-side-installation), который будет получать транслируемые события на клиентской стороне.

<a name="open-source-alternatives"></a>
### Альтернативы с открытым исходным кодом

<a name="open-source-alternatives-php"></a>
#### PHP

Пакет [laravel-websockets](https://github.com/beyondcode/laravel-websockets) – это пакет для Laravel на «чистом» PHP, совместимый с Pusher. Этот пакет позволяет вам использовать всю мощь трансляции Laravel без использования платных провайдеров WebSocket. Для получения дополнительной информации об установке и использовании этого пакета обратитесь к его [официальной документации](https://beyondco.de/docs/laravel-websockets).

<a name="open-source-alternatives-node"></a>
#### Node

[Soketi] (https://github.com/soketi/soketi) – это сервер WebSocket для Laravel, совместимый с Pusher на основе Node. Под капотом Soketi использует µWebSockets.js для максимальной масштабируемости и скорости. Этот пакет позволяет вам использовать всю мощь трансляции Laravel без коммерческого провайдера WebSocket. Для получения дополнительной информации об установке и использовании этого пакета обратитесь к его [официальной документации](https://docs.soketi.app/).

<a name="client-side-installation"></a>
## Установка на стороне клиента

<a name="client-pusher-channels"></a>
### Pusher Channels

[Laravel Echo](https://github.com/laravel/echo) – это JavaScript-библиотека, которая позволяет безболезненно подписаться на каналы и прослушивать события, транслируемые вашим драйвером трансляции на стороне сервера. Вы можете установить Echo через менеджер пакетов NPM. В этом примере мы также установим пакет `pusher-js`, так как мы будем использовать вещатель Pusher Channels:

```shell
npm install --save-dev laravel-echo pusher-js
```

После установки Echo вы готовы создать новый экземпляр Echo в JavaScript вашего приложения. Отличное место для этого – окончание файла `resources/js/bootstrap.js`, который уже содержится в фреймворке Laravel. По умолчанию пример конфигурации Echo также находится в этом файле – вам просто нужно раскомментировать его:

```js
import Echo from 'laravel-echo';

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```

После того, как вы раскомментировали и настроили конфигурацию Echo в соответствии с вашими потребностями, вы можете скомпилировать исходники вашего приложения:

```shell
npm run dev
```

> {tip} Чтобы узнать больше о компиляции JavaScript-исходников вашего приложения, обратитесь к документации [Laravel Mix](mix.md).

<a name="using-an-existing-client-instance"></a>
#### Использование существующего экземпляра клиента

Если у вас уже есть предварительно настроенный экземпляр клиента Pusher Channels, который вы бы хотели использовать в Echo, то вы можете передать его Echo с помощью свойства конфигурации `client`:

```js
import Echo from 'laravel-echo';

const client = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key',
    client: client
});
```

<a name="client-ably"></a>
### Ably

[Laravel Echo](https://github.com/laravel/echo) – это JavaScript-библиотека, которая позволяет безболезненно подписаться на каналы и прослушивать события, транслируемые вашим драйвером трансляции на стороне сервера. Вы можете установить Echo через менеджер пакетов NPM. В этом примере мы также установим пакет `pusher-js`.

Вы можете задаться вопросом, зачем нам устанавливать библиотеку `pusher-js` JavaScript, даже если мы используем Ably для трансляции наших событий. К счастью, Ably имеет режим совместимости Pusher, который позволяет нам использовать протокол Pusher при прослушивании событий в нашем клиентском приложении:

```shell
npm install --save-dev laravel-echo pusher-js
```

**Прежде чем продолжить, вы должны включить поддержку протокола Pusher в настройках вашего приложения Ably. Вы можете включить эту функцию в разделе настроек «Protocol Adapter Settings» панели вашего приложения Ably.**

После установки Echo вы готовы создать новый экземпляр Echo в JavaScript вашего приложения. Отличное место для этого – окончание файла `resources/js/bootstrap.js`, который уже содержится в фреймворке Laravel. По умолчанию пример конфигурации Echo также находится в этом файле – вам просто нужно раскомментировать его; однако конфигурация по умолчанию в файле `bootstrap.js` предназначена для Pusher. Вы можете скопировать конфигурацию ниже, чтобы применить ее к Ably:

```js
import Echo from 'laravel-echo';

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_ABLY_PUBLIC_KEY,
    wsHost: 'realtime-pusher.ably.io',
    wsPort: 443,
    disableStats: true,
    encrypted: true,
});
```

Обратите внимание, что наша конфигурация Echo для Ably ссылается на переменную окружения `MIX_ABLY_PUBLIC_KEY`. Значение этой переменной должно быть вашим публичным ключом Ably. Ваш публичный ключ – это часть ключа Ably перед символом `:`.

После того, как вы раскомментировали и настроили конфигурацию Echo в соответствии с вашими потребностями, вы можете скомпилировать исходники вашего приложения:

```shell
npm run dev
```

> {tip} Чтобы узнать больше о компиляции JavaScript-исходников вашего приложения, обратитесь к документации [Laravel Mix](mix.md).

<a name="concept-overview"></a>
## Обзор концепции

Трансляция событий Laravel позволяет транслировать серверные события Laravel в JavaScript-приложение на клиентской стороне, используя драйверный подход к WebSockets. В настоящее время Laravel поставляется с драйверами [Pusher Channels](https://pusher.com/channels) и [Ably](https://ably.io). События могут быть легко обработаны на стороне клиента с помощью JavaScript-пакета [Laravel Echo](#client-side-installation).

События транслируются по «каналам», которые могут быть публичными или частными. Любой посетитель вашего приложения может подписаться на публичный канал без какой-либо аутентификации или авторизации; однако, чтобы подписаться на частный канал, пользователь должен быть аутентифицирован и авторизован для прослушивания событий на этом канале.

> {tip} Если вы хотите рассмотреть альтернативы Pusher с открытым исходным кодом, то обратитесь к нашей документации по [альтернативам с открытым исходным кодом](#open-source-alternatives).

<a name="using-example-application"></a>
### Пример использования

Прежде чем углубляться в каждый аспект трансляции событий, давайте сделаем общий обзор на примере интернет-магазина.

Предположим, что в нашем приложении у нас есть страница, которая позволяет пользователям просматривать статус доставки своих заказов. Предположим также, что событие `OrderShipmentStatusUpdated` запускается, когда приложение обрабатывает обновление статуса доставки:

    use App\Events\OrderShipmentStatusUpdated;

    OrderShipmentStatusUpdated::dispatch($order);

<a name="the-shouldbroadcast-interface"></a>
#### Интерфейс `ShouldBroadcast`

Когда пользователь просматривает один из своих заказов, мы не хотим, чтобы ему приходилось обновлять страницу для просмотра статуса обновлений. Вместо этого мы хотим транслировать обновления в приложение по мере их создания. Итак, нам нужно пометить событие `OrderShipmentStatusUpdated` интерфейсом `ShouldBroadcast`. Это укажет Laravel транслировать событие при его запуске:

    <?php

    namespace App\Events;

    use App\Models\Order;
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class OrderShipmentStatusUpdated implements ShouldBroadcast
    {
        /**
         * Экземпляр заказа.
         *
         * @var \App\Order
         */
        public $order;
    }

Интерфейс `ShouldBroadcast` требует, чтобы в нашем классе события был определен метод `broadcastOn`. Этот метод отвечает за возврат каналов, по которым должно транслироваться событие. Пустая заглушка этого метода уже определена в сгенерированных классах событий, поэтому нам нужно только заполнить ее реализацию. Мы хотим, чтобы только создатель заказа мог просматривать статус обновления, поэтому мы будем транслировать событие на частном канале, привязанном к конкретному заказу:

    /**
     * Получить каналы трансляции события.
     *
     * @return \Illuminate\Broadcasting\PrivateChannel
     */
    public function broadcastOn()
    {
        return new PrivateChannel('orders.'.$this->order->id);
    }

<a name="example-application-authorizing-channels"></a>
#### Авторизация каналов

Помните, что пользователи должны иметь разрешение на прослушивание частных каналов. Мы можем определить наши правила авторизации каналов в файле `routes/channels.php` нашего приложения. В этом примере нам нужно убедиться, что любой пользователь, пытающийся прослушивать частный канал `orders.1`, на самом деле является создателем заказа:

    use App\Models\Order;

    Broadcast::channel('orders.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Метод `channel` принимает два аргумента: имя канала и замыкание, которое возвращает `true` или `false`, указывая тем самым, имеет ли пользователь право прослушивать канал.

Все замыкания авторизации получают текущего аутентифицированного пользователя в качестве своего первого аргумента и любые дополнительные параметры в качестве своих последующих аргументов. В этом примере мы используем заполнитель `{orderId}`, чтобы указать, что часть «ID» имени канала является параметром.

<a name="listening-for-event-broadcasts"></a>
#### Прослушивание трансляций событий

Далее все, что остается, – это прослушивать событие в нашем JavaScript-приложении. Мы можем сделать это с помощью [Laravel Echo](#client-side-installation). Во-первых, мы будем использовать метод `private` для подписки на частный канал. Затем мы можем использовать метод `listen` для прослушивания события `OrderShipmentStatusUpdated`. По умолчанию все публичные свойства события будут включены в трансляцию события:

```js
Echo.private(`orders.${orderId}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order);
    });
```

<a name="defining-broadcast-events"></a>
## Определение транслируемых событий

Чтобы сообщить Laravel, что какое-то событие должно транслироваться, вы должны реализовать интерфейс `Illuminate\Contracts\Broadcasting\ShouldBroadcast` в классе события. Этот интерфейс уже импортирован во все классы событий, сгенерированные фреймворком, поэтому вы с легкостью можете добавить его к любому из ваших событий.

Интерфейс `ShouldBroadcast` требует, чтобы вы реализовали единственный метод: `broadcastOn`. Метод `broadcastOn` должен возвращать канал или массив каналов, по которым должно транслироваться событие. Каналы должны быть экземплярами `Channel`, `PrivateChannel` или `PresenceChannel`. Экземпляры `Channel` представляют собой публичные каналы, на которые может подписаться любой пользователь, в то время как `PrivateChannels` и `PresenceChannels` представляют собой частные каналы, для которых требуется [авторизация канала](#authorizing-channels):

    <?php

    namespace App\Events;

    use App\Models\User;
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        /**
         * Пользователь, создавший сервер.
         *
         * @var \App\Models\User
         */
        public $user;

        /**
         * Создать новый экземпляр события.
         *
         * @param  \App\Models\User  $user
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Получить каналы трансляции события.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

После реализации интерфейса `ShouldBroadcast` вам нужно только [запустить событие](events.md), как обычно. После того, как событие будет  запущено, [задание в очереди](queues.md) автоматически транслирует событие, используя указанный вами драйвер трансляции.

<a name="broadcast-name"></a>
### Имя транслируемого события

По умолчанию Laravel будет транслировать событие, используя имя класса события. Однако вы можете изменить имя транслируемого события, определив для события метод `broadcastAs`:

    /**
     * Получить имя транслируемого события.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

Если вы измените имя транслируемого события с помощью метода `broadcastAs`, то вы должны убедиться, что зарегистрировали ваш слушатель с ведущим символом `.`. Это укажет Echo не добавлять пространство имен приложения к событию:

    .listen('.server.created', function (e) {
        ....
    });

<a name="broadcast-data"></a>
### Данные трансляции

При трансляции события, все его публичные свойства автоматически сериализуются и транслируются как полезная нагрузка события, что позволяет вам получить доступ к любым его публичным данным из вашего JavaScript-приложения. Так, например, если ваше событие имеет единственное публичное свойство `$user`, представляющее собой модель Eloquent, то полезная нагрузка при трансляции события будет:

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

Однако, если вы хотите иметь более точный контроль над полезной нагрузкой трансляции, то вы можете определить метод `broadcastWith` вашего события. Этот метод должен возвращать массив данных, которые вы хотите использовать в качестве полезной нагрузки при трансляции события:

    /**
     * Получите данные для трансляции.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### Очередь трансляции

По умолчанию каждое транслируемое событие помещается в очередь по умолчанию и соединение очереди по умолчанию, указанные в вашем конфигурационном файле `config/queue.php`. Вы можете изменить соединение очереди и имя, используемое вещателем, определив свойства `connection` и `queue` в вашем классе события:

    /**
     * Имя соединения очереди, которое будет использоваться при трансляции события.
     *
     * @var string
     */
    public $connection = 'redis';

    /**
     * Имя очереди, в которую нужно поместить задание трансляции.
     *
     * @var string
     */
    public $queue = 'default';

Кроме того, вы можете изменить имя очереди, определив метод `broadcastQueue` для вашего события:

    /**
     * Имя очереди, в которую нужно поместить задание трансляции.
     *
     * @return string
     */
    public function broadcastQueue()
    {
        return 'default';
    }

Если вы хотите транслировать свое событие с помощью очереди `sync` вместо драйвера очереди по умолчанию, то вы можете реализовать интерфейс `ShouldBroadcastNow` вместо `ShouldBroadcast`:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class OrderShipmentStatusUpdated implements ShouldBroadcastNow
    {
        //
    }

<a name="broadcast-conditions"></a>
### Условия трансляции

Иногда необходимо транслировать событие только в том случае, если выполняется определенное условие. Вы можете определить эти условия, добавив метод `broadcastWhen` в ваш класс события:

    /**
     * Определить, условия трансляции события.
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->order->value > 100;
    }

<a name="broadcasting-and-database-transactions"></a>
#### Трансляция и транзакции базы данных

Когда транслируемые события отправляются в транзакциях базы данных, они могут быть обработаны очередью до того, как транзакция базы данных будет зафиксирована. Когда это происходит, любые обновления, внесенные вами в модели или записи базы данных во время транзакции базы данных, могут еще не быть отражены в базе данных. Кроме того, любые модели или записи базы данных, созданные в рамках транзакции, могут не существовать в базе данных. Если ваше событие зависит от этих моделей, могут возникнуть непредвиденные ошибки при обработке задания, транслирующего событие.

Если для параметра `after_commit` конфигурации вашего соединения с очередью установлено значение `false`, то вы все равно можете указать, что конкретное транслируемое событие должно быть отправлено после того, как все открытые транзакции базы данных были зафиксированы, путем определения свойства `$afterCommit` в классе события:

    <?php

    namespace App\Events;

    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $afterCommit = true;
    }

> {tip} Чтобы узнать больше о том, как обойти эти проблемы, просмотрите документацию, касающуюся [заданий в очереди и транзакций базы данных](queues.md#jobs-and-database-transactions).

<a name="authorizing-channels"></a>
## Авторизация каналов

Частные каналы требуют, чтобы текущий аутентифицированный пользователь был авторизован и действительно мог прослушивать канал. Это достигается путем отправки HTTP-запроса вашему приложению Laravel с именем канала, что позволит вашему приложению определить, может ли пользователь прослушивать этот канал. При использовании [Laravel Echo](#client-side-installation) HTTP-запрос на авторизацию подписок на частные каналы будет выполнен автоматически; однако вам необходимо определить верные маршруты для ответа на эти запросы.

<a name="defining-authorization-routes"></a>
### Определение маршрутов авторизации

Laravel упрощает определение маршрутов для ответа на запросы об авторизации канала. В поставщике `App\Providers\BroadcastServiceProvider` вы увидите вызов метода `Broadcast::routes`. Этот метод зарегистрирует маршрут `/broadcasting/auth` для обработки запросов авторизации:

    Broadcast::routes();

Метод `Broadcast::routes` автоматически применит посредника `web` для [группы своих маршрутов](routing.md#route-groups); однако вы можете передать в метод массив атрибутов маршрута, если хотите изменить присваиваемые им атрибуты:

    Broadcast::routes($attributes);

<a name="customizing-the-authorization-endpoint"></a>
#### Изменение конечной точки авторизации

По умолчанию Echo будет использовать конечную точку `/broadcasting/auth` для авторизации доступа к каналу. Однако вы можете указать свою собственную конечную точку авторизации, передав параметр конфигурации `authEndpoint` вашему экземпляру Echo:

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    authEndpoint: '/custom/endpoint/auth'
});
```

<a name="customizing-the-authorization-request"></a>
#### Настройка запроса на авторизацию

Вы можете изменить выполнение запросов авторизации Laravel Echo при его инициализации, предоставив параметр `authorizer`:

```js
window.Echo = new Echo({
    // ...
    authorizer: (channel, options) => {
        return {
            authorize: (socketId, callback) => {
                axios.post('/api/broadcasting/auth', {
                    socket_id: socketId,
                    channel_name: channel.name
                })
                .then(response => {
                    callback(false, response.data);
                })
                .catch(error => {
                    callback(true, error);
                });
            }
        };
    },
})
```

<a name="defining-authorization-callbacks"></a>
### Определение авторизации канала

Затем нам нужно определить логику, которая фактически будет определять, может ли текущий аутентифицированный пользователь прослушивать указанный канал. Это делается в файле `routes/channels.php`, находящемся в вашем приложении. В этом файле вы можете использовать метод `Broadcast::channel` для регистрации замыканий авторизации канала:

    Broadcast::channel('orders.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Метод `channel` принимает два аргумента: имя канала и замыкание, которое возвращает `true` или `false`, указывая тем самым, имеет ли пользователь право прослушивать канал.

Все замыкания авторизации получают текущего аутентифицированного пользователя в качестве своего первого аргумента и любые дополнительные параметры в качестве своих последующих аргументов. В этом примере мы используем заполнитель `{orderId}`, чтобы указать, что часть «ID» имени канала является параметром.

<a name="authorization-callback-model-binding"></a>
#### Привязка модели к авторизации

Как и HTTP-маршруты, для маршрутов каналов также могут использоваться неявные и явные [привязки модели к маршруту](routing.md#route-model-binding). Например, вместо получения строкового или числового идентификатора заказа вы можете запросить фактический экземпляр модели `Order`:

    use App\Models\Order;

    Broadcast::channel('orders.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

> {note} В отличие от привязки модели к HTTP-маршруту, привязка модели канала не поддерживает [ограничение неявной привязки модели](routing.md#implicit-model-binding-scoping). Однако это редко представляет собой проблему, потому что большинство каналов можно ограничить на основе уникального первичного ключа одной модели.

<a name="authorization-callback-authentication"></a>
#### Предварительная аутентификация авторизации канала

Частные каналы и каналы присутствия аутентифицируют текущего пользователя через стандартного охранника аутентификации вашего приложения. Если пользователь не аутентифицирован, то авторизация канала автоматически отклоняется, и обратный вызов авторизации никогда не выполняется. Однако вы можете назначить несколько своих охранников, которые должны при необходимости аутентифицировать входящий запрос:

    Broadcast::channel('channel', function () {
        // ...
    }, ['guards' => ['web', 'admin']]);

<a name="defining-channel-classes"></a>
### Определение класса канала

Если ваше приложение использует много разных каналов, то ваш файл `routes/channels.php` может стать громоздким. Таким образом, вместо использования замыканий для авторизации каналов вы можете использовать классы каналов. Чтобы сгенерировать новый канал, используйте команду `make:channel` [Artisan](artisan.md). Эта команда поместит новый класс канала в каталог `app/Broadcasting` вашего приложения:

```shell
php artisan make:channel OrderChannel
```

Затем зарегистрируйте свой канал в файле `routes/channels.php`:

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('orders.{order}', OrderChannel::class);

Наконец, вы можете поместить логику авторизации для своего канала в метод `join` класса канала. Этот метод будет содержать ту же логику, которую вы обычно использовали бы в замыкании  при авторизации вашего канала. Вы также можете воспользоваться преимуществами привязки модели канала:

    <?php

    namespace App\Broadcasting;

    use App\Models\Order;
    use App\Models\User;

    class OrderChannel
    {
        /**
         * Создать новый экземпляр канала.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Подтвердить доступ пользователя к каналу.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

> {tip} Как и многие другие классы в Laravel, классы каналов будут автоматически разрешены [контейнером служб](container.md). Таким образом, вы можете указать любые зависимости, необходимые для вашего канала, в его конструкторе.

<a name="broadcasting-events"></a>
## Трансляция событий

После того как вы определили событие и отметили его интерфейсом `ShouldBroadcast`, вам нужно только запустить событие, используя метод отправки события. Диспетчер событий заметит, что событие помечено интерфейсом `ShouldBroadcast`, и поставит событие в очередь для дальнейшей трансляции:

    use App\Events\OrderShipmentStatusUpdated;

    OrderShipmentStatusUpdated::dispatch($order));

<a name="only-to-others"></a>
### Трансляция событий только остальным пользователям

При создании приложения, использующего трансляцию событий, иногда требуется трансляция события всем подписчикам канала, кроме текущего пользователя. Вы можете сделать это с помощью помощника `broadcast` и метода `toOthers`:

    use App\Events\OrderShipmentStatusUpdated;

    broadcast(new OrderShipmentStatusUpdated($update))->toOthers();

Чтобы лучше понять необходимость использования метода `toOthers`, давайте представим приложение со списком задач, в котором пользователь может создать новую задачу, введя имя задачи. Чтобы создать задачу, ваше приложение может сделать запрос к URL-адресу `/task`, который транслирует создание задачи и возвращает JSON-представление новой задачи. Когда ваше JavaScript-приложение получает ответ от конечной точки, оно может напрямую вставить новую задачу в свой список задач следующим образом:

```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

Однако помните, что мы также транслируем создание задачи. Если ваше JavaScript-приложение также прослушивает это событие, чтобы добавить задачи в список задач, у вас будут дублирующиеся задачи в вашем списке: одна из конечной точки и одна из трансляции. Вы можете решить эту проблему, используя метод `toOthers`, чтобы указать вещателю не транслировать событие текущему пользователю.

> {note} Ваше событие должно использовать трейт `Illuminate\Broadcasting\InteractsWithSockets` для вызова метода `toOthers`.

<a name="only-to-others-configuration"></a>
#### Конфигурирование при использовании метода `toOthers`

Когда вы инициализируете экземпляр Laravel Echo, соединению назначается идентификатор сокета. Если вы используете глобальный экземпляр [Axios](https://github.com/mzabriskie/axios) для выполнения HTTP-запросов из вашего JavaScript-приложения, то идентификатор сокета будет автоматически прикрепляться к каждому исходящему запросу в заголовке `X-Socket-ID`. Затем, когда вы вызываете метод `toOthers`, Laravel извлечет идентификатор сокета из заголовка и укажет вещателю не транслировать никакие соединения с этим идентификатором сокета.

Если вы не используете глобальный экземпляр Axios, то вам необходимо вручную сконфигурировать JavaScript-приложение для отправки заголовка `X-Socket-ID` со всеми исходящими запросами. Вы можете получить идентификатор сокета, используя метод `Echo.socketId`:

```js
var socketId = Echo.socketId();
```

<a name="customizing-the-connection"></a>
### Настройка соединения

Если ваше приложение взаимодействует с несколькими соединениями трансляции, и вы хотите транслировать событие через собственный вещатель, то, используя метод `via`, вы можете указать соединение для отправки событий:

    use App\Events\OrderShipmentStatusUpdated;

    broadcast(new OrderShipmentStatusUpdated($update))->via('pusher');

В качестве альтернативы вы можете указать соединение транслируемого события, вызвав метод `broadcastVia` в конструкторе события. Однако перед этим вы должны убедиться, что класс события использует трейт `InteractsWithBroadcasting`:

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithBroadcasting;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class OrderShipmentStatusUpdated implements ShouldBroadcast
    {
        use InteractsWithBroadcasting;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->broadcastVia('pusher');
        }
    }

<a name="receiving-broadcasts"></a>
## Прием трансляций

<a name="listening-for-events"></a>
### Прослушивание событий

После того, как вы [установили и создали экземпляр Laravel Echo](#client-side-installation), вы готовы к прослушиванию событий, которые транслируются из вашего приложения Laravel. Сначала используйте метод `channel` для получения экземпляра канала, затем вызовите метод `listen` для прослушивания конкретного события:

```js
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

Если вы хотите прослушивать события на частном канале, то используйте вместо этого метод `private`. Вы можете продолжить цепочку вызовов метода `listen` для прослушивания нескольких событий на одном канале:

```js
Echo.private(`orders.${this.order.id}`)
    .listen(/* ... */)
    .listen(/* ... */)
    .listen(/* ... */);
```

<a name="stop-listening-for-events"></a>
#### Прекращение прослушивание событий

Если вы хотите прекратить прослушивание конкретного события, не [покидая канал](#leaving-a-channel), то используйте метод `stopListening`:

```js
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated')
```

<a name="leaving-a-channel"></a>
### Покидание канала

Чтобы покинуть канал, вы можете вызвать метод `leaveChannel` вашего экземпляра Echo:

```js
Echo.leaveChannel(`orders.${this.order.id}`);
```

Если вы хотите покинуть канал, а также связанные с ним частные каналы и каналы присутствия, вы можете вызвать метод `leave`:

```js
Echo.leave(`orders.${this.order.id}`);
```
<a name="namespaces"></a>
### Пространства имён

Вы могли заметить в приведенных выше примерах, что мы не указали полное пространство имен `App\Events` для классов событий. Это связано с тем, что Echo автоматически предполагает, что события находятся в пространстве имен `App\Events`. Однако вы можете изменить корневое пространство имен при создании экземпляра Echo, передав параметр конфигурации `namespace`:

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    namespace: 'App.Other.Namespace'
});
```

В качестве альтернативы вы можете добавить к классам событий префикс `.` при подписке на них с помощью Echo. Это позволит вам всегда указывать полное имя класса:

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        //
    });
```

<a name="presence-channels"></a>
## Каналы присутствия

Каналы присутствия основаны на безопасности частных каналов, но в то же время предоставляя дополнительную функцию осведомленности о том, кто подписан на канал. Это упрощает создание мощных функций приложения для совместной работы, таких как уведомление пользователей, когда другой пользователь просматривает ту же страницу, или перечисление пользователей комнаты чата.

<a name="authorizing-presence-channels"></a>
### Авторизация каналов присутствия

Все каналы присутствия также являются частными; следовательно, пользователи должны быть [авторизованы для доступа к ним](#authorizing-channels). Однако при определении замыканий авторизации для каналов присутствия вы не должны возвращать `true`, если пользователь авторизован для присоединения к каналу. Вместо этого вы должны вернуть массив данных о пользователе.

Данные, возвращаемые замыканием авторизации, будут доступны для слушателей событий канала присутствия в вашем JavaScript-приложении. Если пользователь не авторизован для присоединения к каналу присутствия, то вы должны вернуть `false` или `null`:

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### Присоединение к каналам присутствия

Чтобы присоединиться к каналу присутствия, вы можете использовать метод `join` Echo. Метод `join` вернет реализацию `PresenceChannel`, которая, наряду с методом `listen`, позволяет вам подписаться на события `here`, `joining` и `leave`.

```js
Echo.join(`chat.${roomId}`)
    .here((users) => {
        //
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    })
    .error((error) => {
        console.error(error);
    });
```

Замыкание `here` будет выполнено сразу после успешного присоединения к каналу и получит массив, содержащий информацию о пользователе для всех других пользователей, которые в настоящее время подписаны на канал. Метод `joining` будет выполнен, когда новый пользователь присоединится к каналу, а метод `leaving` будет выполнен при покидании пользователем канала. Метод `error` будет выполнен, когда конечная точка аутентификации вернет код состояния HTTP, отличный от 200, или если возникнет проблема с синтаксическим анализом возвращенного JSON.

<a name="broadcasting-to-presence-channels"></a>
### Трансляция на каналы присутствия

Каналы присутствия могут получать события так же, как публичные или частные каналы. Используя пример чата, мы можем захотеть транслировать события `NewMessage` на канал присутствия комнаты. Для этого мы вернем экземпляр `PresenceChannel` из метода `broadcastOn` события:

    /**
     * Получить каналы трансляции события.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

Как и в случае с другими событиями, вы можете использовать помощник `broadcast` и метод `toOthers`, чтобы исключить текущего пользователя из приема трансляции:

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

Как и для других типов событий, вы можете прослушивать события, отправленные в каналы присутствия, используя метод `listen` Echo:

```js
Echo.join(`chat.${roomId}`)
    .here(/* ... */)
    .joining(/* ... */)
    .leaving(/* ... */)
    .listen('NewMessage', (e) => {
        //
    });
```

<a name="model-broadcasting"></a>
## Трансляция событий модели

> {note} Прежде чем читать следующую документацию о трансляции событий модели, мы рекомендуем вам ознакомиться с общими концепциями служб трансляции событий модели Laravel, а также с тем, как самостоятельно создавать и прослушивать трансляции событий.

Обычно транслируются события, когда [модели Eloquent](eloquent.md) вашего приложения создаются, обновляются или удаляются. Конечно, это можно легко сделать, [определив пользовательские события изменений состояния модели Eloquent](eloquent.md#events) и применив интерфейс `ShouldBroadcast` к этим событиям.

Однако, если вы не используете эти события для каких-либо других целей в своем приложении, создание классов событий с единственной целью их дальнейшей трансляции может оказаться обременительным. Чтобы исправить это, Laravel позволяет вам указать, что модель Eloquent должна автоматически транслировать изменения своего состояния.

Для начала ваша модель Eloquent должна использовать трейт `Illuminate\Database\Eloquent\BroadcastsEvents`. Кроме того, модель должна определять метод `broadcastOn`, который будет возвращать массив каналов, по которым должны транслироваться события модели:

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Database\Eloquent\BroadcastsEvents;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use BroadcastsEvents, HasFactory;

    /**
     * Получить автора поста.
     */
    public function user()
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Получить каналы трансляции событий модели.
     *
     * @param  string  $event
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn($event)
    {
        return [$this, $this->user];
    }
}
```

Как только будет включен этот трейт и определены каналы вещания, модель начнет автоматически транслировать события при создании, обновлении, удалении, программном удалении / восстановлении экземпляра модели.

Кроме того, вы могли заметить, что метод `broadcastOn` получает строковый аргумент `$event`. Этот аргумент содержит тип события, которое произошло в модели, и может иметь одно из значений: `created`, `updated`, `deleted`, `trashed` или `restored`. Проверяя значение этой переменной, вы можете определить, на какие каналы (имеющиеся) модель должна транслировать конкретное событие:

```php
/**
 * Получить каналы трансляции событий модели.
 *
 * @param  string  $event
 * @return \Illuminate\Broadcasting\Channel|array
 */
public function broadcastOn($event)
{
    return match ($event) {
        'deleted' => [],
        default => [$this, $this->user],
    };
}
```

<a name="customizing-model-broadcasting-event-creation"></a>
#### Настройка создания транслируемого события модели

По желанию можно настроить, как Laravel должен создавать транслируемое событие модели. Вы можете добиться этого, определив метод `newBroadcastableEvent` в вашей модели Eloquent. Этот метод должен возвращать экземпляр `Illuminate\Database\Eloquent\BroadcastableModelEventOccurred`:

```php
use Illuminate\Database\Eloquent\BroadcastableModelEventOccurred;

/**
 * Создать новое транслируемое событие модели.
 *
 * @param  string  $event
 * @return \Illuminate\Database\Eloquent\BroadcastableModelEventOccurred
 */
protected function newBroadcastableEvent($event)
{
    return (new BroadcastableModelEventOccurred(
        $this, $event
    ))->dontBroadcastToCurrentUser();
}
```

<a name="model-broadcasting-conventions"></a>
### Соглашения трансляции событий модели

<a name="model-broadcasting-channel-conventions"></a>
#### Соглашения о каналах

Как вы могли заметить, метод `broadcastOn` в приведенном выше примере модели не возвращал экземпляры `Channel`. Вместо этого модели Eloquent были возвращены напрямую. Если экземпляр модели Eloquent возвращается методом `broadcastOn` вашей модели (или содержится в массиве, возвращаемом методом), Laravel автоматически создаст экземпляр частного канала для модели, используя имя класса модели и идентификатор первичного ключа в качестве имени канала.

Итак, модель `App\Models\User` с идентификатором `1` будет преобразована в экземпляр `Illuminate\Broadcasting\PrivateChannel` с именем `App.Models.User.1`. Конечно, в дополнение к возврату экземпляров модели Eloquent из метода `broadcastOn` вашей модели, вы можете возвращать полноценные экземпляры `Channel`, чтобы иметь полный контроль над именами каналов модели:

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Получить каналы трансляции событий модели.
 *
 * @param  string  $event
 * @return \Illuminate\Broadcasting\Channel|array
 */
public function broadcastOn($event)
{
    return [new PrivateChannel('user.'.$this->id)];
}
```

Если вы планируете явно возвращать экземпляр канала из метода `broadcastOn` вашей модели, вы можете передать экземпляр модели Eloquent в конструктор канала. При этом Laravel будет использовать описанные выше соглашения о каналах модели, чтобы преобразовать модель Eloquent в строку имени канала:

```php
return [new Channel($this->user)];
```

Если вам нужно определить имя канала модели, вы можете вызвать метод `broadcastChannel` любого экземпляра модели. Например, этот метод возвращает строку `App.Models.User.1` для модели `App\Models\User` с `id` равным `1`:

```php
$user->broadcastChannel()
```

<a name="model-broadcasting-event-conventions"></a>
#### Соглашения о событиях

Поскольку трансляция событий модели не связаны с «фактическим» событием в каталоге `App\Events` вашего приложения, им присваивается имя и полезная нагрузка на основе соглашений. Соглашение Laravel состоит в том, чтобы транслировать событие, используя имя класса модели (не включая пространство имен) и имя события модели, которое инициировало трансляцию.

Так, например, обновление модели `App\Models\Post` будет транслировать событие в ваше клиентское приложение как `PostUpdated` со следующей полезной нагрузкой:

```json
{
    "model": {
        "id": 1,
        "title": "My first post"
        ...
    },
    ...
    "socket": "someSocketId",
}
```

Удаление модели `App\Models\User` будет транслировано событием с именем `UserDeleted`.

При желании вы можете определить собственные имя и полезную нагрузку трансляции, добавив к вашей модели методы `broadcastAs` и `broadcastWith`. Эти методы получают имя происходящего события / операции модели, что позволяет настроить имя события и полезную нагрузку для каждой операции модели. Если из метода `broadcastAs` возвращается `null`, то Laravel будет использовать соглашения об именах транслируемых событий модели, обсужденные выше, при трансляции события:

```php
/**
 * Получить имя транслируемого события модели.
 *
 * @param  string  $event
 * @return string|null
 */
public function broadcastAs($event)
{
    return match ($event) {
        'created' => 'post.created',
        default => null,
    };
}

/**
 * Получить данные транслируемого события модели.
 *
 * @param  string  $event
 * @return array
 */
public function broadcastWith($event)
{
    return match ($event) {
        'created' => ['title' => $this->title],
        default => ['model' => $this],
    };
}
```

<a name="listening-for-model-broadcasts"></a>
### Прослушивание транслируемых событий модели

После того как вы добавили в модель трейт `BroadcastsEvents` и определили метод `broadcastOn` модели, вы готовы начать прослушивание транслируемых событий модели в своем клиентском приложении. Перед тем, как начать, вы можете ознакомиться с полной документацией по [прослушиванию событий](#listening-for-events).

Сначала используйте метод `private` для получения экземпляра канала, затем вызовите метод `listen` для прослушивания указанного события. Как правило, имя канала, присвоенное методу `private`, должно соответствовать [соглашениям о трансляции событий модели](#model-broadcasting-conventions) Laravel.

Как только вы получили экземпляр канала, вы можете использовать метод `listen` для прослушивания определенного события. Поскольку трансляция событий модели не связаны с «фактическим» событием в каталоге вашего приложения `App\Events`, [имя события](#model-broadcasting-event-conventions) должно иметь префикс `.`, указывающий, что событие не привязано к пространству имен. Каждое транслируемое событие модели имеет свойство `model`, которое содержит все транслируемые свойства модели:

```js
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.PostUpdated', (e) => {
        console.log(e.model);
    });
```

<a name="client-events"></a>
## Клиентские события

> {tip} При использовании [Pusher Channels](https://pusher.com/channels) вы должны включить опцию «Client Events» в разделе «App Settings» вашей [панели управления приложения](https://dashboard.pusher.com/) для отправки клиентских событий.

По желанию можно транслировать событие другим подключенным клиентам, вообще не затрагивая ваше приложение Laravel. Это может быть особенно полезно для таких вещей, как «ввод» уведомлений, когда вы хотите предупредить пользователей вашего приложения о том, что другой пользователь печатает сообщение.

Чтобы транслировать клиентские события, вы можете использовать метод `whisper` Echo:

```js
Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        name: this.user.name
    });
```

Чтобы прослушивать клиентские события, вы можете использовать метод `listenForWhisper`:

```js
Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

<a name="notifications"></a>
## Уведомления

Если связать трансляцию событий с [уведомлениями](notifications.md), то ваше JavaScript-приложение может получать новые уведомления по мере их появления без необходимости в обновлении страницы. Перед началом работы обязательно прочтите документацию по использованию [канала транслируемых уведомлений](notifications.md#broadcast-notifications).

После того, как вы настроили уведомление для использования трансляции канала, вы можете прослушивать транслируемые события, используя метод `notification` Echo. Помните, что имя канала должно соответствовать имени класса объекта, получающего уведомления:

```js
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

В этом примере все уведомления, отправленные экземплярам `App\Models\User` через канал `broadcast`, будут получены в замыкании. Авторизация канала `App.Models.User.{id}` уже изначально содержится в `BroadcastServiceProvider` фреймворка Laravel.
