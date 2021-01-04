# Тесты HTTP

- [Введение](#introduction)
- [Выполнение запросов](#making-requests)
    - [Настройка заголовков запросов](#customizing-request-headers)
    - [Cookies](#cookies)
    - [Сессия / Аутентификация](#session-and-authentication)
    - [Отладка ответов](#debugging-responses)
- [Тестирование JSON API](#testing-json-apis)
- [Тестирование загрузки файлов](#testing-file-uploads)
- [Тестирование шаблонной системы](#testing-views)
    - [Отрисовка Blade и компоненты](#rendering-blade-and-components)
- [Доступные утверждения](#available-assertions)
    - [Утверждения ответов](#response-assertions)
    - [Утверждения аутентификации](#authentication-assertions)

<a name="introduction"></a>
## Введение

Laravel предоставляет очень гибкий API для выполнения HTTP-запросов к вашему приложению и проверки ответов. Например, взгляните на функциональный тест, расположенный ниже:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Базовый пример теста.
         *
         * @return void
         */
        public function test_a_basic_request()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

Метод `get` отправляет в приложение запрос `GET`, а метод `assertStatus` утверждает, что возвращаемый ответ должен иметь указанный код состояния HTTP. Помимо этого простого утверждения, Laravel также содержит множество утверждений для проверки заголовков ответов, содержимого, структуры JSON и многого другого.

<a name="making-requests"></a>
## Выполнение запросов

Чтобы сделать запрос к вашему приложению, вы можете вызвать в своем тесте методы `get`, `post`, `put`, `patch`, или `delete`. Эти методы фактически не отправляют вашему приложению «настоящий» HTTP-запрос. Вместо этого внутри моделируется полный сетевой запрос.

Вместо того, чтобы возвращать экземпляр `Illuminate\Http\Response`, методы тестового запроса возвращают экземпляр `Illuminate\Testing\TestResponse`, который предоставляет [множество полезных утверждений](#available-assertions), которые позволяют вам инспектировать ответы вашего приложения:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Базовый пример теста.
         *
         * @return void
         */
        public function test_a_basic_request()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

> {tip} Для удобства посредник CSRF автоматически отключается при запуске тестов.

<a name="customizing-request-headers"></a>
### Настройка заголовков запросов

Вы можете использовать метод `withHeaders` для настройки заголовков запроса перед его отправкой в ​​приложение. Этот метод позволяет вам добавлять в запрос любые пользовательские заголовки:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Пример базового функционального теста.
         *
         * @return void
         */
        public function test_interacting_with_headers()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->post('/user', ['name' => 'Sally']);

            $response->assertStatus(201);
        }
    }

<a name="cookies"></a>
### Cookies

Вы можете использовать методы `withCookie` или `withCookies` для установки значений файлов Cookies перед отправкой запроса. Метод `withCookie` принимает имя и значение Cookie в качестве двух аргументов, а метод `withCookies` принимает массив пар имя / значение:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_interacting_with_cookies()
        {
            $response = $this->withCookie('color', 'blue')->get('/');

            $response = $this->withCookies([
                'color' => 'blue',
                'name' => 'Taylor',
            ])->get('/');
        }
    }

<a name="session-and-authentication"></a>
### Сессия / Аутентификация

Laravel предоставляет несколько помощников для взаимодействия с сессией во время HTTP-тестирования. Во-первых, вы можете установить данные сессии, передав массив, используя метод `withSession`. Это полезно для загрузки сессии данными перед отправкой запроса вашему приложению:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_interacting_with_the_session()
        {
            $response = $this->withSession(['banned' => false])->get('/');
        }
    }

Сессия Laravel обычно используется для сохранения состояния текущего аутентифицированного пользователя. Таким образом, вспомогательный метод `actingAs` предоставляет простой способ аутентифицировать переданного пользователя как текущего. Например, мы можем использовать [фабрику модели](database-testing.md#writing-factories) для генерации и аутентификации пользователя:

    <?php

    namespace Tests\Feature;

    use App\Models\User;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_an_action_that_requires_authentication()
        {
            $user = User::factory()->create();

            $response = $this->actingAs($user)
                             ->withSession(['banned' => false])
                             ->get('/');
        }
    }

Вы также можете указать, какой охранник должен использоваться для аутентификации переданного пользователя, передав имя охранника в качестве второго аргумента методу `actingAs`:

    $this->actingAs($user, 'api')

<a name="debugging-responses"></a>
### Отладка ответов

После выполнения тестового запроса к вашему приложению методы `dump`, `dumpHeaders`, и `dumpSession` могут быть использованы для проверки и отладки содержимого ответа:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Базовый пример теста.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->dumpHeaders();

            $response->dumpSession();

            $response->dump();
        }
    }

<a name="testing-json-apis"></a>
## Тестирование JSON API

Laravel также содержит несколько помощников для тестирования API-интерфейсов JSON и их ответов. Например, методы `json`, `getJson`, `postJson`, `putJson`, `patchJson`, `deleteJson`, и `optionsJson` могут использоваться для отправки запросов JSON с различными HTTP-командами. Вы также можете легко передавать данные и заголовки этим методам. Для начала давайте напишем тест, чтобы сделать запрос `POST` к `/api/user` и убедиться, что были возвращены ожидаемые данные JSON:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Пример базового функционального теста.
         *
         * @return void
         */
        public function test_making_an_api_request()
        {
            $response = $this->postJson('/api/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

Кроме того, к данным ответа JSON можно получить доступ как к переменным массива в ответе, что позволяет удобно проверять отдельные значения, возвращаемые в JSON-ответе:

    $this->assertTrue($response['created']);

> {tip} Метод `assertJson` преобразует ответ в массив и использует `PHPUnit::assertArraySubset` для проверки того, что переданный массив существует в ответе JSON, возвращаемом приложением. Итак, если в ответе JSON есть другие свойства, этот тест все равно будет проходить, пока присутствует переданный фрагмент.

<a name="verifying-exact-match"></a>
#### Утверждение точных совпадений JSON

Как упоминалось ранее, метод `assertJson` может использоваться для подтверждения наличия фрагмента JSON в ответе JSON. Если вы хотите убедиться, что данный массив **в точности соответствует** JSON, возвращаемому вашим приложением, вы должны использовать метод `assertExactJson`:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Пример базового функционального теста.
         *
         * @return void
         */
        public function test_asserting_an_exact_json_match()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="verifying-json-paths"></a>
#### Утверждения JSON-путей

Если вы хотите убедиться, что ответ JSON содержит данные по указанному пути, вам следует использовать метод `assertJsonPath`:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Пример базового функционального теста.
         *
         * @return void
         */
        public function test_asserting_a_json_paths_value()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJsonPath('team.owner.name', 'Darian');
        }
    }

<a name="testing-file-uploads"></a>
## Тестирование загрузки файлов

Класс `Illuminate\Http\UploadedFile` предоставляет метод `fake`, который можно использовать для создания фиктивных файлов или изображений для тестирования. Это, в сочетании с методом `fake` фасада `Storage`, значительно упрощает тестирование загрузки файлов. Например, вы можете объединить эти две функции, чтобы легко протестировать форму загрузки аватара:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_avatars_can_be_uploaded()
        {
            Storage::fake('avatars');

            $file = UploadedFile::fake()->image('avatar.jpg');

            $response = $this->post('/avatar', [
                'avatar' => $file,
            ]);

            Storage::disk('avatars')->assertExists($file->hashName());
        }
    }

Если вы хотите подтвердить, что переданный файл не существует, вы можете использовать метод `assertMissing` фасада `Storage`:

    Storage::fake('avatars');

    // ...

    Storage::disk('avatars')->assertMissing('missing.jpg');

<a name="fake-file-customization"></a>
#### Настройка фиктивного файла

При создании файлов с использованием метода `fake`, предоставляемого классом `UploadedFile`, вы можете указать ширину, высоту и размер изображения (в килобайтах), чтобы лучше протестировать правила валидации вашего приложения:

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

Помимо создания изображений, вы можете создавать файлы любого другого типа, используя метод `create`:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

При необходимости вы можете передать аргумент `$mimeType` методу, чтобы явно определить MIME-тип, который должен возвращать файл:

    UploadedFile::fake()->create(
        'document.pdf', $sizeInKilobytes, 'application/pdf'
    );

<a name="testing-views"></a>
## Тестирование шаблонной системы

Laravel также позволяет отображать шаблоны без имитации HTTP-запроса к приложению. Для этого вы можете вызвать в своем тесте метод `view`. Метод `view` принимает имя шаблона и необязательный массив данных. Метод возвращает экземпляр `Illuminate\Testing\TestView`, который предлагает несколько методов для удобных утверждений о содержимом шаблона:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_a_welcome_view_can_be_rendered()
        {
            $view = $this->view('welcome', ['name' => 'Taylor']);

            $view->assertSee('Taylor');
        }
    }

Класс `TestView` содержит следующие методы утверждения: `assertSee`, `assertSeeInOrder`, `assertSeeText`, `assertSeeTextInOrder`, `assertDontSee` и `assertDontSeeText`.

При необходимости вы можете получить необработанное отрисованное содержимое шаблона, преобразовав экземпляр `TestView` в строку:

    $contents = (string) $this->view('welcome');

<a name="sharing-errors"></a>
#### Передача ошибок валидации в шаблоны

Некоторые шаблоны могут зависеть от ошибок, хранящихся в [глобальной коллекции ошибок Laravel](validation.md#quick-displaying-the-validation-errors). Чтобы добавить в эту коллекцию сообщения об ошибках, вы можете использовать метод `withViewErrors`:

    $view = $this->withViewErrors([
        'name' => ['Please provide a valid name.']
    ])->view('form');

    $view->assertSee('Please provide a valid name.');

<a name="rendering-blade-and-components"></a>
### Отрисовка Blade и компоненты

Если необходимо, вы можете использовать метод `blade` для анализа и отрисовки необработанной строки [Blade](blade.md). Подобно методу `view`, метод `blade` возвращает экземпляр `Illuminate\Testing\TestView`:

    $view = $this->blade(
        '<x-component :name="$name" />',
        ['name' => 'Taylor']
    );

    $view->assertSee('Taylor');

Вы можете использовать метод `component` для анализа и отрисовки [компонента Blade](blade.md#components). Как и метод `view`, метод `component` возвращает экземпляр `Illuminate\Testing\TestView`:

    $view = $this->component(Profile::class, ['name' => 'Taylor']);

    $view->assertSee('Taylor');

<a name="available-assertions"></a>
## Доступные утверждения

<a name="response-assertions"></a>
### Утверждения ответов

Класс `Illuminate\Testing\TestResponse` содержит множество своих методов утверждения, которые вы можете использовать при тестировании вашего приложения. К этим утверждениям можно получить доступ в ответе, возвращаемом тестовыми методами `json`, `get`, `post`, `put`, и `delete`:

<!-- <style>
    .collection-method-list > p {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style> -->

<!-- <div class="collection-method-list" markdown="1"> -->

- [assertCookie](#assert-cookie)
- [assertCookieExpired](#assert-cookie-expired)
- [assertCookieNotExpired](#assert-cookie-not-expired)
- [assertCookieMissing](#assert-cookie-missing)
- [assertCreated](#assert-created)
- [assertDontSee](#assert-dont-see)
- [assertDontSeeText](#assert-dont-see-text)
- [assertExactJson](#assert-exact-json)
- [assertForbidden](#assert-forbidden)
- [assertHeader](#assert-header)
- [assertHeaderMissing](#assert-header-missing)
- [assertJson](#assert-json)
- [assertJsonCount](#assert-json-count)
- [assertJsonFragment](#assert-json-fragment)
- [assertJsonMissing](#assert-json-missing)
- [assertJsonMissingExact](#assert-json-missing-exact)
- [assertJsonMissingValidationErrors](#assert-json-missing-validation-errors)
- [assertJsonPath](#assert-json-path)
- [assertJsonStructure](#assert-json-structure)
- [assertJsonValidationErrors](#assert-json-validation-errors)
- [assertLocation](#assert-location)
- [assertNoContent](#assert-no-content)
- [assertNotFound](#assert-not-found)
- [assertOk](#assert-ok)
- [assertPlainCookie](#assert-plain-cookie)
- [assertRedirect](#assert-redirect)
- [assertSee](#assert-see)
- [assertSeeInOrder](#assert-see-in-order)
- [assertSeeText](#assert-see-text)
- [assertSeeTextInOrder](#assert-see-text-in-order)
- [assertSessionHas](#assert-session-has)
- [assertSessionHasInput](#assert-session-has-input)
- [assertSessionHasAll](#assert-session-has-all)
- [assertSessionHasErrors](#assert-session-has-errors)
- [assertSessionHasErrorsIn](#assert-session-has-errors-in)
- [assertSessionHasNoErrors](#assert-session-has-no-errors)
- [assertSessionDoesntHaveErrors](#assert-session-doesnt-have-errors)
- [assertSessionMissing](#assert-session-missing)
- [assertStatus](#assert-status)
- [assertSuccessful](#assert-successful)
- [assertUnauthorized](#assert-unauthorized)
- [assertViewHas](#assert-view-has)
- [assertViewHasAll](#assert-view-has-all)
- [assertViewIs](#assert-view-is)
- [assertViewMissing](#assert-view-missing)

<!-- </div> -->

<a name="assert-cookie"></a>
#### assertCookie

Утверждает, что ответ содержит переданный cookie:

    $response->assertCookie($cookieName, $value = null);

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

Утверждает, что в ответе содержится переданный cookie и срок его действия истек:

    $response->assertCookieExpired($cookieName);

<a name="assert-cookie-not-expired"></a>
#### assertCookieNotExpired

Утверждает, что в ответе содержится переданный cookie и срок его действия не истек:

    $response->assertCookieNotExpired($cookieName);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Утверждает, что ответ не содержит переданный cookie:

    $response->assertCookieMissing($cookieName);

<a name="assert-created"></a>
#### assertCreated

Утверждает, что ответ имеет код `201` состояния HTTP:

    $response->assertCreated();

<a name="assert-dont-see"></a>
#### assertDontSee

Утверждает, что переданная строка не содержится в ответе, возвращаемом приложением. Это утверждение автоматически экранирует переданную строку, если вы не передадите второй аргумент как `false`:

    $response->assertDontSee($value, $escaped = true);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

Утверждает, что переданная строка не содержится в тексте ответа. Это утверждение автоматически экранирует переданную строку, если вы не передадите второй аргумент как `false`. Этот метод передаст содержимое ответа PHP-функции `strip_tags` перед тем, как выполнить утверждение:

    $response->assertDontSeeText($value, $escaped = true);

<a name="assert-exact-json"></a>
#### assertExactJson

Утверждает, что ответ содержит точное совпадение указанных данных JSON:

    $response->assertExactJson(array $data);

<a name="assert-forbidden"></a>
#### assertForbidden

Утверждает, что ответ имеет код `403` состояния HTTP – `forbidden`:

    $response->assertForbidden();

<a name="assert-header"></a>
#### assertHeader

Утверждает, что переданный заголовок и значение присутствуют в ответе:

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>
#### assertHeaderMissing

Утверждает, что переданный заголовок отсутствует в ответе:

    $response->assertHeaderMissing($headerName);

<a name="assert-json"></a>
#### assertJson

Утверждает, что ответ содержит указанные данные JSON:

    $response->assertJson(array $data, $strict = false);

Метод `assertJson` преобразует ответ в массив и использует `PHPUnit::assertArraySubset` для проверки того, что переданный массив существует в ответе JSON, возвращаемом приложением. Итак, если в ответе JSON есть другие свойства, этот тест все равно будет проходить, пока присутствует переданный фрагмент.

<a name="assert-json-count"></a>
#### assertJsonCount

Утверждает, что ответ JSON имеет массив с ожидаемым количеством элементов указанного ключа:

    $response->assertJsonCount($count, $key = null);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

Утверждает, что ответ содержит указанные данные JSON в любом месте ответа:

    Route::get('/users', function () {
        return [
            'users' => [
                [
                    'name' => 'Taylor Otwell',
                ],
            ],
        ];
    });

    $response->assertJsonFragment(['name' => 'Taylor Otwell']);

<a name="assert-json-missing"></a>
#### assertJsonMissing

Утверждает, что ответ не содержит указанных данных JSON:

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

Утверждает, что ответ не содержит точных указанных данных JSON:

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

Утверждает, что ответ не содержит ошибок валидации JSON для переданных ключей:

    $response->assertJsonMissingValidationErrors($keys);

<a name="assert-json-path"></a>
#### assertJsonPath

Утверждает, что ответ содержит конкретные данные по указанному пути:

    $response->assertJsonPath($path, array $data, $strict = true);

Например, если ответ JSON, возвращаемый вашим приложением, содержит следующие данные:

```js
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Вы можете утверждать, что свойство `name` объекта `user` соответствует переданному значению следующим образом:

    $response->assertJsonPath('user.name', 'Steve Schoger');

<a name="assert-json-structure"></a>
#### assertJsonStructure

Утверждает, что ответ имеет переданную структуру JSON:

    $response->assertJsonStructure(array $structure);

Например, если ответ JSON, возвращаемый вашим приложением, содержит следующие данные:

```js
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Вы можете утверждать, что структура JSON соответствует вашим ожиданиям, например:

    $response->assertJsonStructure([
        'user' => [
            'name',
        ]
    ]);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

Утверждает, что ответ содержит переданные ошибки валидации JSON для переданных ключей. Этот метод следует использовать при утверждении ответов, в которых ошибки валидации возвращаются как структура JSON, а не кратковременно передаются в сесиию:

    $response->assertJsonValidationErrors(array $data);

<a name="assert-location"></a>
#### assertLocation

Утверждает, что ответ имеет переданное значение URI в заголовке `Location`:

    $response->assertLocation($uri);

<a name="assert-no-content"></a>
#### assertNoContent

Утверждает, что ответ имеет код `204` состояния HTTP – `no content`:

    $response->assertNoContent($status = 204);

<a name="assert-not-found"></a>
#### assertNotFound

Утверждает, что ответ имеет код `404` состояния HTTP – `not found`:

    $response->assertNotFound();

<a name="assert-ok"></a>
#### assertOk

Утверждает, что ответ имеет код `200` состояния HTTP – `OK`:

    $response->assertOk();

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

Утверждает, что ответ содержит переданный незашифрованный cookie:

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### assertRedirect

Утверждает, что ответ является перенаправлением на указанный URI:

    $response->assertRedirect($uri);

<a name="assert-see"></a>
#### assertSee

Утверждает, что переданная строка содержится в ответе. Это утверждение автоматически экранирует переданную строку, если вы не передадите второй аргумент как `false`:

    $response->assertSee($value, $escaped = true);

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

Утверждает, что переданные строки содержатся в ответе в указанном порядке. Это утверждение автоматически экранирует переданные строки, если вы не передадите второй аргумент как `false`:

    $response->assertSeeInOrder(array $values, $escaped = true);

<a name="assert-see-text"></a>
#### assertSeeText

Утверждает, что переданная строка содержится в тексте ответа. Это утверждение автоматически экранирует переданную строку, если вы не передадите второй аргумент как `false`. Этот метод передаст содержимое ответа PHP-функции `strip_tags` перед тем, как выполнить утверждение:

    $response->assertSeeText($value, $escaped = true);

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

Утверждает, что переданные строки содержатся в тексте ответа в указанном порядке. Это утверждение автоматически экранирует переданные строки, если вы не передадите второй аргумент как `false`. Этот метод передаст содержимое ответа PHP-функции `strip_tags` перед тем, как выполнить утверждение:

    $response->assertSeeTextInOrder(array $values, $escaped = true);

<a name="assert-session-has"></a>
#### assertSessionHas

Утверждает, что сессия содержит переданный фрагмент данных:

    $response->assertSessionHas($key, $value = null);

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

Утверждает, что сессия имеет переданное значение в [массиве входящих данных кратковременного сохранения](responses.md#redirecting-with-flashed-session-data):

    $response->assertSessionHasInput($key, $value = null);

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

Утверждает, что сессия содержит переданный массив пар ключ / значение:

    $response->assertSessionHasAll(array $data);

Например, если сессия вашего приложения содержит ключи `name` и `status`, вы можете утверждать, что оба они существуют и имеют указанные значения, например:

    $response->assertSessionHasAll([
        'name' => 'Taylor Otwell',
        'status' => 'active',
    ]);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

Утверждает, что сессия содержит ошибку для переданных `$keys`. Если `$keys` является ассоциативным массивом, следует утверждать, что сессия содержит конкретное сообщение об ошибке (значение) для каждого поля (ключа). Этот метод следует использовать при тестировании маршрутов, которые передают ошибки валидации в сессию вместо того, чтобы возвращать их в виде структуры JSON:

    $response->assertSessionHasErrors(
        array $keys, $format = null, $errorBag = 'default'
    );

Например, чтобы утверждать, что поля `name` и `email` содержат сообщения об ошибках валидации, которые были переданы в сессию, вы можете вызвать метод `assertSessionHasErrors` следующим образом:

    $response->assertSessionHasErrors(['name', 'email']);

Или вы можете утверждать, что переданное поле имеет конкретное сообщение об ошибке валидации:

    $response->assertSessionHasErrors([
        'name' => 'The given name was invalid.'
    ]);

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

Утверждает, что сессия содержит ошибку для переданных `$keys` в конкретной [коллекции ошибок](validation.md#named-error-bags). Если `$keys` является ассоциативным массивом, убедитесь, что сессия содержит конкретное сообщение об ошибке (значение) для каждого поля (ключа) в коллекции ошибок:

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

Утверждает, что в сессии нет ошибок валидации:

    $response->assertSessionHasNoErrors();

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

Утверждает, что в сессии нет ошибок валидации для переданных ключей:

    $response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');

<a name="assert-session-missing"></a>
#### assertSessionMissing

Утверждает, что сессия не содержит переданного ключа:

    $response->assertSessionMissing($key);

<a name="assert-status"></a>
#### assertStatus

Утверждает, что ответ имеет указанный код `$code` состояния HTTP:

    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

Утверждает, что ответ имеет код `>= 200` и `< 300` состояния HTTP – `successful`:

    $response->assertSuccessful();

<a name="assert-unauthorized"></a>
#### assertUnauthorized

Утверждает, что ответ имеет код `401` состояния HTTP – `unauthorized`:

    $response->assertUnauthorized();

<a name="assert-view-has"></a>
#### assertViewHas

Утверждает, что шаблон ответа содержит переданный фрагмент данных:

    $response->assertViewHas($key, $value = null);

Кроме того, данные шаблона могут быть доступны как переменные массива в ответе, что позволяет вам удобно инспектировать их:

    $this->assertEquals('Taylor', $response['name']);

<a name="assert-view-has-all"></a>
#### assertViewHasAll

Утверждает, что в шаблон ответа содержит переданный список данных:

    $response->assertViewHasAll(array $data);

Этот метод может использоваться, чтобы утверждать, что шаблон просто содержит данные с соответствующими переданными ключами:

    $rseponse->assertViewHasAll([
        'name',
        'email',
    ]);

Или вы можете утверждать, что данные шаблона присутствуют и имеют определенные значения:

    $rseponse->assertViewHasAll([
        'name' => 'Taylor Otwell',
        'email' => 'taylor@example.com,',
    ]);

<a name="assert-view-is"></a>
#### assertViewIs

Утверждает, что маршрутом был возвращен указанный шаблон:

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

Утверждает, что переданный ключ данных не был доступен для шаблона, возвращенного ответом приложения:

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>
### Утверждения аутентификации

Laravel также содержит множество утверждений, связанных с аутентификацией, которые вы можете использовать в функциональных тестах вашего приложения. Обратите внимание, что эти методы вызываются в самом тестовом классе, а не в экземпляре `Illuminate\Testing\TestResponse`, возвращаемом такими методами, как `get` и `post`.

<a name="assert-authenticated"></a>
#### assertAuthenticated

Утверждает, что пользователь аутентифицирован:

    $this->assertAuthenticated($guard = null);

<a name="assert-guest"></a>
#### assertGuest

Утверждает, что пользователь не аутентифицирован:

    $this->assertGuest($guard = null);

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

Утверждает, что конкретный пользователь аутентифицирован:

    $this->assertAuthenticatedAs($user, $guard = null);
