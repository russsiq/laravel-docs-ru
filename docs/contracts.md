# Laravel 9 · Контракты

- [Введение](#introduction)
    - [Контракты против Фасадов](#contracts-vs-facades)
- [Когда использовать контракты](#when-to-use-contracts)
- [Как использовать контракты](#how-to-use-contracts)
- [Справочник контрактов](#contract-reference)

<a name="introduction"></a>
## Введение

«Контракты» Laravel – это набор интерфейсов, которые определяют основные службы фреймворка. Например, контракт `Illuminate\Contracts\Queue\Queue` определяет методы, необходимые для постановки заданий в очередь, а контракт `Illuminate\Contracts\Mail\Mailer` – для отправки электронной почты.

Каждый контракт имеет соответствующую реализацию, предусмотренную структурой. Например, Laravel предлагает реализацию очереди с множеством драйверов и реализацию почтовой программы, которая работает на [Symfony Mailer](https://symfony.com/doc/6.0/mailer.html).

Все контракты Laravel находятся в [их собственном репозитории](https://github.com/illuminate/contracts) GitHub. Это обеспечивает быстрый доступ к списку всех доступных контрактов, а также единый, отдельный пакет, который используется разработчиками пакетов, взаимодействующих со службами Laravel.

<a name="contracts-vs-facades"></a>
### Контракты против Фасадов

[Фасады](facades.md) и глобальные помощники Laravel обеспечивают простой способ использования служб Laravel без объявления типов зависимости и извлечения их реализаций из контейнера служб. В большинстве случаев каждый фасад имеет эквивалентный контракт.

В отличие от фасадов, которые не требуют, чтобы они находились в конструкторе вашего класса, контракты позволяют вам определять явные зависимости для ваших классов. Некоторые разработчики предпочитают явно определять свои зависимости таким образом и поэтому предпочитают использовать контракты, в то время как другие разработчики пользуются удобством фасадов. **В общем, большинство приложений могут без проблем использовать фасады во время разработки.**

<a name="when-to-use-contracts"></a>
## Когда использовать контракты

Решение об использовании контрактов или фасадов будет зависеть от личного вкуса и вкусов вашей команды разработчиков. И контракты, и фасады могут использоваться для создания надежных, хорошо тестируемых приложений Laravel. Контракты и фасады не исключают друг друга. Некоторые части ваших приложений могут использовать фасады, а другие могут зависеть от контрактов. До тех пор, пока вы сосредоточены на обязанностях класса, вы не заметите практических различий между использованием контрактов и фасадов.

В общем, большинство приложений могут без проблем использовать фасады во время разработки. Если вы создаете пакет, который интегрируется с несколькими фреймворками PHP, то вы можете указать пакет [`illuminate/contracts`](https://github.com/illuminate/contracts) в файле `composer.json` вашего пакета для определения вашей интеграции со службами Laravel без необходимости требований конкретных реализаций Laravel.

<a name="how-to-use-contracts"></a>
## Как использовать контракты

Итак, как получить реализацию контракта? На самом деле это довольно просто.

В Laravel многие типы классов извлекаются через [контейнер служб](container.md); к таким классам относятся контроллеры, слушатели событий, посредники, очереди заданий и даже замыкания маршрутов. Итак, чтобы получить реализацию контракта, вы можете просто «объявить тип» интерфейса в конструкторе извлекаемого класса.

Например, взгляните на этого слушателя события:

    <?php

    namespace App\Listeners;

    use App\Events\OrderWasPlaced;
    use App\Models\User;
    use Illuminate\Contracts\Redis\Factory;

    class CacheOrderInformation
    {
        /**
         * Реализация фабрики Redis.
         *
         * @var \Illuminate\Contracts\Redis\Factory
         */
        protected $redis;

        /**
         * Создать новый экземпляр обработчика события.
         *
         * @param  \Illuminate\Contracts\Redis\Factory  $redis
         * @return void
         */
        public function __construct(Factory $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Обработать событие.
         *
         * @param  \App\Events\OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

Когда слушатель события будет извлечен, контейнер служб, используя объявление типов в конструкторе класса, внедрит соответствующую зависимость. Чтобы узнать больше о регистрации в контейнере служб, ознакомьтесь с [его документацией](container.md).

<a name="contract-reference"></a>
## Справочник контрактов

В этой таблице содержится краткий справочник по всем контрактам Laravel и их эквивалентным фасадам:

|  Контракт  |  Фасад  |
|-------------|-------------|
|  [Illuminate\Contracts\Auth\Access\Authorizable](https://github.com/illuminate/contracts/blob/9.x/Auth/Access/Authorizable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Auth\Access\Gate](https://github.com/illuminate/contracts/blob/9.x/Auth/Access/Gate.php)  |  `Gate`  |
|  [Illuminate\Contracts\Auth\Authenticatable](https://github.com/illuminate/contracts/blob/9.x/Auth/Authenticatable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Auth\CanResetPassword](https://github.com/illuminate/contracts/blob/9.x/Auth/CanResetPassword.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/9.x/Auth/Factory.php)  |  `Auth`  |
|  [Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/9.x/Auth/Guard.php)  |  `Auth::guard()`  |
|  [Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/9.x/Auth/PasswordBroker.php)  |  `Password::broker()`  |
|  [Illuminate\Contracts\Auth\PasswordBrokerFactory](https://github.com/illuminate/contracts/blob/9.x/Auth/PasswordBrokerFactory.php)  |  `Password`  |
|  [Illuminate\Contracts\Auth\StatefulGuard](https://github.com/illuminate/contracts/blob/9.x/Auth/StatefulGuard.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Auth\SupportsBasicAuth](https://github.com/illuminate/contracts/blob/9.x/Auth/SupportsBasicAuth.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Auth\UserProvider](https://github.com/illuminate/contracts/blob/9.x/Auth/UserProvider.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/9.x/Bus/Dispatcher.php)  |  `Bus`  |
|  [Illuminate\Contracts\Bus\QueueingDispatcher](https://github.com/illuminate/contracts/blob/9.x/Bus/QueueingDispatcher.php)  |  `Bus::dispatchToQueue()`  |
|  [Illuminate\Contracts\Broadcasting\Factory](https://github.com/illuminate/contracts/blob/9.x/Broadcasting/Factory.php)  |  `Broadcast`  |
|  [Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/9.x/Broadcasting/Broadcaster.php)  |  `Broadcast::connection()` |
|  [Illuminate\Contracts\Broadcasting\ShouldBroadcast](https://github.com/illuminate/contracts/blob/9.x/Broadcasting/ShouldBroadcast.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Broadcasting\ShouldBroadcastNow](https://github.com/illuminate/contracts/blob/9.x/Broadcasting/ShouldBroadcastNow.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/9.x/Cache/Factory.php)  |  `Cache`  |
|  [Illuminate\Contracts\Cache\Lock](https://github.com/illuminate/contracts/blob/9.x/Cache/Lock.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Cache\LockProvider](https://github.com/illuminate/contracts/blob/9.x/Cache/LockProvider.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/9.x/Cache/Repository.php)  |  `Cache::driver()`  |
|  [Illuminate\Contracts\Cache\Store](https://github.com/illuminate/contracts/blob/9.x/Cache/Store.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/9.x/Config/Repository.php)  |  `Config`  |
|  [Illuminate\Contracts\Console\Application](https://github.com/illuminate/contracts/blob/9.x/Console/Application.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Console\Kernel](https://github.com/illuminate/contracts/blob/9.x/Console/Kernel.php)  |  `Artisan`  |
|  [Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/9.x/Container/Container.php)  |  `App`  |
|  [Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/9.x/Cookie/Factory.php)  |  `Cookie`  |
|  [Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/9.x/Cookie/QueueingFactory.php)  |  `Cookie::queue()`  |
|  [Illuminate\Contracts\Database\ModelIdentifier](https://github.com/illuminate/contracts/blob/9.x/Database/ModelIdentifier.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Debug\ExceptionHandler](https://github.com/illuminate/contracts/blob/9.x/Debug/ExceptionHandler.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/9.x/Encryption/Encrypter.php)  |  `Crypt`  |
|  [Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/9.x/Events/Dispatcher.php)  |  `Event`  |
|  [Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/9.x/Filesystem/Cloud.php)  |  `Storage::cloud()`  |
|  [Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/9.x/Filesystem/Factory.php)  |  `Storage`  |
|  [Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/9.x/Filesystem/Filesystem.php)  |  `Storage::disk()`  |
|  [Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/9.x/Foundation/Application.php)  |  `App`  |
|  [Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/9.x/Hashing/Hasher.php)  |  `Hash`  |
|  [Illuminate\Contracts\Http\Kernel](https://github.com/illuminate/contracts/blob/9.x/Http/Kernel.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/9.x/Mail/MailQueue.php)  |  `Mail::queue()`  |
|  [Illuminate\Contracts\Mail\Mailable](https://github.com/illuminate/contracts/blob/9.x/Mail/Mailable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/9.x/Mail/Mailer.php)  |  `Mail`  |
|  [Illuminate\Contracts\Notifications\Dispatcher](https://github.com/illuminate/contracts/blob/9.x/Notifications/Dispatcher.php)  |  `Notification`  |
|  [Illuminate\Contracts\Notifications\Factory](https://github.com/illuminate/contracts/blob/9.x/Notifications/Factory.php)  |  `Notification`  |
|  [Illuminate\Contracts\Pagination\LengthAwarePaginator](https://github.com/illuminate/contracts/blob/9.x/Pagination/LengthAwarePaginator.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Pagination\Paginator](https://github.com/illuminate/contracts/blob/9.x/Pagination/Paginator.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Pipeline\Hub](https://github.com/illuminate/contracts/blob/9.x/Pipeline/Hub.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Pipeline\Pipeline](https://github.com/illuminate/contracts/blob/9.x/Pipeline/Pipeline.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Queue\EntityResolver](https://github.com/illuminate/contracts/blob/9.x/Queue/EntityResolver.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/9.x/Queue/Factory.php)  |  `Queue`  |
|  [Illuminate\Contracts\Queue\Job](https://github.com/illuminate/contracts/blob/9.x/Queue/Job.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Queue\Monitor](https://github.com/illuminate/contracts/blob/9.x/Queue/Monitor.php)  |  `Queue`  |
|  [Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/9.x/Queue/Queue.php)  |  `Queue::connection()`  |
|  [Illuminate\Contracts\Queue\QueueableCollection](https://github.com/illuminate/contracts/blob/9.x/Queue/QueueableCollection.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Queue\QueueableEntity](https://github.com/illuminate/contracts/blob/9.x/Queue/QueueableEntity.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Queue\ShouldQueue](https://github.com/illuminate/contracts/blob/9.x/Queue/ShouldQueue.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/9.x/Redis/Factory.php)  |  `Redis`  |
|  [Illuminate\Contracts\Routing\BindingRegistrar](https://github.com/illuminate/contracts/blob/9.x/Routing/BindingRegistrar.php)  |  `Route`  |
|  [Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/9.x/Routing/Registrar.php)  |  `Route`  |
|  [Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/9.x/Routing/ResponseFactory.php)  |  `Response`  |
|  [Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/9.x/Routing/UrlGenerator.php)  |  `URL`  |
|  [Illuminate\Contracts\Routing\UrlRoutable](https://github.com/illuminate/contracts/blob/9.x/Routing/UrlRoutable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Session\Session](https://github.com/illuminate/contracts/blob/9.x/Session/Session.php)  |  `Session::driver()`  |
|  [Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/9.x/Support/Arrayable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Support\Htmlable](https://github.com/illuminate/contracts/blob/9.x/Support/Htmlable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/9.x/Support/Jsonable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Support\MessageBag](https://github.com/illuminate/contracts/blob/9.x/Support/MessageBag.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Support\MessageProvider](https://github.com/illuminate/contracts/blob/9.x/Support/MessageProvider.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/9.x/Support/Renderable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Support\Responsable](https://github.com/illuminate/contracts/blob/9.x/Support/Responsable.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Translation\Loader](https://github.com/illuminate/contracts/blob/9.x/Translation/Loader.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Translation\Translator](https://github.com/illuminate/contracts/blob/9.x/Translation/Translator.php)  |  `Lang`  |
|  [Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/9.x/Validation/Factory.php)  |  `Validator`  |
|  [Illuminate\Contracts\Validation\ImplicitRule](https://github.com/illuminate/contracts/blob/9.x/Validation/ImplicitRule.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Validation\Rule](https://github.com/illuminate/contracts/blob/9.x/Validation/Rule.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Validation\ValidatesWhenResolved](https://github.com/illuminate/contracts/blob/9.x/Validation/ValidatesWhenResolved.php)  |  &nbsp;  |
|  [Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/9.x/Validation/Validator.php)  |  `Validator::make()`  |
|  [Illuminate\Contracts\View\Engine](https://github.com/illuminate/contracts/blob/9.x/View/Engine.php)  |  &nbsp;  |
|  [Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/9.x/View/Factory.php)  |  `View`  |
|  [Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/9.x/View/View.php)  |  `View::make()`  |
