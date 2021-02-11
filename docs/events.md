# Laravel 8.x — События

- [Введение](#introduction)
- [Регистрация событий и слушателей](#registering-events-and-listeners)
    - [Генерация событий и слушателей](#generating-events-and-listeners)
    - [Ручная регистрация событий](#manually-registering-events)
    - [Автообнаружение событий](#event-discovery)
- [Определение событий](#defining-events)
- [Определение слушателей](#defining-listeners)
- [Слушатели событий в очереди](#queued-event-listeners)
    - [Взаимодействие с очередью вручную](#manually-interacting-the-queue)
    - [Слушатели событий в очереди и транзакции базы данных](#queued-event-listeners-and-database-transactions)
    - [Обработка невыполненных заданий](#handling-failed-jobs)
- [Отправка событий](#dispatching-events)
- [Подписчики событий](#event-subscribers)
    - [Написание подписчиков на события](#writing-event-subscribers)
    - [Регистрация подписчиков на события](#registering-event-subscribers)

<a name="introduction"></a>
## Введение

События Laravel обеспечивают простую реализацию шаблона Наблюдатель, позволяя вам подписываться и отслеживать различные события, происходящие в вашем приложении. Классы событий обычно хранятся в каталоге `app/Events`, а их слушатели – в `app/Listeners`. Не беспокойтесь, если вы не видите эти каталоги в своем приложении, так как они будут созданы для вас, когда вы будете генерировать события и слушатели с помощью команд консоли Artisan.

События служат отличным способом разделения различных аспектов вашего приложения, поскольку одно событие может иметь несколько слушателей, которые не зависят друг от друга. Например, вы можете захотеть отправлять уведомление Slack своему пользователю каждый раз, когда заказ будет отправлен. Вместо того, чтобы связывать код обработки заказа с кодом уведомления Slack, вы можете вызвать событие `App\Events\OrderShipped`, которое слушатель может получить и использовать для отправки уведомления Slack.

<a name="registering-events-and-listeners"></a>
## Регистрация событий и слушателей

Поставщик `App\Providers\EventServiceProvider` Laravel, предоставляет удобное место для регистрации всех слушателей событий вашего приложения. Свойство `$listen` содержит массив всех событий (ключей) и их слушателей (значений). Вы можете добавить в этот массив столько событий, сколько требуется вашему приложению. Например, добавим событие `OrderShipped`:

    use App\Events\OrderShipped;
    use App\Listeners\SendShipmentNotification;

    /**
     * Карта слушателей событий приложения.
     *
     * @var array
     */
    protected $listen = [
        OrderShipped::class => [
            SendShipmentNotification::class,
        ],
    ];

> {tip} Команда `event:list` может использоваться для отображения списка всех событий и слушателей, зарегистрированных вашим приложением.

<a name="generating-events-and-listeners"></a>
### Генерация событий и слушателей

Конечно, вручную создавать файлы для каждого события и слушателя сложно. Вместо этого добавьте события и слушателей в свой `EventServiceProvider` и используйте команду `event:generate` Artisan. Эта команда будет генерировать любые события или слушатели, перечисленные в вашем `EventServiceProvider`, но которые еще не существуют:

    php artisan event:generate

В качестве альтернативы вы можете использовать команды `make:event` и `make:listener` Artisan для генерации отдельных событий и слушателей:

    php artisan make:event PodcastProcessed

    php artisan make:listener SendPodcastNotification --event=PodcastProcessed

<a name="manually-registering-events"></a>
### Ручная регистрация событий

Обычно события должны регистрироваться через массив `$listen` поставщика `EventServiceProvider`; однако, вы также можете зарегистрировать слушателей событий на основе классов или замыканий вручную в методе `boot` вашего `EventServiceProvider`:

    use App\Events\PodcastProcessed;
    use App\Listeners\SendPodcastNotification;
    use Illuminate\Support\Facades\Event;

    /**
     * Регистрация любых событий вашего приложения.
     *
     * @return void
     */
    public function boot()
    {
        Event::listen(
            PodcastProcessed::class,
            [SendPodcastNotification::class, 'handle']
        );

        Event::listen(function (PodcastProcessed $event) {
            //
        });
    }

<a name="queuable-anonymous-event-listeners"></a>
#### Слушатели анонимных событий в очереди

При регистрации слушателей событий на основе замыкания вручную вы можете передать замыкание слушателя в функцию `Illuminate\Events\queueable`, чтобы дать Laravel команду выполнить слушатель с использованием [очереди](queues.md):

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;

    /**
     * Регистрация любых событий вашего приложения.
     *
     * @return void
     */
    public function boot()
    {
        Event::listen(queueable(function (PodcastProcessed $event) {
            //
        }));
    }

Как и задания в очереди, вы можете использовать методы `onConnection`, `onQueue` и `delay` для настройки выполнения слушателя в очереди:

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));

Если вы хотите обрабатывать сбои анонимного слушателя в очереди, то вы можете передать замыкание методу `catch` при определении слушателя `queueable`. Это замыкание получит экземпляр события и экземпляр `Throwable`, который вызвал сбой слушателя:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // Событие в очереди завершилось неудачно ...
    }));

<a name="wildcard-event-listeners"></a>
#### Wildcard Event Listeners

Вы даже можете зарегистрировать слушателей, используя `*` в качестве параметра подстановочного знака, что позволит вам перехватывать несколько событий на одном и том же слушателе. Слушатели подстановочных знаков получают имя события в качестве первого аргумента и весь массив данных события в качестве второго аргумента:

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="event-discovery"></a>
### Автообнаружение событий

Вместо того, чтобы вручную регистрировать события и слушателей в массиве `$listen` поставщика `EventServiceProvider`, вы можете включить автоматическое обнаружение событий. Когда обнаружение событий включено, Laravel автоматически найдет и зарегистрирует ваши события и слушателей, сканируя каталог `Listeners` вашего приложения. Кроме того, любые явно определенные события, перечисленные в `EventServiceProvider`, по-прежнему будут зарегистрированы.

Laravel находит слушателей событий, сканируя классы слушателей с помощью рефлексии PHP. Когда Laravel находит какой-либо метод класса слушателя, который начинается с `handle`, Laravel зарегистрирует эти методы как слушатели событий для события, тип которого указан в сигнатуре метода:

    use App\Events\PodcastProcessed;

    class SendPodcastNotification
    {
        /**
         * Обработать переданное событие.
         *
         * @param  \App\Events\PodcastProcessed
         * @return void
         */
        public function handle(PodcastProcessed $event)
        {
            //
        }
    }

Обнаружение событий по умолчанию отключено, но вы можете включить его, переопределив метод `shouldDiscoverEvents` вашего `EventServiceProvider`:

    /**
     * Определить, должны ли автоматически обнаруживаться события и слушатели.
     *
     * @return bool
     */
    public function shouldDiscoverEvents()
    {
        return true;
    }

По умолчанию все слушатели в каталоге вашего приложения `app/Listeners` будут сканироваться. Если вы хотите определить дополнительные каталоги для сканирования, вы можете переопределить метод `discoverEventsWithin` в своем `EventServiceProvider`:

    /**
     * Получить каталоги слушателей, которые следует использовать для обнаружения событий.
     *
     * @return array
     */
    protected function discoverEventsWithin()
    {
        return [
            $this->app->path('Listeners'),
        ];
    }

<a name="event-discovery-in-production"></a>
#### Кеширование событий

В эксплуатационном окружении для фреймворка неэффективно сканировать всех ваших слушателей по каждому запросу. Следовательно, в процессе развертывания вы должны запустить команду `event:cache` Artisan, чтобы кешировать манифест всех событий и слушателей вашего приложения. Этот манифест будет использоваться платформой для ускорения процесса регистрации события. Команда `event:clear` может использоваться для уничтожения кеша.

<a name="defining-events"></a>
## Определение событий

Класс событий – это, по сути, контейнер данных, который содержит информацию, относящуюся к событию. Например, предположим, что событие `App\Events\OrderShipped` получает объект [Eloquent ORM](eloquent.md):

    <?php

    namespace App\Events;

    use App\Models\Order;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Foundation\Events\Dispatchable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use Dispatchable, InteractsWithSockets, SerializesModels;

        /**
         * Экземпляр заказа.
         *
         * @var \App\Models\Order
         */
        public $order;

        /**
         * Создать новый экземпляр события.
         *
         * @param  \App\Models\Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

Как видите, в этом классе событий нет логики. Это контейнер для экземпляра `App\Models\Order` заказа, который был выполнен. Трейт `SerializesModels`, используемый событием, будет изящно сериализовать любые модели Eloquent, если объект события сериализуется с использованием функции `serialize` PHP, например, при использовании [слушателей в очереди](#queued-event-listeners).

<a name="defining-listeners"></a>
## Определение слушателей

Затем, давайте посмотрим на слушателя для нашего примера события. Слушатели событий получают экземпляры событий в своем методе `handle`. Команды `event:generate` и` make:listener` Artisan автоматически импортируют соответствующий класс события и внедрят событие в методе `handle`. В методе `handle` вы можете выполнять любые действия, необходимые для ответа на событие:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Создать слушателя событий.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Обработать событие.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Доступ к заказу с помощью `$event->order` ...
        }
    }

> {tip} В конструкторе ваших слушателей событий могут быть объявлены любые типы зависимостей, которые им нужны. Все слушатели событий извлекаются через [контейнер службы](container.md) Laravel, поэтому зависимости будут внедрены автоматически.

<a name="stopping-the-propagation-of-an-event"></a>
#### Остановка распространения события

Иногда вы можете захотеть остановить распространение события среди других слушателей. Вы можете сделать это, вернув `false` из метода `handle` вашего слушателя.

<a name="queued-event-listeners"></a>
## Слушатели событий в очереди

Слушатели в очереди могут быть полезны, если ваш слушатель собирается выполнять медленную задачу, такую ​​как отправка электронной почты или выполнение HTTP-запроса. Перед использованием слушателей в очереди убедитесь, что вы [сконфигурировали свою очередь](queues.md) и запустили обработчик очереди на вашем сервере или в локальной среде разработки.

Чтобы указать, что слушатель должен быть поставлен в очередь, добавьте интерфейс `ShouldQueue` в класс слушателя. Слушатели, сгенерированные командами `event:generate` и `make:listener` Artisan, уже будут иметь этот интерфейс, импортируемый в текущее пространство имен, поэтому вы можете использовать его немедленно:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

Это все! Теперь, когда отправляется событие, обрабатываемое этим слушателем, слушатель автоматически ставится в очередь диспетчером событий с использованием [системы очередей](queues.md) Laravel. Если при выполнении слушателя в очереди не возникает никаких исключений, задание в очереди будет автоматически удалено после завершения обработки.

<a name="customizing-the-queue-connection-queue-name"></a>
#### Настройка соединения очереди и имени очереди

Если вы хотите настроить соединение очереди, имя очереди или время задержки очереди для слушателя событий, то вы можете определить свойства `$connection`, `$queue`, или `$delay` в своем классе слушателя:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * Имя соединения, на которое должно быть отправлено задание.
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * Имя очереди, в которую должно быть отправлено задание.
         *
         * @var string|null
         */
        public $queue = 'listeners';

        /**
         * Время (в секундах) до обработки задания.
         *
         * @var int
         */
        public $delay = 60;
    }

Если вы хотите определить очередь слушателя во время выполнения, вы можете определить метод `viaQueue` слушателя:

    /**
     * Получить имя очереди слушателя.
     *
     * @return string
     */
    public function viaQueue()
    {
        return 'listeners';
    }

<a name="conditionally-queueing-listeners"></a>
#### Условная отправка слушателей в очередь

Иногда вам может потребоваться определить, следует ли ставить слушателя в очередь на основе некоторых данных, доступных только во время выполнения. Для этого к слушателю может быть добавлен метод `shouldQueue`, чтобы определить, следует ли поставить слушателя в очередь<!--. Если метод `shouldQueue` возвращает `false`, слушатель не будет выполнен-->:

    <?php

    namespace App\Listeners;

    use App\Events\OrderCreated;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class RewardGiftCard implements ShouldQueue
    {
        /**
         * Наградить покупателя подарочной картой.
         *
         * @param  \App\Events\OrderCreated  $event
         * @return void
         */
        public function handle(OrderCreated $event)
        {
            //
        }

        /**
         * Определить, следует ли ставить слушателя в очередь.
         *
         * @param  \App\Events\OrderCreated  $event
         * @return bool
         */
        public function shouldQueue(OrderCreated $event)
        {
            return $event->order->subtotal >= 5000;
        }
    }

<a name="manually-interacting-the-queue"></a>
### Взаимодействие с очередью вручную

Если вам нужно вручную получить доступ к методам `delete` и `release` базового задания в очереди слушателя, вы можете сделать это с помощью трейта `Illuminate\Queue\InteractsWithQueue`. Этот трейт по умолчанию импортируется в сгенерированные слушатели и предоставляет доступ к этим методам:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Обработать событие.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="queued-event-listeners-and-database-transactions"></a>
### Слушатели событий в очереди и транзакции базы данных

Когда слушатели в очереди отправляются в транзакциях базы данных, они могут быть обработаны очередью до того, как транзакция базы данных будет зафиксирована. Когда это происходит, любые обновления, внесенные вами в модели или записи базы данных во время транзакции базы данных, могут еще не быть отражены в базе данных. Кроме того, любые модели или записи базы данных, созданные в рамках транзакции, могут не существовать в базе данных. Если ваш слушатель зависит от этих моделей, могут возникнуть непредвиденные ошибки при обработке задания, которое отправляет поставленный в очередь слушатель.

Если для параметра `after_commit` конфигурации вашего соединения с очередью установлено значение `false`, то вы все равно можете указать, что конкретный слушатель в очереди должен быть отправлен после того, как все открытые транзакции базы данных были зафиксированы, путем определения свойства `$afterCommit` в классе слушателя:

    <?php

    namespace App\Listeners;

    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        public $afterCommit = true;
    }

> {tip} Чтобы узнать больше о том, как обойти эти проблемы, просмотрите документацию, касающуюся [заданий в очереди и транзакций базы данных](queues.md#jobs-and-database-transactions).

<a name="handling-failed-jobs"></a>
### Обработка невыполненных заданий

Иногда ваши слушатели событий в очереди могут дать сбой. Если слушатель в очереди превышает максимальное количество попыток, определенное вашим обработчиком очереди, для вашего слушателя будет вызван метод `failed`. Метод `failed` получает экземпляр события и `Throwable`, вызвавший сбой:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Обработать событие.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            //
        }

        /**
         * Обработать провал задания.
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Throwable  $exception
         * @return void
         */
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="specifying-queued-listener-maximum-attempts"></a>
#### Указание максимального количества попыток слушателя в очереди

Если один из ваших слушателей в очереди обнаруживает ошибку, вы, вероятно, не хотите, чтобы он продолжал повторять попытки бесконечно. Таким образом, Laravel предоставляет различные способы указать, сколько раз и как долго может выполняться попытка прослушивания.

Вы можете определить свойство `$tries` в своем классе слушателя, чтобы указать, сколько раз можно попытаться выполнить слушатель, прежде чем он будет считаться неудачным:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Количество попыток слушателя в очереди.
         *
         * @var int
         */
        public $tries = 5;
    }

В качестве альтернативы определению того, сколько раз можно попытаться выполнить слушатель, прежде чем он потерпит неудачу, вы можете определить время, через которое слушатель больше не должен выполняться. Это позволяет попытаться выполнить прослушивание любое количество раз в течение заданного периода времени. Чтобы определить время, через которое больше не следует предпринимать попытки прослушивания, добавьте метод `retryUntil` в свой класс слушателя. Этот метод должен возвращать экземпляр `DateTime`:

    /**
     * Определить время, через которое слушатель должен отключиться.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addMinutes(5);
    }

<a name="dispatching-events"></a>
## Отправка событий

Чтобы отправить событие, вы можете вызвать статический метод `dispatch` события. Этот метод доступен в событии с помощью трейта `Illuminate\Foundation\Events\Dispatchable`. Любые аргументы, переданные методу `dispatch`, будут переданы конструктору события:

    <?php

    namespace App\Http\Controllers;

    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;
    use App\Models\Order;
    use Illuminate\Http\Request;

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

            // Логика отправки заказа ...

            OrderShipped::dispatch($order);
        }
    }

> {tip} При тестировании может быть полезно утверждать, что определенные события были отправлены без фактического запуска их слушателей. [Встроенные помощники по тестированию](mocking.md#event-fake) Laravel делает его легко.

<a name="event-subscribers"></a>
## Подписчики событий

<a name="writing-event-subscribers"></a>
### Написание подписчиков на события

Подписчики событий – это классы, которые могут подписываться на несколько событий из самого класса подписчика, что позволяет вам определять несколько обработчиков событий в одном классе. Подписчики должны определить метод `subscribe`, которому будет передан экземпляр диспетчера событий. Вы можете вызвать метод `listen` у данного диспетчера для регистрации слушателей событий:

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * Обработать событие входа пользователя в систему.
         */
        public function handleUserLogin($event) {}

        /**
         * Обработать событие выхода пользователя из системы.
         */
        public function handleUserLogout($event) {}

        /**
         * Зарегистрировать слушателей для подписчика.
         *
         * @param  \Illuminate\Events\Dispatcher  $events
         * @return void
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                [UserEventSubscriber::class, 'handleUserLogin']
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                [UserEventSubscriber::class, 'handleUserLogout']
            );
        }
    }

<a name="registering-event-subscribers"></a>
### Регистрация подписчиков на события

После написания подписчика вы готовы зарегистрировать его в диспетчере событий. Вы можете регистрировать подписчиков с помощью свойства `$subscribe` поставщика `EventServiceProvider`. Например, добавим подписчик `UserEventSubscriber` в список:

    <?php

    namespace App\Providers;

    use App\Listeners\UserEventSubscriber;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * Карта слушателей событий приложения.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * Классы подписчиков для регистрации.
         *
         * @var array
         */
        protected $subscribe = [
            UserEventSubscriber::class,
        ];
    }
