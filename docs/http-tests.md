# Laravel 9 · Тестирование · Тесты HTTP

- [Введение](#introduction)
- [Выполнение запросов](#making-requests)
    - [Настройка заголовков запросов](#customizing-request-headers)
    - [Cookies](#cookies)
    - [Сессия / Аутентификация](#session-and-authentication)
    - [Отладка ответов](#debugging-responses)
    - [Обработка исключений](#exception-handling)
- [Тестирование JSON API](#testing-json-apis)
    - [Последовательное тестирование JSON](#fluent-json-testing)
- [Тестирование загрузки файлов](#testing-file-uploads)
- [Тестирование шаблонной системы](#testing-views)
    - [Отрисовка Blade и компоненты](#rendering-blade-and-components)
- [Доступные утверждения](#available-assertions)
    - [Утверждения ответов](#response-assertions)
    - [Утверждения аутентификации](#authentication-assertions)

<a name="introduction"></a>
## Введение

Laravel предлагает гибкий API в составе вашего приложения для выполнения запросов HTTP и получения информации об ответах. Например, взгляните на следующий функциональный тест:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Отвлеченный пример функционального теста.
         *
         * @return void
         */
        public function test_a_basic_request()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

Метод `get` отправляет в приложение запрос `GET`, а метод `assertStatus` проверяет утверждение о том, что возвращаемый ответ имеет указанный код состояния HTTP. Помимо этого простого утверждения, Laravel также содержит множество утверждений для получения информации о заголовках ответов, их содержимого, структуры JSON и др.

<a name="making-requests"></a>
## Выполнение запросов

Чтобы сделать запрос к вашему приложению, вы можете вызвать в своем тесте методы `get`, `post`, `put`, `patch` или `delete`. Эти методы фактически не отправляют вашему приложению «настоящий» HTTP-запрос. Вместо этого внутри моделируется полный сетевой запрос.

Вместо того, чтобы возвращать экземпляр `Illuminate\Http\Response`, методы тестового запроса возвращают экземпляр `Illuminate\Testing\TestResponse`, который содержит [множество полезных утверждений](#available-assertions), которые позволяют вам инспектировать ответы вашего приложения:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Отвлеченный пример функционального теста.
         *
         * @return void
         */
        public function test_a_basic_request()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

В общем, каждый из ваших тестов должен делать только один запрос к вашему приложению. Выполнение нескольких запросов в одном тестовом методе может спровоцировать непредвиденное поведение.

> {tip} Для удобства посредник CSRF автоматически отключается при запуске тестов.

<a name="customizing-request-headers"></a>
### Настройка заголовков запросов

Вы можете использовать метод `withHeaders` для настройки заголовков запроса перед его отправкой в приложение. Этот метод позволяет вам добавлять в запрос любые пользовательские заголовки:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Отвлеченный пример функционального теста.
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

Laravel предлагает несколько помощников для взаимодействия с сессией во время HTTP-тестирования. Во-первых, вы можете установить данные сессии, передав массив, используя метод `withSession`. Это полезно для загрузки сессии данными перед отправкой запроса вашему приложению:

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

Сессия Laravel обычно используется для сохранения состояния текущего аутентифицированного пользователя. Вспомогательный метод `actingAs` – это простой способ аутентифицировать конкретного пользователя как текущего. Например, мы можем использовать [фабрику модели](database-testing.md#defining-model-factories) для генерации и аутентификации пользователя:

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

Вы также можете указать, какой охранник должен использоваться для аутентификации конкретного пользователя, передав имя охранника в качестве второго аргумента методу `actingAs`:

    $this->actingAs($user, 'web')

<a name="debugging-responses"></a>
### Отладка ответов

После выполнения тестового запроса к вашему приложению методы `dump`, `dumpHeaders` и `dumpSession` могут быть использованы для проверки и отладки содержимого ответа:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Отвлеченный пример функционального теста.
         *
         * @return void
         */
        public function test_basic_test()
        {
            $response = $this->get('/');

            $response->dumpHeaders();

            $response->dumpSession();

            $response->dump();
        }
    }

В качестве альтернативы вы можете использовать методы `dd`, `ddHeaders` и `ddSession` для вывода информации об ответе и дальнейшей остановки выполнения теста:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Отвлеченный пример функционального теста.
         *
         * @return void
         */
        public function test_basic_test()
        {
            $response = $this->get('/');

            $response->ddHeaders();

            $response->ddSession();

            $response->dd();
        }
    }

<a name="exception-handling"></a>
### Обработка исключений

Иногда требуется тестирование конкретного исключения, выбрасываемого вашим приложение. Чтобы исключение не было перехвачено обработчиком исключений Laravel и не было возвращено в виде ответа HTTP, вы можете вызвать метод `withoutExceptionHandling` перед отправкой запроса:

    $response = $this->withoutExceptionHandling()->get('/');

Кроме того, если вы хотите убедиться, что ваше приложение не использует функции, которые являются устаревшими в контексте языка PHP или библиотек, которые использует ваше приложение, то вы можете вызвать метод `withoutDeprecationHandling` перед тем, как сделать свой запрос. Когда обработка устаревших методов отключена, то такие предупреждения будут преобразованы в исключения, что приведет к сбою вашего теста:

    $response = $this->withoutDeprecationHandling()->get('/');

<a name="testing-json-apis"></a>
## Тестирование JSON API

Laravel также содержит несколько помощников для тестирования API-интерфейсов JSON и их ответов. Например, методы `json`, `getJson`, `postJson`, `putJson`, `patchJson`, `deleteJson` и `optionsJson` могут использоваться для отправки запросов JSON с различными HTTP-командами. Вы также можете легко передавать данные и заголовки этим методам. Для начала давайте напишем тест, чтобы сделать запрос `POST` к `/api/user` и убедиться, что были возвращены ожидаемые данные JSON:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Отвлеченный пример функционального теста.
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

Как упоминалось ранее, метод `assertJson` используется для подтверждения наличия фрагмента JSON в ответе JSON. Если вы хотите убедиться, что данный массив **в точности соответствует** JSON, возвращаемому вашим приложением, вы должны использовать метод `assertExactJson`:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * Отвлеченный пример функционального теста.
         *
         * @return void
         */
        public function test_asserting_an_exact_json_match()
        {
            $response = $this->postJson('/user', ['name' => 'Sally']);

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
         * Отвлеченный пример функционального теста.
         *
         * @return void
         */
        public function test_asserting_a_json_paths_value()
        {
            $response = $this->postJson('/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJsonPath('team.owner.name', 'Darian');
        }
    }

Метод `assertJsonPath` также принимает замыкание, которое можно использовать для динамического определения утверждения:

    $response->assertJsonPath('team.owner.name', fn ($name) => strlen($name) >= 3);

<a name="fluent-json-testing"></a>
### Последовательное тестирование JSON

Laravel также предлагает прекрасный способ последовательного тестирования ответов JSON вашего приложения. Для начала передайте замыкание методу `assertJson`. Это замыкание будет вызываться с экземпляром класса `Illuminate\Testing\Fluent\AssertableJson`, который можно использовать для создания утверждений в отношении JSON, возвращенного вашим приложением. Метод `where` может использоваться для утверждения определенного атрибута JSON, в то время как метод `missing` может использоваться для утверждения отсутствия конкретного атрибута в JSON:

    use Illuminate\Testing\Fluent\AssertableJson;

    /**
     * Отвлеченный пример функционального теста.
     *
     * @return void
     */
    public function test_fluent_json()
    {
        $response = $this->getJson('/users/1');

        $response
            ->assertJson(fn (AssertableJson $json) =>
                $json->where('id', 1)
                     ->where('name', 'Victoria Faith')
                     ->missing('password')
                     ->etc()
            );
    }

#### Понимание метода `etc`

В приведенном выше примере вы могли заметить, что мы вызвали метод `etc` в конце нашей цепочки утверждений. Этот метод сообщает Laravel, что в объекте JSON могут присутствовать другие атрибуты. Если метод `etc` не используется, то тест завершится неудачно, если в объекте JSON существуют другие атрибуты, для которых вы не сделали утверждений.

Цель такого поведения – защитить вас от непреднамеренного раскрытия конфиденциальной информации в ваших ответах JSON, заставив вас либо явно сделать утверждение относительно атрибута, либо явно разрешить дополнительные атрибуты с помощью метода `etc`.

<a name="asserting-json-attribute-presence-and-absence"></a>
#### Утверждения относительно наличия / отсутствия атрибута JSON

Для утверждений, что атрибут присутствует или отсутствует, вы можете использовать методы `has` и `missing`:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('data')
             ->missing('message')
    );

Кроме того, методы `hasAll` и `missingAll` позволяют одновременно делать утверждения о наличии или отсутствия нескольких атрибутов:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->hasAll(['status', 'data'])
             ->missingAll(['message', 'code'])
    );

Вы можете использовать метод `hasAny`, чтобы определить, присутствует ли хотя бы один атрибут, из указанных в списке:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('status')
             ->hasAny('data', 'message', 'code')
    );

<a name="asserting-against-json-collections"></a>
#### Утверждения относительно коллекций JSON

Часто ваш маршрут возвращает ответ JSON, содержащий несколько элементов, например нескольких пользователей:

    Route::get('/users', function () {
        return User::all();
    });

В этих ситуациях можно использовать метод `has` последовательного тестирования JSON, чтобы сделать утверждения относительно пользователей, содержащихся в ответе. Например, предположим, что ответ JSON содержит трех пользователей. Затем мы сделаем некоторые утверждения относительно первого пользователя в коллекции, используя метод `first`. Метод `first` принимает замыкание, получающее другой экземпляр `AssertableJson`, который можно использовать для создания утверждений относительно первого объекта коллекции JSON:

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has(3)
                 ->first(fn ($json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->missing('password')
                         ->etc()
                 )
        );

<a name="scoping-json-collection-assertions"></a>
#### Уровень вложенности утверждения относительно коллекций JSON

Иногда маршрутами вашего приложения могут быть возвращены коллекции JSON, которым назначены именованные ключи:

    Route::get('/users', function () {
        return [
            'meta' => [...],
            'users' => User::all(),
        ];
    })

При тестировании этих маршрутов вы можете использовать метод `has` для утверждения относительно количества элементов в коллекции. Кроме того, вы можете использовать метод `has` для определения цепочки утверждений:

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has('meta')
                 ->has('users', 3)
                 ->has('users.0', fn ($json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->missing('password')
                         ->etc()
                 )
        );

Однако вместо того, чтобы делать два отдельных вызова метода `has` для утверждения в отношении коллекции `users`, вы можете сделать один вызов, обеспеченный замыканием в качестве третьего параметра. При этом автоматически вызывается замыкание, область действия которого будет ограниченно уровнем вложенности первого элемента коллекции:

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has('meta')
                 ->has('users', 3, fn ($json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->missing('password')
                         ->etc()
                 )
        );

<a name="asserting-json-types"></a>
#### Утверждения относительно типов JSON

При необходимости можно утверждать, что свойства в ответе JSON имеют определенный тип. Класс `Illuminate\Testing\Fluent\AssertableJson` содержит методы `whereType` и `whereAllType`, обеспечивающие простоту таких утверждений:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->whereType('id', 'integer')
             ->whereAllType([
                'users.0.name' => 'string',
                'meta' => 'array'
            ])
    );

Можно указать несколько типов в качестве второго параметра метода `whereType`, разделив их символом `|`, или передав массив необходимых типов. Утверждение будет успешно, если значение ответа будет иметь какой-либо из перечисленных типов:

    $response->assertJson(fn (AssertableJson $json) =>
        $json->whereType('name', 'string|null')
             ->whereType('id', ['string', 'integer'])
    );

Методы `whereType` и `whereAllType` применимы к следующим типам: `string`, `integer`, `double`, `boolean`, `array` и `null`.

<a name="testing-file-uploads"></a>
## Тестирование загрузки файлов

Класс `Illuminate\Http\UploadedFile` содержит метод `fake`, который можно использовать для создания фиктивных файлов или изображений для тестирования. Это, в сочетании с методом `fake` фасада `Storage`, значительно упрощает тестирование загрузки файлов. Например, вы можете объединить эти две функции, чтобы легко протестировать форму загрузки аватара:

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

Класс `Illuminate\Testing\TestResponse` содержит множество своих методов утверждения, которые вы можете использовать при тестировании вашего приложения. К этим утверждениям можно получить доступ в ответе, возвращаемом тестовыми методами `json`, `get`, `post`, `put` и `delete`:

<!-- <style>
    .collection-method-list > p {
        columns: 14.4em 2; -moz-columns: 14.4em 2; -webkit-columns: 14.4em 2;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
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
- [assertDownload](#assert-download)
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
- [assertJsonMissingPath](#assert-json-missing-path)
- [assertJsonStructure](#assert-json-structure)
- [assertJsonValidationErrors](#assert-json-validation-errors)
- [assertJsonValidationErrorFor](#assert-json-validation-error-for)
- [assertLocation](#assert-location)
- [assertNoContent](#assert-no-content)
- [assertNotFound](#assert-not-found)
- [assertOk](#assert-ok)
- [assertPlainCookie](#assert-plain-cookie)
- [assertRedirect](#assert-redirect)
- [assertRedirectContains](#assert-redirect-contains)
- [assertRedirectToSignedRoute](#assert-redirect-to-signed-route)
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
- [assertUnprocessable](#assert-unprocessable)
- [assertValid](#assert-valid)
- [assertInvalid](#assert-invalid)
- [assertViewHas](#assert-view-has)
- [assertViewHasAll](#assert-view-has-all)
- [assertViewIs](#assert-view-is)
- [assertViewMissing](#assert-view-missing)

<!-- </div> -->

<a name="assert-cookie"></a>
#### assertCookie

Утверждение о том, что ответ содержит переданный cookie:

    $response->assertCookie($cookieName, $value = null);

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

Утверждение о том, что в ответе содержится переданный cookie и срок его действия истек:

    $response->assertCookieExpired($cookieName);

<a name="assert-cookie-not-expired"></a>
#### assertCookieNotExpired

Утверждение о том, что в ответе содержится переданный cookie и срок его действия не истек:

    $response->assertCookieNotExpired($cookieName);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Утверждение о том, что ответ не содержит переданный cookie:

    $response->assertCookieMissing($cookieName);

<a name="assert-created"></a>
#### assertCreated

Утверждение о том, что ответ имеет код `201` состояния HTTP:

    $response->assertCreated();

<a name="assert-dont-see"></a>
#### assertDontSee

Утверждение о том, что переданная строка не содержится в ответе, возвращаемом приложением. Это утверждение автоматически экранирует переданную строку, если вы не передадите второй аргумент как `false`:

    $response->assertDontSee($value, $escaped = true);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

Утверждение о том, что переданная строка не содержится в тексте ответа. Это утверждение автоматически экранирует переданную строку, если вы не передадите второй аргумент как `false`. Этот метод передаст содержимое ответа PHP-функции `strip_tags` перед тем, как выполнить утверждение:

    $response->assertDontSeeText($value, $escaped = true);

<a name="assert-download"></a>
#### assertDownload

Утверждение о том, что ответ является «загрузкой». Обычно это означает, что был вызван маршрут, который вернул ответ `Response::download`, `BinaryFileResponse` или `Storage::download`:

    $response->assertDownload();

При желании вы можете утверждать, что загружаемому файлу было присвоено указанное имя:

    $response->assertDownload('image.jpg');

<a name="assert-exact-json"></a>
#### assertExactJson

Утверждение о том, что ответ содержит точное совпадение указанных данных JSON:

    $response->assertExactJson(array $data);

<a name="assert-forbidden"></a>
#### assertForbidden

Утверждение о том, что ответ имеет код `403` состояния HTTP – `forbidden`:

    $response->assertForbidden();

<a name="assert-header"></a>
#### assertHeader

Утверждение о том, что переданный заголовок и значение присутствуют в ответе:

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>
#### assertHeaderMissing

Утверждение о том, что переданный заголовок отсутствует в ответе:

    $response->assertHeaderMissing($headerName);

<a name="assert-json"></a>
#### assertJson

Утверждение о том, что ответ содержит указанные данные JSON:

    $response->assertJson(array $data, $strict = false);

Метод `assertJson` преобразует ответ в массив и использует `PHPUnit::assertArraySubset` для проверки того, что переданный массив существует в ответе JSON, возвращаемом приложением. Итак, если в ответе JSON есть другие свойства, этот тест все равно будет проходить, пока присутствует переданный фрагмент.

<a name="assert-json-count"></a>
#### assertJsonCount

Утверждение о том, что ответ JSON имеет массив с ожидаемым количеством элементов указанного ключа:

    $response->assertJsonCount($count, $key = null);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

Утверждение о том, что ответ содержит указанные данные JSON в любом месте ответа:

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

Утверждение о том, что ответ не содержит указанных данных JSON:

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

Утверждение о том, что ответ не содержит точных указанных данных JSON:

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

Утверждение о том, что ответ не содержит ошибок валидации JSON для переданных ключей:

    $response->assertJsonMissingValidationErrors($keys);

> {tip} Более общий метод [assertValid](#assert-valid) может использоваться для утверждения о том, что ответ не содержит ошибок валидации, которые были возвращены как JSON **и**, что ошибки не были записаны в сессию.

<a name="assert-json-path"></a>
#### assertJsonPath

Утверждение о том, что ответ содержит конкретные данные по указанному пути:

    $response->assertJsonPath($path, $expectedValue);

Например, если ваше приложение возвращает следующий ответ JSON:

```js
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Вы можете утверждать, что свойство `name` объекта `user` соответствует переданному значению следующим образом:

    $response->assertJsonPath('user.name', 'Steve Schoger');

<a name="assert-json-missing-path"></a>
#### assertJsonMissingPath

Утверждение о том, что ответ не содержит указанный путь:

    $response->assertJsonMissingPath($path);

Например, если ваше приложение возвращает следующий ответ JSON:

```js
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Вы можете утверждать, что ответ не содержит свойства `email` объекта `user`:

    $response->assertJsonMissingPath('user.email');

<a name="assert-json-structure"></a>
#### assertJsonStructure

Утверждение о том, что ответ имеет переданную структуру JSON:

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

Иногда ответы JSON, возвращаемые вашим приложением, могут содержать массивы объектов:

```js
{
    "user": [
        {
            "name": "Steve Schoger",
            "age": 55,
            "location": "Earth"
        },
        {
            "name": "Mary Schoger",
            "age": 60,
            "location": "Earth"
        }
    ]
}
```

В этой ситуации вы можете использовать метасимвол `*` для утверждения о структуре каждого объекта в массиве:

    $response->assertJsonStructure([
        'user' => [
            '*' => [
                 'name',
                 'age',
                 'location'
            ]
        ]
    ]);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

Утверждение о том, что ответ содержит переданные ошибки валидации JSON для переданных ключей. Этот метод следует использовать при утверждении ответов, в которых ошибки валидации возвращаются как структура JSON, а не кратковременно передаются в сессию:

    $response->assertJsonValidationErrors(array $data, $responseKey = 'errors');

> {tip} Более общий метод [assertInvalid](#assert-invalid) может использоваться для утверждения о том, что ответ содержит ошибки валидации, которые были возвращены как JSON **или**, что ошибки были записаны в сессию.

<a name="assert-json-validation-error-for"></a>
#### assertJsonValidationErrorFor

Утверждение о том, что ответ содержит какие-либо ошибки валидации JSON для указанного ключа:

    $response->assertJsonValidationErrorFor(string $key, $responseKey = 'errors');

<a name="assert-location"></a>
#### assertLocation

Утверждение о том, что ответ имеет переданное значение URI в заголовке `Location`:

    $response->assertLocation($uri);

<a name="assert-no-content"></a>
#### assertNoContent

Утверждение о том, что ответ имеет код `204` состояния HTTP – `no content`:

    $response->assertNoContent($status = 204);

<a name="assert-not-found"></a>
#### assertNotFound

Утверждение о том, что ответ имеет код `404` состояния HTTP – `not found`:

    $response->assertNotFound();

<a name="assert-ok"></a>
#### assertOk

Утверждение о том, что ответ имеет код `200` состояния HTTP – `OK`:

    $response->assertOk();

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

Утверждение о том, что ответ содержит переданный незашифрованный cookie:

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### assertRedirect

Утверждение о том, что ответ является перенаправлением на указанный URI:

    $response->assertRedirect($uri);

<a name="assert-redirect-contains"></a>
#### assertRedirectContains

Утверждение о том, что ответ является перенаправлением на URI, который содержит указанную строку:

    $response->assertRedirectContains($string);

<a name="assert-redirect-to-signed-route"></a>
#### assertRedirectToSignedRoute

Утверждение о том, что ответ является перенаправлением на указанный [подписанный маршрут](urls.md#signed-urls):

    $response->assertRedirectToSignedRoute($name = null, $parameters = []);

<a name="assert-see"></a>
#### assertSee

Утверждение о том, что переданная строка содержится в ответе. Это утверждение автоматически экранирует переданную строку, если вы не передадите второй аргумент как `false`:

    $response->assertSee($value, $escaped = true);

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

Утверждение о том, что переданные строки содержатся в ответе в указанном порядке. Это утверждение автоматически экранирует переданные строки, если вы не передадите второй аргумент как `false`:

    $response->assertSeeInOrder(array $values, $escaped = true);

<a name="assert-see-text"></a>
#### assertSeeText

Утверждение о том, что переданная строка содержится в тексте ответа. Это утверждение автоматически экранирует переданную строку, если вы не передадите второй аргумент как `false`. Этот метод передаст содержимое ответа PHP-функции `strip_tags` перед тем, как выполнить утверждение:

    $response->assertSeeText($value, $escaped = true);

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

Утверждение о том, что переданные строки содержатся в тексте ответа в указанном порядке. Это утверждение автоматически экранирует переданные строки, если вы не передадите второй аргумент как `false`. Этот метод передаст содержимое ответа PHP-функции `strip_tags` перед тем, как выполнить утверждение:

    $response->assertSeeTextInOrder(array $values, $escaped = true);

<a name="assert-session-has"></a>
#### assertSessionHas

Утверждение о том, что сессия содержит переданный фрагмент данных:

    $response->assertSessionHas($key, $value = null);

При необходимости в качестве второго аргумента метода `assertSessionHas` может быть передано замыкание. Утверждение будет пройдено, если замыкание вернет `true`:

    $response->assertSessionHas($key, function ($value) {
        return $value->name === 'Taylor Otwell';
    });

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

Утверждение о том, что сессия имеет переданное значение в [массиве кратковременно хранящихся входных данных](responses.md#redirecting-with-flashed-session-data):

    $response->assertSessionHasInput($key, $value = null);

При необходимости в качестве второго аргумента метода `assertSessionHasInput` может быть передано замыкание. Утверждение будет пройдено, если замыкание вернет `true`:

    $response->assertSessionHasInput($key, function ($value) {
        return Crypt::decryptString($value) === 'secret';
    });

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

Утверждение о том, что сессия содержит переданный массив пар ключ / значение:

    $response->assertSessionHasAll(array $data);

Например, если сессия вашего приложения содержит ключи `name` и `status`, вы можете утверждать, что оба они существуют и имеют указанные значения, например:

    $response->assertSessionHasAll([
        'name' => 'Taylor Otwell',
        'status' => 'active',
    ]);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

Утверждение о том, что сессия содержит ошибку для переданных `$keys`. Если `$keys` является ассоциативным массивом, следует утверждать, что сессия содержит конкретное сообщение об ошибке (значение) для каждого поля (ключа). Этот метод следует использовать при тестировании маршрутов, которые передают ошибки валидации в сессию вместо того, чтобы возвращать их в виде структуры JSON:

    $response->assertSessionHasErrors(
        array $keys, $format = null, $errorBag = 'default'
    );

Например, чтобы утверждать, что поля `name` и `email` содержат сообщения об ошибках валидации, которые были переданы в сессию, вы можете вызвать метод `assertSessionHasErrors` следующим образом:

    $response->assertSessionHasErrors(['name', 'email']);

Или вы можете утверждать, что переданное поле имеет конкретное сообщение об ошибке валидации:

    $response->assertSessionHasErrors([
        'name' => 'The given name was invalid.'
    ]);

> {tip} Более общий метод [`assertInvalid`](#assert-invalid) может использоваться для утверждения о том, что в ответе есть ошибки валидации, которые были возвращены в виде структуры JSON **или**, что ошибки были переданы в хранилище сессии.

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

Утверждение о том, что сессия содержит ошибку для переданных `$keys` в конкретной [коллекции ошибок](validation.md#named-error-bags). Если `$keys` является ассоциативным массивом, убедитесь, что сессия содержит конкретное сообщение об ошибке (значение) для каждого поля (ключа) в коллекции ошибок:

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

Утверждение о том, что в сессии нет ошибок валидации:

    $response->assertSessionHasNoErrors();

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

Утверждение о том, что в сессии нет ошибок валидации для переданных ключей:

    $response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');

> {tip} Более общий метод [`assertValid`](#assert-valid) может использоваться для утверждения о том, что в ответе нет ошибок валидации, которые были возвращены в виде структуры JSON **и**, что ошибки не были переданы в хранилище сессии.

<a name="assert-session-missing"></a>
#### assertSessionMissing

Утверждение о том, что сессия не содержит переданного ключа:

    $response->assertSessionMissing($key);

<a name="assert-status"></a>
#### assertStatus

Утверждение о том, что ответ имеет указанный код `$code` состояния HTTP:

    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

Утверждение о том, что ответ имеет код `>= 200` и `< 300` состояния HTTP – `successful`:

    $response->assertSuccessful();

<a name="assert-unauthorized"></a>
#### assertUnauthorized

Утверждение о том, что ответ имеет код `401` состояния HTTP – `unauthorized`:

    $response->assertUnauthorized();

<a name="assert-unprocessable"></a>
#### assertUnprocessable

Утверждение о том, что ответ имеет код `422` состояния HTTP – `Unprocessable Entity`:

    $response->assertUnprocessable();

<a name="assert-valid"></a>
#### assertValid

Утверждение о том, что ответ не содержит ошибок валидации для переданных ключей. Этот метод может использоваться для утверждений относительно ответов, в которых ошибки валидации возвращаются в виде структуры JSON или где ошибки валидации были переданы в сессию:

    // Утверждение об отсутствии ошибок валидации ...
    $response->assertValid();

    // Утверждение об отсутствии ошибок валидации для переданных ключей ...
    $response->assertValid(['name', 'email']);

<a name="assert-invalid"></a>
#### assertInvalid

Утверждение о том, что ответ содержит ошибки валидации для переданных ключей. Этот метод может использоваться для утверждений относительно ответов, в которых ошибки валидации возвращаются в виде структуры JSON или где ошибки валидации были переданы в сессию:

    $response->assertInvalid(['name', 'email']);

Вы также можете использовать утверждение, что переданный ключ имеет конкретное сообщение об ошибке валидации. При этом вы можете передать как сообщение полностью, так и его небольшую часть:

    $response->assertInvalid([
        'name' => 'The name field is required.',
        'email' => 'valid email address',
    ]);

<a name="assert-view-has"></a>
#### assertViewHas

Утверждение о том, что шаблон ответа содержит переданный фрагмент данных:

    $response->assertViewHas($key, $value = null);

Передача замыкания в качестве второго аргумента методу `assertViewHas` позволит вам выполнять утверждения в отношении определенной части данных представления:

    $response->assertViewHas('user', function (User $user) {
        return $user->name === 'Taylor';
    });

Кроме того, данные шаблона могут быть доступны как переменные массива в ответе, что позволяет вам удобно инспектировать их:

    $this->assertEquals('Taylor', $response['name']);

<a name="assert-view-has-all"></a>
#### assertViewHasAll

Утверждение о том, что шаблон ответа содержит переданный список данных:

    $response->assertViewHasAll(array $data);

Этот метод может использоваться, чтобы утверждать, что шаблон просто содержит данные с соответствующими переданными ключами:

    $response->assertViewHasAll([
        'name',
        'email',
    ]);

Или вы можете утверждать, что данные шаблона присутствуют и имеют определенные значения:

    $response->assertViewHasAll([
        'name' => 'Taylor Otwell',
        'email' => 'taylor@example.com,',
    ]);

<a name="assert-view-is"></a>
#### assertViewIs

Утверждение о том, что маршрутом был возвращен указанный шаблон:

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

Утверждение о том, что переданный ключ данных не был доступен для шаблона, возвращенного ответом приложения:

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>
### Утверждения аутентификации

Laravel также содержит множество утверждений, связанных с аутентификацией, которые вы можете использовать в функциональных тестах вашего приложения. Обратите внимание, что эти методы вызываются в самом тестовом классе, а не в экземпляре `Illuminate\Testing\TestResponse`, возвращаемом такими методами, как `get` и `post`.

<a name="assert-authenticated"></a>
#### assertAuthenticated

Утверждение о том, что пользователь аутентифицирован:

    $this->assertAuthenticated($guard = null);

<a name="assert-guest"></a>
#### assertGuest

Утверждение о том, что пользователь не аутентифицирован:

    $this->assertGuest($guard = null);

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

Утверждение о том, что конкретный пользователь аутентифицирован:

    $this->assertAuthenticatedAs($user, $guard = null);
