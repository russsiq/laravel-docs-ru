# HTTP-клиент

- [Введение](#introduction)
- [Выполнение запросов](#making-requests)
    - [Данные запроса](#request-data)
    - [Заголовки](#headers)
    - [Аутентификация](#authentication)
    - [Время ожидания](#timeout)
    - [Повторные попытки](#retries)
    - [Обработка ошибок](#error-handling)
    - [Параметры Guzzle](#guzzle-options)
- [Тестирование](#testing)
    - [Фиктивные ответы](#faking-responses)
    - [Инспектирование запросов](#inspecting-requests)

<a name="introduction"></a>
## Введение

Laravel предоставляет выразительный минимальный API-интерфейс для [Guzzle HTTP-клиента](http://docs.guzzlephp.org/en/stable/), позволяющий быстро выполнять исходящие HTTP-запросы для взаимодействия с другими веб-приложениями. Обертка Laravel вокруг Guzzle сосредоточена на наиболее распространенных вариантах использования и дает прекрасные возможности для разработки.

Прежде чем начать, вы должны убедиться, что вы установили пакет Guzzle как зависимость вашего приложения. По умолчанию Laravel автоматически включает эту зависимость. Однако, если вы ранее удалили пакет, вы можете установить его снова через Composer:

    composer require guzzlehttp/guzzle

<a name="making-requests"></a>
## Выполнение запросов

Для отправки запросов вы можете использовать методы `get`, `post`, `put`, `patch` и `delete` фасада `Http`. Сначала давайте рассмотрим, как сделать основной `GET` запрос:

    use Illuminate\Support\Facades\Http;

    $response = Http::get('http://example.com');

Метод `get` возвращает экземпляр `Illuminate\Http\Client\Response`, который содержит множество методов, которые можно использовать для инспектирования ответа:

    $response->body() : string;
    $response->json() : array|mixed;
    $response->status() : int;
    $response->ok() : bool;
    $response->successful() : bool;
    $response->failed() : bool;
    $response->serverError() : bool;
    $response->clientError() : bool;
    $response->header($header) : string;
    $response->headers() : array;

Объект `Illuminate\Http\Client\Response` также реализует интерфейс PHP `ArrayAccess`, позволяющий получать доступ к данным JSON непосредственно ответа:

    return Http::get('http://example.com/users/1')['name'];

<a name="request-data"></a>
### Данные запроса

Конечно, при запросах `POST`, `PUT` и `PATCH` обычно отправляются дополнительные данные, поэтому эти методы принимают массив данных в качестве второго аргумента. По умолчанию данные будут отправляться с использованием типа содержимого `application/json`:

    use Illuminate\Support\Facades\Http;

    $response = Http::post('http://example.com/users', [
        'name' => 'Steve',
        'role' => 'Network Administrator',
    ]);

<a name="get-request-query-parameters"></a>
#### Параметры GET-запроса

При выполнении запросов `GET` вы можете либо добавить строку запроса к URL-адресу напрямую, либо передать массив пар ключ / значение в качестве второго аргумента метода `get`:

    $response = Http::get('http://example.com/users', [
        'name' => 'Taylor',
        'page' => 1,
    ]);

<a name="sending-form-url-encoded-requests"></a>
#### Отправка запросов с передачей данных в URL-кодированной строке

Если вы хотите отправлять данные с использованием типа содержимого `application/x-www-form-urlencoded`, вы должны вызвать метод `asForm` перед выполнением запроса:

    $response = Http::asForm()->post('http://example.com/users', [
        'name' => 'Sara',
        'role' => 'Privacy Consultant',
    ]);

<a name="sending-a-raw-request-body"></a>
#### Отправка необработанного тела запроса

Вы можете использовать метод `withBody`, если хотите передать необработанное тело запроса при его выполнении. Тип контента может быть указан вторым аргументом метода:

    $response = Http::withBody(
        base64_encode($photo), 'image/jpeg'
    )->post('http://example.com/photo');

<a name="multi-part-requests"></a>
#### Составные запросы

Если вы хотите отправлять файлы в запросах, состоящих из нескольких частей, необходимо вызвать метод `attach` перед выполнением запроса. Этот метод принимает имя файла и его содержимое. При желании вы можете указать третий аргумент, который будет считаться именем файла:

    $response = Http::attach(
        'attachment', file_get_contents('photo.jpg'), 'photo.jpg'
    )->post('http://example.com/attachments');

Вместо передачи необработанного содержимого файла вы также можете передать потоковый ресурс:

    $photo = fopen('photo.jpg', 'r');

    $response = Http::attach(
        'attachment', $photo, 'photo.jpg'
    )->post('http://example.com/attachments');

<a name="headers"></a>
### Заголовки

Заголовки могут быть добавлены к запросам с помощью метода `withHeaders`. Этот метод `withHeaders` принимает массив пар ключ / значение:

    $response = Http::withHeaders([
        'X-First' => 'foo',
        'X-Second' => 'bar'
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
    ]);

<a name="authentication"></a>
### Аутентификация

Вы можете указать **basic** и **digest** данные аутентификации, используя методы `withBasicAuth` и `withDigestAuth` соответственно:

    // Обычная проверка подлинности ...
    $response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(...);

    // Дайджест-проверка подлинности ...
    $response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(...);

<a name="bearer-tokens"></a>
#### Bearer токены

Если вы хотите быстро добавить токен-носитель в заголовок запроса `Authorization`, вы можете использовать метод `withToken`:

    $response = Http::withToken('token')->post(...);

<a name="timeout"></a>
### Время ожидания

Метод `timeout` может использоваться для указания максимального количества секунд ожидания ответа:

    $response = Http::timeout(3)->get(...);

Если уазанный тайм-аут превышен, то будет брошен экземпляр `Illuminate\Http\Client\ConnectionException`.

<a name="retries"></a>
### Повторные попытки

Если вы хотите, чтобы HTTP-клиент автоматически повторял запрос при возникновении ошибки клиента или сервера, вы можете использовать метод `retry`. Метод `retry` принимает два аргумента: максимальное количество попыток выполнения запроса и количество миллисекунд, которые Laravel должен ждать между попытками:

    $response = Http::retry(3, 100)->post(...);

Если все запросы терпят неудачу, будет брошен экземпляр `Illuminate\Http\Client\RequestException`.

<a name="error-handling"></a>
### Обработка ошибок

В отличие от поведения Guzzle по умолчанию, оболочка HTTP-клиента Laravel не генерирует исключений при ошибках клиента или сервера (ответы `400` и` 500` от серверов). Вы можете определить, была ли возвращена одна из этих ошибок, используя методы `successful`, `clientError`, или `serverError`:

    // Определить, имеет ли ответ код состояния >= 200 and < 300...
    $response->successful();

    // Определить, имеет ли ответ код состояния >= 400...
    $response->failed();

    // Определить, имеет ли ответ код состояния 400 ...
    $response->clientError();

    // Определить, имеет ли ответ код состояния 500 ...
    $response->serverError();

<a name="throwing-exceptions"></a>
#### Выброс исключений

Если у вас есть экземпляр ответа и вы хотите выбросить экземпляр `Illuminate\Http\Client\RequestException`, если код состояния ответа указывает на ошибку клиента или сервера, вы можете использовать метод `throw`:

    $response = Http::post(...);

    // Выбросить исключение, если произошла ошибка клиента или сервера ...
    $response->throw();

    return $response['user']['id'];

Экземпляр `Illuminate\Http\Client\RequestException` имеет общедоступное свойство `$response`, которое позволит вам проверить возвращенный ответ.

Метод `throw` возвращает экземпляр ответа, если ошибки не произошло, что позволяет вам использовать цепочку вызовов после метода `throw`:

    return Http::post(...)->throw()->json();

Если вы хотите выполнить некоторую дополнительную логику до того, как будет сгенерировано исключение, вы можете передать замыкание методу `throw`. Исключение будет сгенерировано автоматически после вызова замыкания, поэтому вам не нужно повторно генерировать исключение изнутри замыкания:

    return Http::post(...)->throw(function ($response, $e) {
        //
    })->json();

<a name="guzzle-options"></a>
### Параметры Guzzle

Вы можете указать дополнительные [параметры запроса для Guzzle](http://docs.guzzlephp.org/en/stable/request-options.html), используя метод `withOptions`. Метод `withOptions` принимает массив пар ключ / значение:

    $response = Http::withOptions([
        'debug' => true,
    ])->get('http://example.com/users');

<a name="testing"></a>
## Тестирование

Многие службы Laravel содержат функционал, помогающий вам легко и выразительно писать тесты, и HTTP-оболочка Laravel не является исключением. Метод `fake` фасада `Http` позволяет указать HTTP-клиенту возвращать заглушенные / фиктивные ответы при выполнении запросов.

<a name="faking-responses"></a>
### Фиктивные ответы

Например, чтобы дать указание HTTP-клиенту возвращать пустые ответы c кодом состояния `200` на каждый запрос, вы можете вызвать метод `fake` без аргументов:

    use Illuminate\Support\Facades\Http;

    Http::fake();

    $response = Http::post(...);

> {note} При фальсификации запросов посредник HTTP-клиента не выполняется. Вы должны определить ожидания для фиктивных ответов, как если бы этот посредник работал правильно.

<a name="faking-specific-urls"></a>
#### Фальсификация конкретных URL

В качестве альтернативы вы можете передать массив методу `fake`. Ключи массива должны представлять шаблоны URL, которые вы хотите подделать, и связанные с ними ответы. Символ `*` может использоваться как символ подстановки. Любые запросы к URL-адресам, которые не были сфальсифицированы, будут выполнены фактически. Вы можете использовать метод `response` фасада `Http` для создания заглушек / фиктивных ответов для этих адресов:

    Http::fake([
        // Заглушка JSON ответа для адресов GitHub ...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),

        // Заглушка строкового ответа для адресов Google ...
        'google.com/*' => Http::response('Hello World', 200, $headers),
    ]);

Если вы хотите указать шаблон резервного URL-адреса, который будет заглушать все несопоставленные URL-адреса, то используйте символ `*`:

    Http::fake([
        // Заглушка JSON ответа для адресов GitHub ...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Заглушка строкового ответа для всех остальных адресов ...
        '*' => Http::response('Hello World', 200, ['Headers']),
    ]);

<a name="faking-response-sequences"></a>
#### Фальсификация серии ответов

По желанию можно указать, что один URL должен возвращать серию фиктивных ответов в определенном порядке. Вы можете сделать это, используя метод `Http::sequence` для составления ответов:

    Http::fake([
        // Заглушка серии ответов для адресов GitHub ...
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->pushStatus(404),
    ]);

Когда все ответы в этой последовательности будут использованы, любые дальнейшие запросы приведут к выбросу исключения. Если вы хотите указать ответ по умолчанию, который должен возвращаться, когда последовательность пуста, то вы можете использовать метод `whenEmpty`:

    Http::fake([
        // Заглушка серии ответов для адресов GitHub ...
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->whenEmpty(Http::response()),
    ]);

Если вы хотите подделать серию ответов, но вам не нужно указывать конкретный шаблон URL, который следует подделать, то вы можете использовать метод `Http::fakeSequence`:

    Http::fakeSequence()
            ->push('Hello World', 200)
            ->whenEmpty(Http::response());

<a name="fake-callback"></a>
#### Анонимные фальсификаторы

Если вам требуется более сложная логика для определения того, какие ответы возвращать для определенных адресов, то вы можете передать замыкание методу `fake`. Это замыкание получит экземпляр `Illuminate\Http\Client\Request` и должно вернуть экземпляр ответа. В замыкании вы можете выполнить любую логику, необходимую для определения типа ответа, который нужно вернуть:

    Http::fake(function ($request) {
        return Http::response('Hello World', 200);
    });

<a name="inspecting-requests"></a>
### Инспектирование запросов

При фальсификации ответов вы можете иногда захотеть проверить запросы, которые получает клиент, чтобы убедиться, что ваше приложение отправляет правильные данные или заголовки. Вы можете сделать это, вызвав метод `Http::assertSent` после вызова `Http::fake`.

Метод `assertSent` принимает замыкание, которому будет передан экземпляр `Illuminate\Http\Client\Request` и, которое должно возвращать значение логического типа, указывающее, соответствует ли запрос вашим ожиданиям. Для успешного прохождения теста должен быть выдан хотя бы один запрос, соответствующий указанным ожиданиям:

    use Illuminate\Http\Client\Request;
    use Illuminate\Support\Facades\Http;

    Http::fake();

    Http::withHeaders([
        'X-First' => 'foo',
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertSent(function (Request $request) {
        return $request->hasHeader('X-First', 'foo') &&
               $request->url() == 'http://example.com/users' &&
               $request['name'] == 'Taylor' &&
               $request['role'] == 'Developer';
    });

При необходимости вы можете проверить утверждение с помощью метода `assertNotSent`, что конкретный запрос не был отправлен:

    use Illuminate\Http\Client\Request;
    use Illuminate\Support\Facades\Http;

    Http::fake();

    Http::post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertNotSent(function (Request $request) {
        return $request->url() === 'http://example.com/posts';
    });

Или вы можете использовать метод `assertNothingSent`, чтобы проверить утверждение, что во время теста не было отправлено никаких запросов:

    Http::fake();

    Http::assertNothingSent();
