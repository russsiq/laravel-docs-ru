# Laravel 9 · Ограничители частоты

- [Введение](#introduction)
    - [Конфигурация кеша](#cache-configuration)
- [Основы использования](#basic-usage)
    - [Ручное увеличение попыток](#manually-incrementing-attempts)
    - [Очистка попыток](#clearing-attempts)

<a name="introduction"></a>
## Введение

Laravel включает простую в использовании абстракцию ограничения частоты, которая в сочетании с [кешем](cache.md) вашего приложения обеспечивает простой способ ограничить любое действие в течение указанного периода времени.

> {tip} Если вас интересует ограничение частоты входящих HTTP-запросов, обратитесь к [документации посредников](routing.md#rate-limiting).

<a name="cache-configuration"></a>
### Конфигурация кеша

Обычно ограничитель частоты использует кеш вашего приложения по умолчанию, как определено ключом `default` в конфигурационном файле `config/cache.php` вашего приложения. Однако вы можете указать, какой драйвер кеша должен использовать ограничитель частоты, задав ключ `limiter` в конфигурационном файле `config/cache.php` вашего приложения:

    'default' => 'memcached',

    'limiter' => 'redis',

<a name="basic-usage"></a>
## Основы использования

Фасад `Illuminate\Support\Facades\RateLimiter` может использоваться для взаимодействия с ограничителем частоты. Самый простой метод, предлагаемый ограничителем частоты, – это метод `attempt`, который ограничивает частоту переданного замыкания на указанное количество секунд.

Метод `attempt` возвращает `false`, если для замыкания не осталось доступных попыток; в противном случае метод `attempt` вернет результат замыкания или `true`. Первый аргумент, принимаемый методом `attempt`, – это строковый «ключ» ограничителя частоты, представляющий действие, ограничиваемое по частоте:

    use Illuminate\Support\Facades\RateLimiter;

    $executed = RateLimiter::attempt(
        'send-message:'.$user->id,
        $perMinute = 5,
        function() {
            // Send message...
        }
    );

    if (! $executed) {
      return 'Too many messages sent!';
    }

<a name="manually-incrementing-attempts"></a>
### Ручное увеличение попыток

Если вы хотите вручную взаимодействовать с ограничителем частоты, то для этого доступно множество других методов. Например, вы можете вызвать метод `tooManyAttempts`, чтобы определить, не превысил ли заданный ключ ограничителя частоты максимальное количество разрешенных попыток в минуту:

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
        return 'Too many attempts!';
    }

В качестве альтернативы, вы можете использовать метод `remaining`, чтобы получить количество оставшихся попыток для конкретного ключа. Если для переданного ключа остались возможные попытки, вы можете вызвать метод `hit`, чтобы увеличить общее количество попыток:

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
        RateLimiter::hit('send-message:'.$user->id);

        // Send message...
    }

<a name="determining-limiter-availability"></a>
#### Определение доступности ограничителя

Когда для ключа больше не осталось попыток, метод `availableIn` возвращает количество секунд, оставшихся до тех пор, пока не станут доступны другие попытки:

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
        $seconds = RateLimiter::availableIn('send-message:'.$user->id);

        return 'You may try again in '.$seconds.' seconds.';
    }

<a name="clearing-attempts"></a>
### Очистка попыток

Вы можете сбросить количество попыток для конкретного ключа ограничителя частоты, используя метод `clear`. Например, вы можете сбросить количество попыток, когда переданное сообщение прочитано получателем:

    use App\Models\Message;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Отметить сообщение как прочитанное.
     *
     * @param  \App\Models\Message  $message
     * @return \App\Models\Message
     */
    public function read(Message $message)
    {
        $message->markAsRead();

        RateLimiter::clear('send-message:'.$message->user_id);

        return $message;
    }
