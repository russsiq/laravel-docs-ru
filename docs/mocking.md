# Laravel 9 · Тестирование · Имитация

- [Введение](#introduction)
- [Подставные объекты](#mocking-objects)
- [Имитация фасадов](#mocking-facades)
    - [Шпионы фасадов](#facade-spies)
- [Фальсификация Bus](#bus-fake)
    - [Цепочка заданий](#bus-job-chains)
    - [Пакетная обработка заданий](#job-batches)
- [Фальсификация Event](#event-fake)
    - [Ограниченная фальсификация событий](#scoped-event-fakes)
- [Фальсификация HTTP](#http-fake)
- [Фальсификация Mail](#mail-fake)
- [Фальсификация Notification](#notification-fake)
- [Фальсификация Queue](#queue-fake)
    - [Цепочка заданий](#job-chains)
- [Фальсификация Storage](#storage-fake)
- [Взаимодействие со временем](#interacting-with-time)

<a name="introduction"></a>
## Введение

При тестировании приложений Laravel бывает необходимо «сымитировать» определенные аспекты вашего приложения, чтобы они фактически не выполнялись во время текущего теста. Например, при тестировании контроллера, который инициирует событие, вы можете смоделировать слушателей событий, чтобы они фактически не выполнялись во время теста. Это позволяет вам тестировать только HTTP-ответ контроллера, не беспокоясь о запуске слушателей событий, поскольку слушатели событий могут быть протестированы в их собственном тестовом классе.

Laravel предлагает полезные методы для имитации событий, заданий и других фасадов из коробки. Эти помощники в первую очередь обеспечивают удобную обертку над Mockery, поэтому вам не нужно вручную выполнять сложные вызовы методов Mockery.

<a name="mocking-objects"></a>
## Подставные объекты

При имитации объекта, который будет внедрен в ваше приложение через [контейнер служб](container.md) Laravel, вам нужно будет привязать ваш подставной экземпляр к контейнеру с помощью `instance`. Это даст указание контейнеру использовать ваш подставной экземпляр объекта вместо создания самого объекта:

    use App\Service;
    use Mockery;
    use Mockery\MockInterface;

    public function test_something_can_be_mocked()
    {
        $this->instance(
            Service::class,
            Mockery::mock(Service::class, function (MockInterface $mock) {
                $mock->shouldReceive('process')->once();
            })
        );
    }

Чтобы сделать это более удобным, вы можете использовать метод `mock`, который обеспечен базовым классом тестов Laravel. Например, следующий пример эквивалентен приведенному выше примеру:

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->mock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

Вы можете использовать метод `partialMock`, когда вам нужно только имитировать несколько методов объекта. Методы, которые не являются сымитированными, при вызове будут выполняться в обычном режиме:

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->partialMock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

Точно так же, если вы хотите [шпионить](http://docs.mockery.io/en/latest/reference/spies.html) за объектом, базовый класс тестов Laravel содержит метод `spy` в качестве удобной обертки для метода `Mockery::spy`. Шпионы похожи на подставные объекты; однако, шпионы записывают любое взаимодействие между шпионом и тестируемым кодом, позволяя вам делать утверждения после выполнения кода:

    use App\Service;

    $spy = $this->spy(Service::class);

    // ...

    $spy->shouldHaveReceived('process');

<a name="mocking-facades"></a>
## Имитация фасадов

В отличие от традиционных вызовов статических методов, [фасады](facades.md), включая [фасады в реальном времени](facades.md#real-time-facades) можно имитировать. Это дает большое преимущество перед традиционными статическими методами и дает вам такую же возможность тестирования, как если бы вы использовали традиционное внедрение зависимостей. При тестировании вы часто можете имитировать вызов фасада Laravel, происходящий в одном из ваших контроллеров. Например, рассмотрим следующее действие контроллера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Получить список всех пользователей приложения.
         *
         * @return \Illuminate\Http\Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

Мы можем имитировать вызов фасада `Cache`, используя метод `shouldReceive`, который вернет экземпляр [Mockery](https://github.com/padraic/mockery). Поскольку фасады фактически извлекаются и управляются [контейнером служб](container.md) Laravel, то они имеют гораздо большую тестируемость, чем типичный статический класс. Например, давайте сымитируем наш вызов метода `get` фасада `Cache`:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Cache;
    use Tests\TestCase;

    class UserControllerTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> {note} Вы не должны имитировать фасад `Request`. Вместо этого передайте требуемые данные в [методы тестирования HTTP](http-tests.md), такие как `get` и `post`, при запуске вашего теста. Аналогично, вместо имитации фасада `Config`, вызовите метод `Config::set` в ваших тестах.

<a name="facade-spies"></a>
### Шпионы фасадов

Если вы хотите [шпионить](http://docs.mockery.io/en/latest/reference/spies.html) за фасадом, то вы можете вызвать метод `spy` на соответствующем фасаде. Шпионы похожи на подставные объекты; однако, шпионы записывают любое взаимодействие между шпионом и тестируемым кодом, позволяя вам делать утверждения после выполнения кода:

    use Illuminate\Support\Facades\Cache;

    public function test_values_are_be_stored_in_cache()
    {
        Cache::spy();

        $response = $this->get('/');

        $response->assertStatus(200);

        Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
    }

<a name="bus-fake"></a>
## Фальсификация Bus

При тестировании кода, который отправляет задания, вы обычно хотите подтвердить, что переданное задание было отправлено, при этом не помещая его в очередь или не выполняя это задание. Это связано с тем, что выполнение задания обычно можно протестировать в отдельном тестовом классе.

Вы можете использовать метод `fake` фасада `Bus` для предотвращения отправки заданий в очередь. Затем, после выполнения тестируемого кода, вы можете проверить, какие задания приложение пыталось отправить, используя методы `assertDispatched` и `assertNotDispatched`:

    <?php

    namespace Tests\Feature;

    use App\Jobs\ShipOrder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Bus;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped()
        {
            Bus::fake();

            // Осуществляем доставку заказа ...

            // Утверждаем, что задание было отправлено ...
            Bus::assertDispatched(ShipOrder::class);

            // Утверждаем, что задание не было отправлено ...
            Bus::assertNotDispatched(AnotherJob::class);

            // Утверждаем, что задание было отправлено синхронно ...
            Bus::assertDispatchedSync(AnotherJob::class);

            // Утверждаем, что задание не было отправлено синхронно ...
            Bus::assertNotDispatchedSync(AnotherJob::class);

            // Утверждаем, что задание было отправлено после отправки ответа ...
            Bus::assertDispatchedAfterResponse(AnotherJob::class);

            // Утверждаем, что задание не было отправлено после отправки ответа...
            Bus::assertNotDispatchedAfterResponse(AnotherJob::class);

            // Утверждаем, что никаких заданий не было отправлено ...
            Bus::assertNothingDispatched();
        }
    }

Вы можете передать замыкание в доступные методы, чтобы утверждать, что было отправлено задание, которое проходит переданный «тест истинности». Если было отправлено хотя бы одно задание, которое проходит указанный тест на истинность, то и утверждение будет считаться успешным. Например, вы можете утверждать, что задание было отправлено для определенного заказа:

    Bus::assertDispatched(function (ShipOrder $job) use ($order) {
        return $job->order->id === $order->id;
    });

<a name="bus-job-chains"></a>
### Цепочка заданий

Метод `assertChained` фасада `Bus` используется для подтверждения отправки [цепочки заданий](queues.md#job-chaining). Метод `assertChained` принимает массив связанных заданий в качестве первого аргумента:

    use App\Jobs\RecordShipment;
    use App\Jobs\ShipOrder;
    use App\Jobs\UpdateInventory;
    use Illuminate\Support\Facades\Bus;

    Bus::assertChained([
        ShipOrder::class,
        RecordShipment::class,
        UpdateInventory::class
    ]);

Как вы можете видеть в приведенном выше примере, массив связанных заданий может быть массивом имен классов заданий. Однако, вы также можете предоставить массив фактических экземпляров задания. При этом Laravel гарантирует, что экземпляры задания относятся к одному классу и имеют те же значения свойств, что и связанные задания, отправленные вашим приложением:

    Bus::assertChained([
        new ShipOrder,
        new RecordShipment,
        new UpdateInventory,
    ]);

<a name="job-batches"></a>
### Пакетная обработка заданий

Метод `assertBatched` фасада `Bus` используется для подтверждения того, что [пакет заданий](queues.md#job-batching) был отправлен. Замыкание, переданное методу `assertBatched`, получает экземпляр `Illuminate\Bus\PendingBatch`, который может использоваться для инспектирования заданий в пакете:

    use Illuminate\Bus\PendingBatch;
    use Illuminate\Support\Facades\Bus;

    Bus::assertBatched(function (PendingBatch $batch) {
        return $batch->name == 'import-csv' &&
               $batch->jobs->count() === 10;
    });

<a name="event-fake"></a>
## Фальсификация Event

При тестировании кода, отправляющего события, вы можете указать Laravel не выполнять слушателей событий. Используя метод `fake` фасада `Event`, вы можете запретить выполнение слушателей, выполнить тестируемый код, а затем утверждать, какие события были отправлены вашим приложением, используя методы `assertDispatched`, `assertNotDispatched` и `assertNothingDispatched`:

    <?php

    namespace Tests\Feature;

    use App\Events\OrderFailedToShip;
    use App\Events\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Event;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Тест доставки заказа.
         */
        public function test_orders_can_be_shipped()
        {
            Event::fake();

            // Осуществляем доставку заказа ...

            // Утверждаем, что событие было отправлено ...
            Event::assertDispatched(OrderShipped::class);

            // Утверждаем, что событие было отправлено дважды ...
            Event::assertDispatchedTimes(OrderShipped::class, 2);

            // Утверждаем, что событие не было отправлено ...
            Event::assertNotDispatched(OrderFailedToShip::class);

            // Утверждаем, что никаких событий не было отправлено ...
            Event::assertNothingDispatched();
        }
    }

Вы можете передать замыкание в методы `assertDispatched` или `assertNotDispatched`, чтобы утверждать, что было отправлено событие, которое проходит данный «тест истинности». Если было отправлено хотя бы одно событие, которое проходит заданный тест на истинность, то и утверждение будет считаться успешным:

    Event::assertDispatched(function (OrderShipped $event) use ($order) {
        return $event->order->id === $order->id;
    });

Если требуется простое утверждение того, что слушатель события прослушивает конкретное событие, то можно использовать метод `assertListening`:

    Event::assertListening(
        OrderShipped::class,
        SendShipmentNotification::class
    );

> {note} После вызова `Event::fake()` никакие слушатели событий выполняться не будут. Итак, если в ваших тестах используются фабрики моделей, которые полагаются на события, такие как создание UUID во время события `creating` модели, вы должны вызвать `Event::fake()` **после** использования ваших фабрик.

<a name="faking-a-subset-of-events"></a>
#### Фальсификация подмножества событий

Если вы хотите подделать слушателей событий только для определенного набора событий, вы можете передать их методу `fake` или `fakeFor`:

    /**
     * Тест обработки заказа.
     */
    public function test_orders_can_be_processed()
    {
        Event::fake([
            OrderCreated::class,
        ]);

        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        // Другие события отправляются как обычно ...
        $order->update([...]);
    }

<a name="scoped-event-fakes"></a>
### Ограниченная фальсификация событий

Если вы хотите подделать слушателей событий только для части вашего теста, вы можете использовать метод `fakeFor`:

    <?php

    namespace Tests\Feature;

    use App\Events\OrderCreated;
    use App\Models\Order;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Тест обработки заказа.
         */
        public function test_orders_can_be_processed()
        {
            $order = Event::fakeFor(function () {
                $order = Order::factory()->create();

                Event::assertDispatched(OrderCreated::class);

                return $order;
            });

            // События отправляются как обычно, и наблюдатели запускаются ...
            $order->update([...]);
        }
    }

<a name="http-fake"></a>
## Фальсификация HTTP

Метод `fake` фасада `Http` позволяет указать HTTP-клиенту возвращать заглушенные / фиктивные ответы при выполнении запросов. Дополнительную информацию о фальсификации исходящих HTTP-запросов см. в [документации тестирования HTTP-клиента](http-client.md#testing).

<a name="mail-fake"></a>
## Фальсификация Mail

Вы можете использовать метод `fake` фасада `Mail` для предотвращения отправки почты. Обычно отправка почты не связана с кодом, который вы фактически тестируете. Скорее всего, достаточно просто утверждать, что Laravel получил указание отправить переданное почтовое сообщение.

После вызова метода `fake` фасада `Mail` вы можете утверждать, что [почтовые службы](mail.md) были проинструктированы об отправке пользователям, и даже проверять данные, полученные почтовыми службами:

    <?php

    namespace Tests\Feature;

    use App\Mail\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Mail;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped()
        {
            Mail::fake();

            // Осуществляем доставку заказа ...

            // Утверждаем, что почтовые сообщения не были отправлены ...
            Mail::assertNothingSent();

            // Утверждаем, что почтовое сообщение было отправлено ...
            Mail::assertSent(OrderShipped::class);

            // Утверждаем, что почтовое сообщение было отправлено дважды ...
            Mail::assertSent(OrderShipped::class, 2);

            // Утверждаем, что конкретное почтовое сообщение не было отправлено ...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

Если вы ставите почтовые сообщения в очередь для доставки в фоновом режиме, вы должны использовать метод `assertQueued` вместо `assertSent`:

    Mail::assertQueued(OrderShipped::class);

    Mail::assertNotQueued(OrderShipped::class);

    Mail::assertNothingQueued();

Вы можете передать замыкание в методы `assertSent`, `assertNotSent`, `assertQueued` или `assertNotQueued`, чтобы утверждать, что было отправлено почтовое сообщение, которое проходит данный «тест истинности». Если было отправлено хотя бы одно почтовое сообщение, которое проходит заданный тест на истинность, то и утверждение будет считаться успешным:

    Mail::assertSent(function (OrderShipped $mail) use ($order) {
        return $mail->order->id === $order->id;
    });

При вызове методов утверждений фасада `Mail` экземпляр почтового сообщения, переданный замыканию, содержит полезные методы:

    Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
        return $mail->hasTo($user->email) &&
               $mail->hasCc('...') &&
               $mail->hasBcc('...') &&
               $mail->hasReplyTo('...') &&
               $mail->hasFrom('...') &&
               $mail->hasSubject('...');
    });

Вы могли заметить, что есть два метода утверждения о том, что письмо не было отправлено: `assertNotSent` и `assertNotQueued`. При желании вы можете утверждать о том, что почта не была отправлена ​​**или** поставлена ​​в очередь. Для этого вы можете использовать методы `assertNothingOutgoing` и `assertNotOutgoing`:

    Mail::assertNothingOutgoing();

    Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
        return $mail->order->id === $order->id;
    });

<a name="testing-mailable-content"></a>
#### Тестирование содержимого почтового отправления

Мы предлагаем тестировать содержимое ваших почтовых отправлений отдельно от ваших тестов, содержащих утверждение о том, что данное почтовое отправление было «отправлено» конкретному пользователю. Чтобы узнать, как тестировать содержимое ваших почтовых отправлений, ознакомьтесь с документацией по [тестированию почтовых отправлений](mail.md#testing-mailables).

<a name="notification-fake"></a>
## Фальсификация Notification

Вы можете использовать метод `fake` фасада `Notification` для предотвращения отправки уведомлений. Обычно отправка уведомлений не связана с кодом, который вы фактически тестируете. Скорее всего, достаточно просто утверждать, что Laravel получил указание отправить переданное уведомление.

После вызова метода `fake` фасада `Notification` вы можете утверждать, что [службы уведомлений](notifications.md) были проинструктированы об отправке пользователям, и даже проверять данные, полученные в уведомлениях:

    <?php

    namespace Tests\Feature;

    use App\Notifications\OrderShipped;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Notification;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped()
        {
            Notification::fake();

            // Осуществляем доставку заказа ...

            // Утверждаем, что уведомления не были отправлены ...
            Notification::assertNothingSent();

            // Утверждаем, что указанным пользователям было отправлено уведомление ...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // Утверждаем, что уведомление не было отправлено ...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );

            // Утверждаем, что указанное количество уведомлений было отправлено ...
            Notification::assertCount(3);
        }
    }

Вы можете передать замыкание в методы `assertSentTo` или `assertNotSentTo`, чтобы утверждать, что было отправлено уведомление, которое проходит данный «тест истинности». Если было отправлено хотя бы одно уведомление, которое проходит заданный тест на истинность, то и утверждение будет считаться успешным:

    Notification::assertSentTo(
        $user,
        function (OrderShipped $notification, $channels) use ($order) {
            return $notification->order->id === $order->id;
        }
    );

<a name="on-demand-notifications"></a>
#### Уведомления по запросу

Если код, который вы тестируете, отправляет [уведомления по запросу](notifications.md#on-demand-notifications), вам нужно будет тестировать отправку таких уведомлений через метод `assertSentOnDemand`:

    Notification::assertSentOnDemand(OrderShipped::class);

Передав замыкание в качестве второго аргумента методу `assertSentOnDemand`, вы можете определить, было ли отправлено уведомление по запросу на правильный адрес «маршрута»:

    Notification::assertSentOnDemand(
        OrderShipped::class,
        function ($notification, $channels, $notifiable) use ($user) {
            return $notifiable->routes['mail'] === $user->email;
        }
    );

<a name="queue-fake"></a>
## Фальсификация Queue

Вы можете использовать метод `fake` фасада `Queue` для предотвращения помещения заданий в очередь. Скорее всего, достаточно просто заявить, что Laravel получил указание поместить переданное задание в очередь, поскольку сами задания в очереди могут быть протестированы в другом тестовом классе.

После вызова метода `fake` фасада `Queue` вы можете утверждать, что приложение пыталось поместить задания в очередь:

    <?php

    namespace Tests\Feature;

    use App\Jobs\AnotherJob;
    use App\Jobs\FinalJob;
    use App\Jobs\ShipOrder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Support\Facades\Queue;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_orders_can_be_shipped()
        {
            Queue::fake();

            // Осуществляем доставку заказа ...

            // Утверждаем, что задания не были помещены ...
            Queue::assertNothingPushed();

            // Утверждаем, что задание было помещено в конкретную очередь ...
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // Утверждаем, что задание было помещено дважды ...
            Queue::assertPushed(ShipOrder::class, 2);

            // Утверждаем, что задание не было помещено ...
            Queue::assertNotPushed(AnotherJob::class);
        }
    }

Вы можете передать замыкание в методы `assertPushed` или `assertNotPushed`, чтобы утверждать, что было отправлено задание, которое проходит данный «тест истинности». Если было отправлено хотя бы одно задание, которое проходит заданный тест на истинность, то и утверждение будет считаться успешным:

    Queue::assertPushed(function (ShipOrder $job) use ($order) {
        return $job->order->id === $order->id;
    });

<a name="job-chains"></a>
### Цепочка заданий

Методы `assertPushedWithChain` и `assertPushedWithoutChain` фасада `Queue` могут использоваться для проверки цепочки заданий отправляемого задания. Метод `assertPushedWithChain` принимает основное задание в качестве своего первого аргумента и массив связанных заданий в качестве второго аргумента:

    use App\Jobs\RecordShipment;
    use App\Jobs\ShipOrder;
    use App\Jobs\UpdateInventory;
    use Illuminate\Support\Facades\Queue;

    Queue::assertPushedWithChain(ShipOrder::class, [
        RecordShipment::class,
        UpdateInventory::class
    ]);

Как вы можете видеть в приведенном выше примере, массив связанных заданий может быть массивом имен классов заданий. Однако, вы также можете предоставить массив фактических экземпляров задания. При этом Laravel гарантирует, что экземпляры задания относятся к одному классу и имеют те же значения свойств, что и связанные задания, отправленные вашим приложением:

    Queue::assertPushedWithChain(ShipOrder::class, [
        new RecordShipment,
        new UpdateInventory,
    ]);

Вы можете использовать метод `assertPushedWithoutChain`, чтобы подтвердить, что задание было отправлено без цепочки заданий:

    Queue::assertPushedWithoutChain(ShipOrder::class);

<a name="storage-fake"></a>
## Фальсификация Storage

Метод `fake` фасада `Storage` позволяет легко сгенерировать фиктивный диск, который в сочетании с утилитами генерации файлов класса `Illuminate\Http\UploadedFile` значительно упрощает тестирование загрузки файлов. Например:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_albums_can_be_uploaded()
        {
            Storage::fake('photos');

            $response = $this->json('POST', '/photos', [
                UploadedFile::fake()->image('photo1.jpg'),
                UploadedFile::fake()->image('photo2.jpg')
            ]);

            // Утверждаем, что один или несколько файлов были сохранены ...
            Storage::disk('photos')->assertExists('photo1.jpg');
            Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

            // Утверждаем, что один или несколько файлов не были сохранены ...
            Storage::disk('photos')->assertMissing('missing.jpg');
            Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

            // Утверждаем, что указанный каталог пуст...
            Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
        }
    }

По умолчанию метод `fake` удалит все файлы во временном каталоге. Если вы хотите сохранить эти файлы, то вы можете вместо этого использовать метод `persistentFake`. Для получения дополнительной информации о тестировании загрузки файлов вы можете ознакомиться с [информацией по загрузке файлов из документации тестирования HTTP](http-tests.md#testing-file-uploads).

> {note} Для метода `image` требуется [расширение GD](https://www.php.net/manual/ru/book.image.php).

<a name="interacting-with-time"></a>
## Взаимодействие со временем

При тестировании вам может иногда потребоваться изменить время, возвращаемое такими помощниками, как `now` или `Illuminate\Support\Carbon::now()`. К счастью, базовый класс тестирования функций Laravel включает помощников, которые позволяют вам управлять текущим временем:

    use Illuminate\Support\Carbon;

    public function testTimeCanBeManipulated()
    {
        // Путешествие в будущее ...
        $this->travel(5)->milliseconds();
        $this->travel(5)->seconds();
        $this->travel(5)->minutes();
        $this->travel(5)->hours();
        $this->travel(5)->days();
        $this->travel(5)->weeks();
        $this->travel(5)->years();

        // Заморозить время и возобновить его после выполнения замыкания ...
        $this->freezeTime(function (Carbon $time) {
            // ...
        });

        // Путешествие в прошлое ...
        $this->travel(-5)->hours();

        // Путешествие в определенное время ...
        $this->travelTo(now()->subHours(6));

        // Вернуться в настоящее время ...
        $this->travelBack();
    }
