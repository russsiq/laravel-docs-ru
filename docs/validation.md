# Laravel 9 · Валидация

- [Введение](#introduction)
- [Быстрый старт](#validation-quickstart)
    - [Определение маршрутов](#quick-defining-the-routes)
    - [Создание контроллера](#quick-creating-the-controller)
    - [Написание логики валидации](#quick-writing-the-validation-logic)
    - [Отображение ошибок валидации](#quick-displaying-the-validation-errors)
    - [Повторное заполнение форм](#repopulating-forms)
    - [Примечание о необязательных полях](#a-note-on-optional-fields)
    - [Формат ответа ошибок валидации](#validation-error-response-format)
- [Валидация запроса формы](#form-request-validation)
    - [Создание запросов формы](#creating-form-requests)
    - [Авторизация запросов](#authorizing-form-requests)
    - [Корректировка сообщений об ошибках](#customizing-the-error-messages)
    - [Подготовка входящих данных для валидации](#preparing-input-for-validation)
- [Создание валидатора по требованию](#manually-creating-validators)
    - [Автоматическое перенаправление](#automatic-redirection)
    - [Именованные коллекции ошибок](#named-error-bags)
    - [Корректировка сообщений об ошибках](#manual-customizing-the-error-messages)
    - [Хук валидатора After](#after-validation-hook)
- [Работа с провалидированными входящими данными](#working-with-validated-input)
- [Работа с сообщениями об ошибках](#working-with-error-messages)
    - [Указание пользовательских сообщений в языковых файлах](#specifying-custom-messages-in-language-files)
    - [Указание атрибутов в языковых файлах](#specifying-attribute-in-language-files)
    - [Указание пользовательских имен для атрибутов в языковых файлах](#specifying-values-in-language-files)
- [Доступные правила валидации](#available-validation-rules)
- [Условное добавление правил](#conditionally-adding-rules)
- [Валидация массивов](#validating-arrays)
    - [Валидация вложенных массивов](#validating-nested-array-input)
    - [Индексы и позиции сообщений об ошибках](#error-message-indexes-and-positions)
- [Валидация паролей](#validating-passwords)
- [Пользовательские правила валидации](#custom-validation-rules)
    - [Использование класса Rule](#using-rule-objects)
    - [Использование замыканий](#using-closures)
    - [Неявные правила](#implicit-rules)

<a name="introduction"></a>
## Введение

Laravel предлагает несколько подходов для проверки входящих данных вашего приложения. Например, метод `validate`, доступен для всех входящих HTTP-запросов. Однако мы обсудим и другие подходы к валидации.

Laravel содержит удобные правила валидации, применяемые к данным, включая валидацию на уникальность значения в конкретной таблице базы данных. Мы подробно рассмотрим каждое из этих правил валидации, чтобы вы были знакомы со всеми особенностями валидации Laravel.

<a name="validation-quickstart"></a>
## Быстрый старт

Чтобы узнать о мощных функциях валидации Laravel, давайте рассмотрим полный пример валидации формы и отображения сообщений об ошибках конечному пользователю. Прочитав этот общий обзор, вы сможете получить представление о том, как проверять данные входящего запроса с помощью Laravel:

<a name="quick-defining-the-routes"></a>
### Определение маршрутов

Во-первых, предположим, что в нашем файле `routes/web.php` определены следующие маршруты:

    use App\Http\Controllers\PostController;

    Route::get('/post/create', [PostController::class, 'create']);
    Route::post('/post', [PostController::class, 'store']);

Маршрут `GET` отобразит форму для пользователя для создания нового сообщения в блоге, а маршрут `POST` сохранит новое сообщение в базе данных.

<a name="quick-creating-the-controller"></a>
### Создание контроллера

Затем, давайте взглянем на простой контроллер, который обрабатывает входящие запросы на эти маршруты. Пока оставим метод `store` пустым:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Показать форму для создания нового сообщения в блоге.
         *
         * @return \Illuminate\View\View
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Сохранить новую запись в блоге.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            // Выполнить валидацию и сохранить сообщение в блоге ...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### Написание логики валидации

Теперь мы готовы заполнить наш метод `store` логикой для валидации нового сообщения в блоге. Для этого мы будем использовать метод `validate`, предоставляемый объектом `Illuminate\Http\Request`. Если правила валидации будут пройдены, то ваш код продолжит нормально выполняться; однако, если проверка не пройдена, то будет выброшено исключение `Illuminate\Validation\ValidationException`, и соответствующий ответ об ошибке будет автоматически отправлен обратно пользователю.

Если валидации не пройдена во время традиционного HTTP-запроса, то будет сгенерирован ответ-перенаправление на предыдущий URL-адрес. Если входящий запрос является XHR-запросом, то будет возвращен [JSON-ответ, содержащий сообщения об ошибках валидации](#validation-error-response-format).

Чтобы лучше понять метод `validate`, давайте вернемся к методу `store`:

    /**
     * Сохранить новую запись в блоге.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // Запись блога корректна ...
    }

Как видите, правила валидации передаются в метод `validate`. Не волнуйтесь – все доступные правила валидации [задокументированы](#available-validation-rules). Опять же, если проверка не пройдена, то будет автоматически сгенерирован корректный ответ. Если проверка пройдет успешно, то наш контроллер продолжит нормальную работу.

В качестве альтернативы правила валидации могут быть указаны как массивы правил вместо одной строки с разделителями `|`:

    $validatedData = $request->validate([
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

Кроме того, вы можете использовать метод `validateWithBag` для валидации запроса и сохранения любых сообщений об ошибках в [именованную коллекцию ошибок](#named-error-bags):

    $validatedData = $request->validateWithBag('post', [
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

<a name="stopping-on-first-validation-failure"></a>
#### Прекращение валидации при возникновении первой ошибки

По желанию можно прекратить выполнение правил валидации для атрибута после первой ошибки. Для этого присвойте атрибуту правило `bail`:

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

В этом примере, если правило `unique` для атрибута `title` не будет пройдено, то правило `max` не будет выполняться. Правила будут проверяться в порядке их назначения.

<a name="a-note-on-nested-attributes"></a>
#### Примечание о вложенных атрибутах

Если входящий HTTP-запрос содержит данные «вложенных» полей, то вы можете указать эти поля в своих правилах валидации, используя «точечную нотацию»:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

С другой стороны, если имя вашего поля буквально содержит точку, то вы можете явно запретить ее интерпретацию как часть «точечной нотации», экранировав точку с помощью обратной косой черты:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'v1\.0' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### Отображение ошибок валидации

Итак, что, если поля входящего запроса не проходят указанные правила валидации? Как упоминалось ранее, Laravel автоматически перенаправит пользователя обратно в его предыдущее местоположение. Кроме того, все ошибки валидации и [входящие данные запроса](requests.md#retrieving-old-input) будут автоматически [записаны в сессию](session.md#flash-data).

Переменная `$errors` используется во всех шаблонах вашего приложения благодаря посреднику `Illuminate\View\Middleware\ShareErrorsFromSession`, который включен в группу посредников `web`. Пока применяется этот посредник, в ваших шаблонах всегда будет доступна переменная `$errors`, что позволяет вам предполагать, что переменная `$errors` всегда определена и может безопасно использоваться. Переменная `$errors` будет экземпляром `Illuminate\Support\MessageBag`. Для получения дополнительной информации о работе с этим объектом [ознакомьтесь с его документацией](#working-with-error-messages).

Итак, в нашем примере пользователь будет перенаправлен на метод нашего контроллера `create`, в случае, если валидация завершится неудачно, что позволит нам отобразить сообщения об ошибках в шаблоне:

```blade
<!-- /resources/views/post/create.blade.php -->

<h1>Создание поста блога</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Форма для создания поста блога -->
```

<a name="quick-customizing-the-error-messages"></a>
#### Корректировка сообщений об ошибках

Каждое встроенное правило валидации Laravel содержит сообщение об ошибке, которое находится в файле `lang/en/validation.php` вашего приложения. В этом файле вы найдете запись о переводе для каждого правила валидации. Вы можете изменять или модифицировать эти сообщения в зависимости от потребностей вашего приложения.

Кроме того, вы можете скопировать этот файл в каталог перевода другого языка, чтобы перевести сообщения на язык вашего приложения. Чтобы узнать больше о локализации Laravel, ознакомьтесь с полной [документацией по локализации](localization.md).

<a name="quick-xhr-requests-and-validation"></a>
#### XHR-запросы и валидация

В этом примере мы использовали традиционную форму для отправки данных в приложение. Однако, многие приложения получают запросы XHR с фронтенда с использованием JavaScript. При использовании метода `validate`, во время выполнения XHR-запроса, Laravel не будет генерировать ответ-перенаправление. Вместо этого Laravel генерирует [JSON-ответ, содержащий все ошибки валидации](#validation-error-response-format). Этот ответ JSON будет отправлен с кодом 422 состояния HTTP.

<a name="the-at-error-directive"></a>
#### Директива `@error`

Вы можете использовать директиву [`@error` Blade](blade.md#validation-errors), чтобы быстро определить, существуют ли сообщения об ошибках валидации для конкретного атрибута, включая сообщения об ошибках в именованной коллекции ошибок. В директиве `@error` вы можете вывести содержимое переменной `$message` для отображения сообщения об ошибке:

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Если вы используете [именованные коллекции ошибок](#named-error-bags), то вы можете передать имя коллекции в качестве второго аргумента директивы `@error`:

```blade
<input ... class="@error('title', 'post') is-invalid @enderror">
```

<a name="repopulating-forms"></a>
### Повторное заполнение форм

Когда Laravel генерирует ответ-перенаправление из-за ошибки валидации, фреймворк автоматически [краткосрочно записывает все входные данные запроса в сессию](session.md#flash-data). Это сделано для того, чтобы вы могли удобно получить доступ к входным данным во время следующего запроса и повторно заполнить форму, которую пользователь попытался отправить.

Чтобы получить входные данные предыдущего запроса, вызовите метод `old` экземпляра `Illuminate\Http\Request`. Метод `old` извлечет ранее записанные входные данные из [сессии](session.md):

    $title = $request->old('title');

Laravel также содержит глобального помощника `old`. Если вы показываете входные данные прошлого запроса в [шаблоне Blade](blade.md), то удобнее использовать помощник `old` для повторного заполнения формы. Если для какого-то поля не были предоставлены данные в прошлом запросе, то будет возвращен `null`:

```blade
<input type="text" name="title" value="{{ old('title') }}">
```

<a name="a-note-on-optional-fields"></a>
### Примечание о необязательных полях

По умолчанию Laravel содержит посредников `App\Http\Middleware\TrimStrings` и `App\Http\Middleware\ConvertEmptyStringsToNull` в глобальном стеке посредников вашего приложения. Эти посредники перечислены в классе `App\Http\Kernel`. Первый из упомянутых посредников будет автоматически обрезать все входящие строковые поля запроса, а второй – конвертировать любые пустые строковые поля в `null`. Из-за этого вам часто нужно будет помечать ваши «необязательные» поля запроса как `nullable`, если вы не хотите, чтобы валидатор не считал такие поля недействительными. Например:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

В этом примере мы указываем, что поле `publish_at` может быть либо `null`, либо допустимым представлением даты. Если модификатор `nullable` не добавлен в определение правила, валидатор сочтет `null` недопустимой датой.

<a name="validation-error-response-format"></a>
### Формат ответа ошибок валидации

Когда ваше приложение выбрасывает исключение `Illuminate\Validation\ValidationException` и на входящий HTTP-запрос ожидается ответ JSON, то Laravel автоматически отформатирует для вас сообщения об ошибках и вернет HTTP-ответ `422 Unprocessable Entity`.

Ниже вы можете просмотреть пример формата ответа JSON для ошибок валидации. Обратите внимание, что вложенные ключи ошибок сведены к «точечной» нотации:

```json
{
    "message": "The team name must be a string. (and 4 more errors)",
    "errors": {
        "team_name": [
            "The team name must be a string.",
            "The team name must be at least 1 characters."
        ],
        "authorization.role": [
            "The selected authorization.role is invalid."
        ],
        "users.0.email": [
            "The users.0.email field is required."
        ],
        "users.2.email": [
            "The users.2.email must be a valid email address."
        ]
    }
}
```

<a name="form-request-validation"></a>
## Валидация запроса формы

<a name="creating-form-requests"></a>
### Создание запросов формы

Для более сложных сценариев валидации вы можете создать «запрос формы». Запрос формы – это ваш класс запроса, который инкапсулирует свою собственную логику валидации и авторизации. Чтобы сгенерировать новый запрос формы, используйте команду `make:request` [Artisan](artisan.md):

```shell
php artisan make:request StorePostRequest
```

Эта команда поместит новый класс запроса формы в каталог `app/Http/Requests` вашего приложения. Если этот каталог не существует в вашем приложении, то Laravel предварительно создаст его, когда вы запустите команду `make:request`. Каждый запрос формы, созданный Laravel, имеет два метода: `authorize` и `rules`.

Как вы могли догадаться, метод `authorize` отвечает за определение того, может ли текущий аутентифицированный пользователь выполнить действие, представленное запросом, в то время как метод `rules` возвращает правила валидации, которые должны применяться к данным запроса:

    /**
     * Получить массив правил валидации, которые будут применены к запросу.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

> {tip} Вы можете объявить любые зависимости, которые вам нужны, в сигнатуре метода `rules`. Они будут автоматически извлечены через [контейнер служб](container.md) Laravel.

Итак, как анализируются правила валидации? Все, что вам нужно сделать, это объявить зависимость от запроса в методе вашего контроллера. Входящий запрос формы проверяется до вызова метода контроллера, что означает, что вам не нужно загромождать контроллер какой-либо логикой валидации:

    /**
     * Сохранить новую запись в блоге.
     *
     * @param  \App\Http\Requests\StorePostRequest  $request
     * @return Illuminate\Http\Response
     */
    public function store(StorePostRequest $request)
    {
        // Входящий запрос прошел валидацию ...

        // Получить провалидированные входные данные ...
        $validated = $request->validated();

        // Получить часть провалидированных входных данных ...
        $validated = $request->safe()->only(['name', 'email']);
        $validated = $request->safe()->except(['name', 'email']);
    }

При неуспешной валидации будет сгенерирован ответ-перенаправление, чтобы отправить пользователя обратно в его предыдущее местоположение. Ошибки также будут краткосрочно записаны в сессию, чтобы они были доступны для отображения. Если запрос был XHR-запросом, то пользователю будет возвращен HTTP-ответ с кодом состояния 422, включая [JSON-представление ошибок валидации](#validation-error-response-format).

<a name="adding-after-hooks-to-form-requests"></a>
#### Добавление хуков `after` для запросов форм

Если вы хотите добавить хук валидации `after` для запроса формы, то вы можете использовать метод `withValidator`. Этот метод получает полностью инициированный валидатор, что позволяет вам вызвать любой из его методов до того, как правила валидации будут фактически проанализированы:

    /**
     * Надстройка экземпляра валидатора.
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Что-то не так с этим полем!');
            }
        });
    }


<a name="request-stopping-on-first-validation-rule-failure"></a>
#### Прекращение валидации после первой неуспешной проверки

Добавив свойство `$stopOnFirstFailure` вашему классу запроса, вы можете сообщить валидатору, что он должен прекратить валидацию всех атрибутов после возникновения первой ошибки валидации:

    /**
     * Остановить валидацию после первой неуспешной проверки.
     *
     * @var bool
     */
    protected $stopOnFirstFailure = true;

<a name="customizing-the-redirect-location"></a>
#### Изменение адреса ответа-перенаправления

Как обсуждалось ранее, будет сгенерирован ответ-перенаправление, чтобы отправить пользователя обратно в его предыдущее местоположение, если валидация запроса формы является неуспешной. Однако вы можете настроить это поведение. Для этого определите свойство `$redirect` в вашем запросе формы:

    /**
     * URI перенаправления пользователей в случае неуспешной валидации.
     *
     * @var string
     */
    protected $redirect = '/dashboard';

Или, если вы хотите перенаправить пользователей на именованный маршрут, то вы можете вместо этого определить свойство `$redirectRoute`:

    /**
     * Маршрут перенаправления пользователей в случае неуспешной валидации.
     *
     * @var string
     */
    protected $redirectRoute = 'dashboard';

<a name="authorizing-form-requests"></a>
### Авторизация запросов

Класс запроса формы также содержит метод `authorize`. В рамках этого метода вы можете определить, действительно ли аутентифицированный пользователь имеет право изменять текущий ресурс. Например, вы можете определить, действительно ли пользователь владеет комментарием в блоге, который он пытается обновить. Скорее всего, вы будете взаимодействовать с вашими [шлюзами и политиками авторизации](authorization.md) в этом методе:

    use App\Models\Comment;

    /**
     * Определить, уполномочен ли пользователь выполнить этот запрос.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

Поскольку все запросы формы расширяют базовый класс запросов Laravel, мы можем использовать метод `user` для доступа к текущему аутентифицированному пользователю. Также обратите внимание на вызов метода `route` в приведенном выше примере. Этот метод обеспечивает вам доступ к параметрам URI, определенным для вызываемого маршрута, таким как параметр `{comment}` в приведенном ниже примере:

    Route::post('/comment/{comment}');

Если ваше приложение использует [привязку модели к маршруту](routing.md#route-model-binding), то ваш код можно сделать еще более лаконичным, если обратиться к извлеченной модели как к свойству текущего запроса:

    return $this->user()->can('update', $this->comment);

Если метод `authorize` возвращает `false`, то будет автоматически возвращен HTTP-ответ с кодом состояния 403, и метод вашего контроллера не будет выполнен.

Если вы планируете обрабатывать логику авторизации для запроса в другой части вашего приложения, то вы можете просто вернуть `true` из метода `authorize`:

    /**
     * Определить, уполномочен ли пользователь выполнить этот запрос.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

> {tip} Вы можете объявить любые зависимости, которые вам нужны, в сигнатуре метода `authorize`. Они будут автоматически извлечены через [контейнер служб](container.md) Laravel.

<a name="customizing-the-error-messages"></a>
### Корректировка сообщений об ошибках

Вы можете изменить сообщения об ошибках, используемые в запросе формы, переопределив метод `messages`. Этот метод должен возвращать массив пар атрибут / правило и соответствующие им сообщения об ошибках:

    /**
     * Получить сообщения об ошибках для определенных правил валидации.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required' => 'A message is required',
        ];
    }

<a name="customizing-the-validation-attributes"></a>
#### Корректировка атрибутов валидации

Многие сообщения об ошибках встроенных правил валидации Laravel содержат заполнитель `:attribute`. Если вы хотите, чтобы заполнитель `:attribute` вашего сообщения валидации был заменен другим именем атрибута, то вы можете указать собственные имена, переопределив метод `attributes`. Этот метод должен возвращать массив пар атрибут / имя:

    /**
     * Получить пользовательские имена атрибутов для формирования ошибок валидатора.
     *
     * @return array
     */
    public function attributes()
    {
        return [
            'email' => 'email address',
        ];
    }

<a name="preparing-input-for-validation"></a>
### Подготовка входящих данных для валидации

Если вам необходимо подготовить или обработать какие-либо данные из запроса перед применением правил валидации, то вы можете использовать метод `prepareForValidation`:

    use Illuminate\Support\Str;

    /**
     * Подготовить данные для валидации.
     *
     * @return void
     */
    protected function prepareForValidation()
    {
        $this->merge([
            'slug' => Str::slug($this->slug),
        ]);
    }

<a name="manually-creating-validators"></a>
## Создание валидатора по требованию

Если вы не хотите использовать метод `validate` запроса, то вы можете создать экземпляр валидатора вручную, используя [фасад](facades.md) `Validator`. Метод `make` фасада генерирует новый экземпляр валидатора:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Validator;

    class PostController extends Controller
    {
        /**
         * Сохранить новую запись в блоге.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Получить провалидированные входные данные ...
            $validated = $validator->validated();

            // Получить часть провалидированных входных данных ...
            $validated = $validator->safe()->only(['name', 'email']);
            $validated = $validator->safe()->except(['name', 'email']);

            // Сохранить сообщение блога ...
        }
    }

Первым аргументом, переданным методу `make`, являются проверяемые данные. Второй аргумент – это массив правил валидации, которые должны применяться к данным.

После определения того, что запрос не прошел валидацию с помощью метода `fails`, вы можете использовать метод `withErrors` для передачи сообщений об ошибках в сессию. При использовании этого метода переменная `$errors` будет автоматически передана вашим шаблонам после перенаправления, что позволит вам легко отобразить их обратно пользователю. Метод `withErrors` принимает экземпляр валидатора, экземпляр `MessageBag` или обычный массив PHP.

#### Прекращение валидации после первой неуспешной проверки

Метод `stopOnFirstFailure` проинформирует валидатор о том, что он должен прекратить валидацию всех атрибутов после возникновения первой ошибки валидации:

    if ($validator->stopOnFirstFailure()->fails()) {
        // ...
    }

<a name="automatic-redirection"></a>
### Автоматическое перенаправление

Если вы хотите создать экземпляр валидатора вручную, но по-прежнему воспользоваться преимуществами автоматического перенаправления, предлагаемого методом `validate` HTTP-запроса, вы можете вызвать метод `validate` созданного экземпляра валидатора. Пользователь будет автоматически перенаправлен или, в случае запроса XHR, [будет возвращен ответ JSON](#validation-error-response-format), если валидация будет не успешной:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

Вы можете использовать метод `validateWithBag` для сохранения сообщений об ошибках в [именованной коллекции ошибок](#named-error-bags), если валидация будет не успешной:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validateWithBag('post');

<a name="named-error-bags"></a>
### Именованные коллекции ошибок

Если у вас есть несколько форм на одной странице, то вы можете задать имя экземпляру `MessageBag`, содержащий ошибки валидации, что позволит вам получать сообщения об ошибках для конкретной формы. Чтобы добиться этого, передайте имя в качестве второго аргумента в метод `withErrors`:

    return redirect('register')->withErrors($validator, 'login');

Затем, вы можете получить доступ к именованному экземпляру `MessageBag` из переменной `$errors`:

```blade
{{ $errors->login->first('email') }}
```

<a name="manual-customizing-the-error-messages"></a>
### Корректировка сообщений об ошибках

При необходимости вы можете предоставить собственные сообщения об ошибках, которые должен использовать экземпляр валидатора вместо сообщений об ошибках по умолчанию, предоставляемых Laravel. Есть несколько способов указать собственные сообщения. Во-первых, вы можете передать собственные сообщения в качестве третьего аргумента методу `Validator::make`:

    $validator = Validator::make($input, $rules, $messages = [
        'required' => 'The :attribute field is required.',
    ]);

В этом примере заполнитель `:attribute` будет заменен фактическим именем проверяемого поля. Вы также можете использовать другие заполнители в сообщениях валидатора. Например:

    $messages = [
        'same' => 'The :attribute and :other must match.',
        'size' => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in' => 'The :attribute must be one of the following types: :values',
    ];

<a name="specifying-a-custom-message-for-a-given-attribute"></a>
#### Указание пользовательского сообщения для конкретного атрибута

По желанию можно указать собственное сообщение об ошибке только для определенного атрибута. Вы можете сделать это, используя «точечную нотацию». Сначала укажите имя атрибута, а затем правило:

    $messages = [
        'email.required' => 'We need to know your email address!',
    ];

<a name="specifying-custom-attribute-values"></a>
#### Указание пользовательских имен для атрибутов

Многие сообщения об ошибках встроенных правил валидации Laravel содержат заполнитель `:attribute`, который заменяется именем проверяемого поля или атрибута. Чтобы указать собственные значения, используемые для замены этих заполнителей для конкретных полей, вы можете передать массив ваших атрибутов в качестве четвертого аргумента методу `Validator::make`:

    $validator = Validator::make($input, $rules, $messages, [
        'email' => 'email address',
    ]);

<a name="after-validation-hook"></a>
### Хук валидатора After

Вы также можете определить замыкания, которые будут запускаться после завершения валидации. Это позволяет легко выполнять дальнейшую валидацию и даже добавлять сообщения об ошибках в коллекцию сообщений. Вызовите метод `after` на экземпляре валидатора:

    $validator = Validator::make(/* ... */);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add(
                'field', 'Something is wrong with this field!'
            );
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-validated-input"></a>
## Работа с провалидированными входящими данными

После валидации данных входящего запроса с помощью запроса формы или вручную созданного экземпляра валидатора, вы можете получить данные входящего запроса, которые действительно прошли валидацию. Это можно сделать несколькими способами. Во-первых, вы можете вызвать метод `validated` запроса формы или экземпляра валидатора. Этот метод возвращает массив данных, которые были провалидированы:

    $validated = $request->validated();

    $validated = $validator->validated();

В качестве альтернативы вы можете вызвать метод `safe` запроса формы или экземпляра валидатора. Этот метод возвращает экземпляр `Illuminate\Support\ValidatedInput`. Этот объект предоставляет методы `only`, `except` и `all` для получения как подмножества, так и целого массива провалидированных данных:

    $validated = $request->safe()->only(['name', 'email']);

    $validated = $request->safe()->except(['name', 'email']);

    $validated = $request->safe()->all();

Кроме того, экземпляр `Illuminate\Support\ValidatedInput` может быть итерирован и доступен как массив:

    // Провалидированные данные могут быть итерированы ...
    foreach ($request->safe() as $key => $value) {
        //
    }

    // К провалидированным данным можно получить доступ как к массиву ...
    $validated = $request->safe();

    $email = $validated['email'];

Если вы хотите добавить дополнительные поля к провалидированные данным, то вы можете вызвать метод `merge`:

    $validated = $request->safe()->merge(['name' => 'Taylor Otwell']);

Если вы хотите получить провалидированные данные как экземпляр [коллекции](collections.md), то вы можете вызвать метод `collect`:

    $collection = $request->safe()->collect();

<a name="working-with-error-messages"></a>
## Работа с сообщениями об ошибках

После вызова метода `errors` экземпляр `Validator`, вы получите экземпляр `Illuminate\Support\MessageBag`, который имеет множество удобных методов для работы с сообщениями об ошибках. Переменная `$errors`, которая автоматически становится доступной для всех шаблонов, также является экземпляром класса `MessageBag`.

<a name="retrieving-the-first-error-message-for-a-field"></a>
#### Получение первого сообщения об ошибке для поля

Чтобы получить первое сообщение об ошибке для указанного поля, используйте метод `first`:

    $errors = $validator->errors();

    echo $errors->first('email');

<a name="retrieving-all-error-messages-for-a-field"></a>
#### Получение всех сообщений об ошибках для поля

Если вам нужно получить массив всех сообщений для указанного поля, используйте метод `get`:

    foreach ($errors->get('email') as $message) {
        //
    }

Если вы проверяете массив полей формы, то вы можете получить все сообщения для каждого из элементов массива, используя символ `*`:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

<a name="retrieving-all-error-messages-for-all-fields"></a>
#### Получение всех сообщений об ошибках для всех полей

Чтобы получить массив всех сообщений для всех полей, используйте метод `all`:

    foreach ($errors->all() as $message) {
        //
    }

<a name="determining-if-messages-exist-for-a-field"></a>
#### Определение наличия сообщений для поля

Метод `has` используется для определения наличия сообщений об ошибках для указанного поля:

    if ($errors->has('email')) {
        //
    }

<a name="specifying-custom-messages-in-language-files"></a>
### Указание пользовательских сообщений в языковых файлах

Каждое встроенное правило валидации Laravel содержит сообщение об ошибке, которое находится в файле `lang/en/validation.php` вашего приложения. В этом файле вы найдете запись о переводе для каждого правила валидации. Вы можете изменять или модифицировать эти сообщения в зависимости от потребностей вашего приложения.

Кроме того, вы можете скопировать этот файл в каталог перевода другого языка, чтобы перевести сообщения на язык вашего приложения. Чтобы узнать больше о локализации Laravel, ознакомьтесь с полной [документацией по локализации](localization.md).

<a name="custom-messages-for-specific-attributes"></a>
#### Указание пользовательского сообщения для конкретного атрибута

Вы можете изменить сообщения об ошибках, используемые для указанных комбинаций атрибутов и правил в языковых файлах валидации вашего приложения. Для этого добавьте собственные сообщения в массив `custom` языкового файла `lang/xx/validation.php` вашего приложения:

    'custom' => [
        'email' => [
            'required' => 'We need to know your email address!',
            'max' => 'Your email address is too long!'
        ],
    ],

<a name="specifying-attribute-in-language-files"></a>
### Указание атрибутов в языковых файлах

Многие сообщения об ошибках встроенных правил валидации Laravel содержат заполнитель `:attribute`, который заменяется именем проверяемого поля или атрибута. Если вы хотите, чтобы часть `:attribute` вашего сообщения валидации была заменена собственным значением, то вы можете указать имя настраиваемого атрибута в массиве `attributes` вашего языкового файла `lang/xx/validation.php`:

    'attributes' => [
        'email' => 'email address',
    ],

<a name="specifying-values-in-language-files"></a>
### Указание пользовательских имен для атрибутов в языковых файлах

Некоторые сообщения об ошибках встроенных правил валидации Laravel содержат заполнитель `:value`, который заменяется текущим значением атрибута запроса. Иногда требуется заменить часть `:value` вашего сообщения валидации на собственное значение. Например, рассмотрим следующее правило, которое указывает, что номер кредитной карты требуется обязательно, если для параметра `payment_type` установлено значение `cc`:

    Validator::make($request->all(), [
        'credit_card_number' => 'required_if:payment_type,cc'
    ]);

Если это правило валидации не будет пройдено, то будет выдано следующее сообщение об ошибке:

```none
The credit card number field is required when payment type is cc.
```

Вместо того, чтобы отображать `cc` в качестве значения типа платежа, вы можете указать более удобное для пользователя представление значения в вашем языковом файле `lang/xx/validation.php`, определив массив `values`:

    'values' => [
        'payment_type' => [
            'cc' => 'credit card'
        ],
    ],

После определения этого значения правило валидации выдаст следующее сообщение об ошибке:

```none
The credit card number field is required when payment type is credit card.
```

<a name="available-validation-rules"></a>
## Доступные правила валидации

Ниже приведен список всех доступных правил валидации и их функций:

<!-- <style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style> -->

<!-- <div class="collection-method-list" markdown="1"> -->

- [Accepted](#rule-accepted)
- [Accepted If](#rule-accepted-if)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [After Or Equal (Date)](#rule-after-or-equal)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Array](#rule-array)
- [Bail](#rule-bail)
- [Before (Date)](#rule-before)
- [Before Or Equal (Date)](#rule-before-or-equal)
- [Between](#rule-between)
- [Boolean](#rule-boolean)
- [Confirmed](#rule-confirmed)
- [Current Password](#rule-current-password)
- [Date](#rule-date)
- [Date Equals](#rule-date-equals)
- [Date Format](#rule-date-format)
- [Declined](#rule-declined)
- [Declined If](#rule-declined-if)
- [Different](#rule-different)
- [Digits](#rule-digits)
- [Digits Between](#rule-digits-between)
- [Dimensions (Image Files)](#rule-dimensions)
- [Distinct](#rule-distinct)
- [Email](#rule-email)
- [Ends With](#rule-ends-with)
- [Enum](#rule-enum)
- [Exclude](#rule-exclude)
- [Exclude If](#rule-exclude-if)
- [Exclude Unless](#rule-exclude-unless)
- [Exclude With](#rule-exclude-with)
- [Exclude Without](#rule-exclude-without)
- [Exists (Database)](#rule-exists)
- [File](#rule-file)
- [Filled](#rule-filled)
- [Greater Than](#rule-gt)
- [Greater Than Or Equal](#rule-gte)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [In Array](#rule-in-array)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [JSON](#rule-json)
- [Less Than](#rule-lt)
- [Less Than Or Equal](#rule-lte)
- [MAC Address](#rule-mac)
- [Max](#rule-max)
- [MIME Types](#rule-mimetypes)
- [MIME Type By File Extension](#rule-mimes)
- [Min](#rule-min)
- [Multiple Of](#multiple-of)
- [Not In](#rule-not-in)
- [Not Regex](#rule-not-regex)
- [Nullable](#rule-nullable)
- [Numeric](#rule-numeric)
- [Password](#rule-password)
- [Present](#rule-present)
- [Prohibited](#rule-prohibited)
- [Prohibited If](#rule-prohibited-if)
- [Prohibited Unless](#rule-prohibited-unless)
- [Prohibits](#rule-prohibits)
- [Regex (regular expression)](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required Unless](#rule-required-unless)
- [Required With](#rule-required-with)
- [Required With All](#rule-required-with-all)
- [Required Without](#rule-required-without)
- [Required Without All](#rule-required-without-all)
- [Required Array Keys](#rule-required-array-keys)
- [Same](#rule-same)
- [Size](#rule-size)
- [Sometimes](#validating-when-present)
- [Starts With](#rule-starts-with)
- [String](#rule-string)
- [Timezone](#rule-timezone)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)
- [UUID](#rule-uuid)

<!-- </div> -->

<a name="rule-accepted"></a>
#### accepted

Проверяемое поле должно иметь значение `"yes"`, `"on"`, `1` или `true`. Применяется для валидации принятия раздела «Условия использования» или аналогичных полей.

<a name="rule-accepted-if"></a>
#### accepted_if:anotherfield,value,...

Проверяемое поле должно иметь значение `"yes"`, `"on"`, `1` или `true`, если другое проверяемое поле равно указанному значению. Применяется для валидации принятия раздела «Условия использования» или аналогичных полей.

<a name="rule-active-url"></a>
#### active_url

Проверяемое поле должно иметь допустимую запись A или AAAA в соответствии с функцией `dns_get_record` PHP. Имя хоста указанного URL извлекается с помощью PHP-функции `parse_url` перед передачей в `dns_get_record`.

<a name="rule-after"></a>
#### after:_date_

Проверяемое поле должно иметь значение после указанной даты. Даты будут переданы в функцию `strtotime` PHP для преобразования в действительный экземпляр `DateTime`:

    'start_date' => 'required|date|after:tomorrow'

Вместо передачи строки даты, которая будет проанализирована с помощью `strtotime`, вы можете указать другое поле для сравнения с датой:

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

Проверяемое поле должно иметь значение после указанной даты или равное ей. Для получения дополнительной информации см. правило [after](#rule-after).

<a name="rule-alpha"></a>
#### alpha

Проверяемое поле должно состоять полностью из букв.

<a name="rule-alpha-dash"></a>
#### alpha_dash

Проверяемое поле может содержать буквенно-цифровые символы, а также дефисы и подчеркивания.

<a name="rule-alpha-num"></a>
#### alpha_num

Проверяемое поле должно состоять полностью из буквенно-цифровых символов.

<a name="rule-array"></a>
#### array

Проверяемое поле должно быть массивом PHP.

Когда для правила `array` предоставляются дополнительные значения, каждый ключ во входном массиве должен присутствовать в списке значений, предоставленных правилу. В следующем примере ключ `admin` во входном массиве является недействительным, так как он не содержится в списке значений, предоставленных правилу `array`:

    use Illuminate\Support\Facades\Validator;

    $input = [
        'user' => [
            'name' => 'Taylor Otwell',
            'username' => 'taylorotwell',
            'admin' => true,
        ],
    ];

    Validator::make($input, [
        'user' => 'array:username,locale',
    ]);

В общем, вы всегда должны указывать ключи массива, которые могут присутствовать в вашем массиве.

<a name="rule-bail"></a>
#### bail

Остановить дальнейшее применение правил валидации атрибута после первой неуспешной проверки.

В отличии от правила `bail`, которое прекращает дальнейшую валидацию только конкретного поля, метод `stopOnFirstFailure` сообщит валидатору, что он должен прекратить дальнейшую валидацию всех атрибутов при возникновении первой ошибке:

    if ($validator->stopOnFirstFailure()->fails()) {
        // ...
    }

<a name="rule-before"></a>
#### before:_date_

Проверяемое поле должно быть значением, предшествующим указанной дате. Даты будут переданы в функцию PHP `strtotime` для преобразования в действительный экземпляр `DateTime`. Кроме того, как и в правиле [`after`](#rule-after), имя другого проверяемого поля может быть указано в качестве значения `date`.

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

Проверяемое поле должно иметь значение, предшествующее указанной дате или равное ей. Даты будут переданы в функцию PHP `strtotime` для преобразования в действительный экземпляр `DateTime`. Кроме того, как и в правиле [`after`](#rule-after), имя другого проверяемого поля может быть указано в качестве значения `date`.

<a name="rule-between"></a>
#### between:_min_,_max_

Проверяемое поле должно иметь размер между указанными _min_ и _max_. Строки, числа, массивы и файлы оцениваются так же, как и в правиле [`size`](#rule-size).

<a name="rule-boolean"></a>
#### boolean

Проверяемое поле должно иметь возможность преобразования в логическое значение. Допустимые значения: `true`, `false`, `1`, `0`, `"1"` и `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

Проверяемое поле должно иметь совпадающее поле `{field}_confirmation`. Например, если проверяемое поле – `password`, то поле `password_confirmation` также должно присутствовать во входящих данных.

<a name="rule-current-password"></a>
#### current_password

Проверяемое поле должно соответствовать паролю аутентифицированного пользователя. Вы можете указать [охранника аутентификации](authentication.md), используя первый параметр правила:

    'password' => 'current_password:api'

<a name="rule-date"></a>
#### date

Проверяемое поле должно быть действительной, не относительной датой в соответствии с функцией `strtotime` PHP.

<a name="rule-date-equals"></a>
#### date_equals:_date_

Проверяемое поле должно быть равно указанной дате. Даты будут переданы в функцию `strtotime` PHP для преобразования в действительный экземпляр `DateTime`.

<a name="rule-date-format"></a>
#### date_format:_format_

Проверяемое поле должно соответствовать переданному _format_. При валидации поля следует использовать **либо** `date`, **либо** `date_format`, а не то и другое вместе. Это правило валидации поддерживает все форматы, поддерживаемые классом [`DateTime`](https://www.php.net/manual/ru/class.datetime.php) PHP.

<a name="rule-declined"></a>
#### declined

Проверяемое поле должно иметь значение `"no"`, `"off"`, `0` или `false`.

<a name="rule-declined-if"></a>
#### declined_if:anotherfield,value,...

Проверяемое поле должно иметь значение `"no"`, `"off"`, `0` или `false`, если другое проверяемое поле равно указанному значению.

<a name="rule-different"></a>
#### different:_field_

Проверяемое поле должно иметь значение, отличное от _field_.

<a name="rule-digits"></a>
#### digits:_value_

Проверяемое целое число должно иметь точную длину _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

Проверяемое целое число должно иметь длину между переданными _min_ и _max_.

<a name="rule-dimensions"></a>
#### dimensions

Проверяемый файл должен быть изображением, отвечающим ограничениям размеров, указанным в параметрах правила:

    'avatar' => 'dimensions:min_width=100,min_height=200'

Доступные ограничения: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

Ограничение _ratio_ должно быть представлено как ширина, разделенная на высоту. Это может быть указано дробью вроде `3/2` или числом с плавающей запятой, например `1.5`:

    'avatar' => 'dimensions:ratio=3/2'

Поскольку это правило требует нескольких аргументов, вы можете использовать метод `Rule::dimensions` для гибкости составления правила:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

При валидации массивов проверяемое поле не должно иметь повторяющихся значений:

    'foo.*.id' => 'distinct'

По умолчанию правило `distinct` использует гибкое сравнение переменных. Чтобы использовать жесткое сравнение, вы можете добавить параметр `strict` в определение правила валидации:

    'foo.*.id' => 'distinct:strict'

Вы можете добавить `ignore_case` к аргументам правила валидации, чтобы правило игнорировало различия в использовании регистра букв:

    'foo.*.id' => 'distinct:ignore_case'

<a name="rule-email"></a>
#### email

Проверяемое поле должно быть отформатировано как адрес электронной почты. Это правило валидации использует пакет [`egulias/email-validator`](https://github.com/egulias/EmailValidator) для проверки адреса электронной почты. По умолчанию применяется валидатор `RFCValidation`, но вы также можете применить другие стили валидации:

    'email' => 'email:rfc,dns'

В приведенном выше примере будут применяться проверки `RFCValidation` и `DNSCheckValidation`. Вот полный список стилей проверки, которые вы можете применить:

<!-- <div class="content-list" markdown="1"> -->

- `rfc`: `RFCValidation`
- `strict`: `NoRFCWarningsValidation`
- `dns`: `DNSCheckValidation`
- `spoof`: `SpoofCheckValidation`
- `filter`: `FilterEmailValidation`

<!-- </div> -->

Валидатор `filter`, который использует функцию `filter_var` PHP, поставляется с Laravel и применялся по умолчанию до Laravel версии 5.8.

> {note} Валидаторы `dns` и `spoof` требуют расширения `intl` PHP.

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

Проверяемое поле должно заканчиваться одним из указанных значений.

<a name="rule-enum"></a>
#### enum

Правило `Enum` – это правило на основе класса, которое определяет, имеет ли проверяемое поле допустимое значение перечисления. Правило `Enum` принимает имя перечисления как единственный аргумент конструктора:

    use App\Enums\ServerStatus;
    use Illuminate\Validation\Rules\Enum;

    $request->validate([
        'status' => [new Enum(ServerStatus::class)],
    ]);

> {note} Перечисляемые типы доступны только в [PHP 8.1+](https://www.php.net/manual/ru/language.enumerations.php).

<a name="rule-exclude"></a>
#### exclude

Проверяемое поле будет исключено из данных запроса, возвращаемых методами `validate` и `validated`.

<a name="rule-exclude-if"></a>
#### exclude_if:_anotherfield_,_value_

Проверяемое поле будет исключено из данных запроса, возвращаемых методами `validate` и `validated`, если поле _anotherfield_ равно _value_.

Если требуется сложная логика условного исключения, то вы можете использовать метод `Rule::excludeIf`. Этот метод принимает логическое значение или замыкание. При задании замыкания оно должно возвращать `true` или `false` для указания, следует ли исключить проверяемое поле:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::excludeIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::excludeIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-exclude-unless"></a>
#### exclude_unless:_anotherfield_,_value_

Проверяемое поле будет исключено из данных запроса, возвращаемых методами `validate` и `validated`, если поле _anotherfield_ не равно _value_. Если _value_ равно `null` (т.е. `exclude_unless:name,null`), то проверяемое поле будет исключено, если поле сравнения не имеет значение `null` или поле сравнения отсутствует в данных запроса.

<a name="rule-exclude-with"></a>
#### exclude_with:_anotherfield_

Проверяемое поле будет исключено из данных запроса, возвращаемых методами `validate` и `validated`, если присутствует поле _anotherfield_.

<a name="rule-exclude-without"></a>
#### exclude_without:_anotherfield_

Проверяемое поле будет исключено из данных запроса, возвращаемых методами `validate` и `validated`, если поле _anotherfield_ отсутствует.

<a name="rule-exists"></a>
#### exists:_table_,_column_

Проверяемое поле должно существовать в указанной таблице базы данных.

<a name="basic-usage-of-exists-rule"></a>
#### Основы использования правила Exists

    'state' => 'exists:states'

Если параметр `column` не указан, будет использоваться имя поля. Таким образом, в этом случае правило будет проверять, что таблица базы данных `states` содержит запись со значением столбца `state`, соответствующим значению атрибута `state` запроса.

<a name="specifying-a-custom-column-name"></a>
#### Указание пользовательского имени столбца

Вы можете явно указать имя столбца базы данных, которое должно использоваться правилом валидации, поместив его после имени таблицы базы данных:

    'state' => 'exists:states,abbreviation'

Иногда требуется указать конкретное соединение с базой данных, которое будет использоваться для запроса `exists`. Вы можете сделать это, добавив имя подключения к имени таблицы:

    'email' => 'exists:connection.staff,email'

Вместо того, чтобы указывать имя таблицы напрямую, вы можете указать модель Eloquent, которая должна использоваться для определения имени таблицы:

    'user_id' => 'exists:App\Models\User,id'

Если вы хотите использовать свой запрос, выполняемый правилом валидации, то вы можете использовать класс `Rule` для гибкости определения правила. В этом примере мы также укажем правила валидации в виде массива вместо использования символа `|` для их разделения:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                return $query->where('account_id', 1);
            }),
        ],
    ]);

Вы можете явно указать имя столбца базы данных, которое должно использоваться правилом `exists`, сгенерированным методом `Rule::exists`, указав имя столбца в качестве второго аргумента метода `exists`:

    'state' => Rule::exists('states', 'abbreviation'),

<a name="rule-file"></a>
#### file

Проверяемое поле должно быть успешно загруженным на сервер файлом.

<a name="rule-filled"></a>
#### filled

Проверяемое поле не должно быть пустым, если оно присутствует.

<a name="rule-gt"></a>
#### gt:_field_

Проверяемое поле должно быть больше указанного _field_. Два поля должны быть одного типа. Строки, числа, массивы и файлы оцениваются с использованием тех же соглашений, что и в правиле [`size`](#rule-size).

<a name="rule-gte"></a>
#### gte:_field_

Проверяемое поле должно быть больше или равно указанному _field_. Два поля должны быть одного типа. Строки, числа, массивы и файлы оцениваются с использованием тех же соглашений, что и в правиле [`size`](#rule-size).

<a name="rule-image"></a>
#### image

Проверяемый файл должен быть изображением (jpg, jpeg, png, bmp, gif, svg или webp).

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

Проверяемое поле должно быть включено в указанный список значений. Поскольку это правило часто требует, чтобы вы «объединяли» массив, то метод `Rule::in` можно использовать для гибкого построения правила:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

Когда правило `in` комбинируется с правилом `array`, тогда каждое значение во входном массиве должно присутствовать в списке значений, предоставленных правилу `in`. В следующем примере код аэропорта `LAS` во входном массиве является недействительным, так как он не содержится в списке аэропортов, предоставленном правилу `in`:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    $input = [
        'airports' => ['NYC', 'LAS'],
    ];

    Validator::make($input, [
        'airports' => [
            'required',
            'array',
        ],
        'airports.*' => Rule::in(['NYC', 'LIT']),
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_.*

Проверяемое поле должно существовать в значениях _anotherfield_.

<a name="rule-integer"></a>
#### integer

Проверяемое поле должно быть целым числом.

> {note} Это правило валидации не проверяет, что значение поля относится к типу переменной `integer`, а только что значение поля относится к типу, принятому правилом `FILTER_VALIDATE_INT` PHP. Если вам нужно проверить значение поля в качестве числа, используйте это правило в сочетании с [правилом валидации `numeric`](#rule-numeric).

<a name="rule-ip"></a>
#### ip

Проверяемое поле должно быть IP-адресом.

<a name="ipv4"></a>
#### ipv4

Проверяемое поле должно быть адресом IPv4.

<a name="ipv6"></a>
#### ipv6

Проверяемое поле должно быть адресом IPv6.

<a name="rule-json"></a>
#### json

Проверяемое поле должно быть допустимой строкой JSON.

<a name="rule-lt"></a>
#### lt:_field_

Проверяемое поле должно быть меньше переданного _field_. Два поля должны быть одного типа. Строки, числа, массивы и файлы оцениваются с использованием тех же соглашений, что и в правиле [`size`](#rule-size).

<a name="rule-lte"></a>
#### lte:_field_

Проверяемое поле должно быть меньше или равно переданному _field_. Два поля должны быть одного типа. Строки, числа, массивы и файлы оцениваются с использованием тех же соглашений, что и в правиле [`size`](#rule-size).

<a name="rule-mac"></a>
#### mac_address

Проверяемое поле должно быть MAC-адресом.

<a name="rule-max"></a>
#### max:_value_

Проверяемое поле должно быть меньше или равно максимальному _value_. Строки, числа, массивы и файлы оцениваются с использованием тех же соглашений, что и в правиле [`size`](#rule-size).

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

Проверяемый файл должен соответствовать одному из указанных MIME-типов:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

Чтобы определить MIME-тип загруженного файла, содержимое файла будет прочитано, и фреймворк попытается угадать MIME-тип, который может отличаться от типа, предоставленного клиентом.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

Проверяемый файл должен иметь MIME-тип, соответствующий одному из перечисленных расширений.

<a name="basic-usage-of-mime-rule"></a>
#### Основы использования правила MIME

    'photo' => 'mimes:jpg,bmp,png'

Несмотря на то, что вам нужно только указать расширения, это правило фактически проверяет MIME-тип файла, читая содержимое файла и угадывая его MIME-тип. Полный список типов MIME и соответствующих им расширений можно найти по следующему адресу:

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

Проверяемое поле должно иметь минимальное значение _value_. Строки, числа, массивы и файлы оцениваются с использованием тех же соглашений, что и в правиле [`size`](#rule-size).

<a name="multiple-of"></a>
#### multiple_of:_value_

Проверяемое поле должно быть кратным _value_.

> {note} Для использования правила `multiple_of` требуется [расширение `bcmath` PHP](https://www.php.net/manual/ru/book.bc.php).

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

Проверяемое поле не должно быть включено в переданный список значений. Метод `Rule::notIn` используется для гибкого построения правила:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

Проверяемое поле не должно соответствовать переданному регулярному выражению.

Внутренне это правило использует функцию `preg_match` PHP. Указанный шаблон должен подчиняться тому же форматированию, требуемому `preg_match`, и, следовательно, также включать допустимые разделители. Например: `'email' => 'not_regex:/^.+$/i'`.

> {note} При использовании шаблонов `regex` / `not_regex` может потребоваться указать ваши правила валидации с использованием массива вместо использования разделителей `|`, особенно если регулярное выражение содержит символ `|`.

<a name="rule-nullable"></a>
#### nullable

Проверяемое поле может быть `null`.

<a name="rule-numeric"></a>
#### numeric

Проверяемое поле должно быть [числовым](https://www.php.net/manual/ru/function.is-numeric.php).

<a name="rule-password"></a>
#### password

Проверяемое поле должно соответствовать паролю аутентифицированного пользователя.

> {note} Это правило было переименовано в `current_password` с его дальнейшим удалением в Laravel 9. Вместо этого используйте правило [Current Password](#rule-current-password).

<a name="rule-present"></a>
#### present

Проверяемое поле должно присутствовать во входных данных, но может быть пустым.

<a name="rule-prohibited"></a>
#### prohibited

Проверяемое поле должно быть пустым или отсутствовать.

<a name="rule-prohibited-if"></a>
#### prohibited_if:_anotherfield_,_value_,...

Проверяемое поле должно быть пустым или отсутствовать, если поле _anotherfield_ равно любому _value_.

Если требуется сложная логика условного запрета присутствия, то вы можете использовать метод `Rule::prohibitedIf`. Этот метод принимает логическое значение или замыкание. При задании замыкания оно должно возвращать `true` или `false` для указания, следует ли запретить присутствие проверяемого поля:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::prohibitedIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::prohibitedIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-prohibited-unless"></a>
#### prohibited_unless:_anotherfield_,_value_,...

Проверяемое поле должно быть пустым или отсутствовать, если поле _anotherfield_ не равно какому-либо _value_.

<a name="rule-prohibits"></a>
#### prohibits:_anotherfield_,...

Если проверяемое поле присутствует, никакие поля в _anotherfield_ не могут присутствовать, даже если они пустые.

<a name="rule-regex"></a>
#### regex:_pattern_

Проверяемое поле должно соответствовать переданному регулярному выражению.

Внутренне это правило использует функцию `preg_match` PHP. Указанный шаблон должен подчиняться тому же форматированию, требуемому `preg_match`, и, следовательно, также включать допустимые разделители. Например: `'email' => 'regex:/^.+@.+$/i'`.

> {note} При использовании шаблонов `regex` / `not_regex` может потребоваться указать ваши правила валидации с использованием массива вместо использования разделителей `|`, особенно если регулярное выражение содержит символ `|`.

<a name="rule-required"></a>
#### required

Проверяемое поле должно присутствовать во входных данных и не быть пустым. Поле считается «пустым», если выполняется одно из следующих условий:

<!-- <div class="content-list" markdown="1"> -->

- Значение поля равно `null`.
- Значение поля – пустая строка.
- Значение поля представляет собой пустой массив или пустой объект, реализующий интерфейс `Countable`.
- Значение поля – загружаемый файл, но без пути.

<!-- </div> -->

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

Проверяемое поле должно присутствовать и не быть пустым, если поле _anotherfield_ равно любому _value_.

Если вы хотите создать более сложное условие для правила `required_if`, вы можете использовать метод `Rule::requiredIf`. Этот метод принимает логическое значение или замыкание. При выполнении замыкания оно должно возвращать `true` или `false`, чтобы указать, обязательно ли проверяемое поле:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

Проверяемое поле должно присутствовать и не быть пустым, если поле _anotherfield_ не равно какому-либо _value_. Это также означает, что в данных запроса должно присутствовать _anotherfield_, если _value_ не имеет значения `null`. Если _value_ равно `null` (т.е. `required_unless:name,null`), то проверяемое поле будет обязательным, если поле сравнения не равно `null` или поле сравнения отсутствует в данных запроса.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

Проверяемое поле должно присутствовать и не быть пустым, _только если_ любое из других указанных полей присутствует и не является пустым.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Проверяемое поле должно присутствовать и не быть пустым, _только если_ все другие указанные поля присутствуют и не являются пустыми.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

Проверяемое поле должно присутствовать и не быть пустым, _только когда_ любое из других указанных полей является пустым или отсутствует.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Проверяемое поле должно присутствовать и не быть пустым, _только когда_ все другие указанные поля являются пустыми или отсутствуют.

<a name="rule-required-array-keys"></a>
#### required_array_keys:_foo_,_bar_,...

Проверяемое поле должно быть массивом и содержать как минимум указанные ключи.

<a name="rule-same"></a>
#### same:_field_

Переданное _field_ должно соответствовать проверяемому полю.

<a name="rule-size"></a>
#### size:_value_

Проверяемое поле должно иметь размер, соответствующий переданному _value_. Для строковых данных _value_ соответствует количеству символов. Для числовых данных _value_ соответствует переданному целочисленному значению (атрибут также должен иметь правило `numeric` или `integer`). Для массива _size_ соответствует `count` массива. Для файлов _size_ соответствует размеру файла в килобайтах. Давайте посмотрим на несколько примеров:

    // Проверяем, что строка содержит ровно 12 символов ...
    'title' => 'size:12';

    // Проверяем, что передано целое число, равно 10 ...
    'seats' => 'integer|size:10';

    // Проверяем, что в массиве ровно 5 элементов ...
    'tags' => 'array|size:5';

    // Проверяем, что размер загружаемого файла составляет ровно 512 килобайт ...
    'image' => 'file|size:512';

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

Проверяемое поле должно начинаться с одного из указанных значений.

<a name="rule-string"></a>
#### string

Проверяемое поле должно быть строкой. Если вы хотите, чтобы поле также могло быть `null`, вы должны назначить этому полю правило `nullable`.

<a name="rule-timezone"></a>
#### timezone

Проверяемое поле должно быть допустимым идентификатором часового пояса в соответствии с функцией `timezone_identifiers_list` PHP.

<a name="rule-unique"></a>
#### unique:_table_,_column_

Проверяемое поле не должно существовать в указанной таблице базы данных.

**Указание пользовательского имени таблицы / имени столбца:**

Вместо того, чтобы указывать имя таблицы напрямую, вы можете указать модель Eloquent, которая должна использоваться для определения имени таблицы:

    'email' => 'unique:App\Models\User,email_address'

Параметр `column` используется для указания соответствующего столбца базы данных поля. Если опция `column` не указана, будет использоваться имя проверяемого поля.

    'email' => 'unique:users,email_address'

**Указание пользовательского соединения базы данных**

Иногда требуется указать конкретное соединение для запросов к базе данных, выполняемых валидатором. Для этого вы можете добавить имя подключения к имени таблицы:

    'email' => 'unique:connection.users,email_address'

**Принудительное игнорирование правилом Unique конкретного идентификатора:**

Иногда вы можете проигнорировать конкретный идентификатор во время валидации `unique`. Например, рассмотрим страницу «Обновления профиля», которая включает имя пользователя, адрес электронной почты и местоположение. Вероятно, вы захотите убедиться, что адрес электронной почты уникален. Однако, если пользователь изменяет только поле имени, а не поле электронной почты, то вы не захотите, чтобы выдавалась ошибка валидация, поскольку пользователь уже является владельцем рассматриваемого адреса электронной почты.

Чтобы указать валидатору игнорировать идентификатор пользователя, мы воспользуемся классом `Rule` для гибкого определения правила. В этом примере мы также укажем правила валидации в виде массива вместо использования символа `|` для их разделения:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

> {note} Вы никогда не должны передавать какое-либо введенное пользователем значение из запроса в метод `ignore`. Вместо этого вы должны передавать только сгенерированный системой уникальный идентификатор, такой как автоинкрементный идентификатор или UUID экземпляра модели Eloquent. В противном случае ваше приложение будет уязвимо для атаки с использованием SQL-инъекции.

Вместо того, чтобы передавать значение ключа модели методу `ignore`, вы также можете передать весь экземпляр модели. Laravel автоматически извлечет ключ из модели:

    Rule::unique('users')->ignore($user)

Если ваша таблица использует имя столбца с первичным ключом, отличное от `id`, то вы можете указать имя столбца при вызове метода `ignore`:

    Rule::unique('users')->ignore($user->id, 'user_id')

По умолчанию правило `unique` проверяет уникальность столбца, совпадающего с именем проверяемого атрибута. Однако вы можете передать другое имя столбца в качестве второго аргумента метода `unique`:

    Rule::unique('users', 'email_address')->ignore($user->id),

**Добавление дополнительных выражений Where:**

Вы можете указать дополнительные условия запроса, изменив запрос с помощью метода `where`. Например, давайте добавим условие запроса, которое ограничивает область запроса только поиском записями, у которых значение столбца `account_id` равно `1`:

    'email' => Rule::unique('users')->where(fn ($query) => $query->where('account_id', 1))

<a name="rule-url"></a>
#### url

Проверяемое поле должно быть действительным URL.

<a name="rule-uuid"></a>
#### uuid

Проверяемое поле должно быть действительным универсальным уникальным идентификатором (UUID) RFC 4122 (версии 1, 3, 4 или 5).

<a name="conditionally-adding-rules"></a>
## Условное добавление правил

<a name="skipping-validation-when-fields-have-certain-values"></a>
#### Пропуск валидации при определенных значениях полей

По желанию можно не проверять конкретное поле, если другое поле имеет указанное значение. Вы можете сделать это, используя правило валидации `exclude_if`. В этом примере поля `appointment_date` и `doctor_name` не будут проверяться, если поле `has_appointment` имеет значение `false`:

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($data, [
        'has_appointment' => 'required|boolean',
        'appointment_date' => 'exclude_if:has_appointment,false|required|date',
        'doctor_name' => 'exclude_if:has_appointment,false|required|string',
    ]);

В качестве альтернативы вы можете использовать правило `exclude_unless`, чтобы не проверять конкретное поле, если другое поле не имеет указанного значения:

    $validator = Validator::make($data, [
        'has_appointment' => 'required|boolean',
        'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
        'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
    ]);

<a name="validating-when-present"></a>
#### Валидация при условии наличия

По желанию можно выполнить валидацию поля, **только** если это поле присутствует в проверяемых данных. Чтобы этого добиться, добавьте правило `sometimes` в свой список правил:

    $validator = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

В приведенном выше примере поле `email` будет проверено, только если оно присутствует в массиве `$request->all()`.

> {tip} Если вы пытаетесь проверить поле, которое всегда должно присутствовать, но может быть пустым, ознакомьтесь с [этим примечанием о необязательных полях](#a-note-on-optional-fields).

<a name="complex-conditional-validation"></a>
#### Комплексная условная проверка

Иногда вы можете добавить правила валидации, основанные на более сложной условной логике. Например, вы можете потребовать обязательного присутствия переданного поля только в том случае, если другое поле имеет значение больше 100. Или вам может потребоваться, чтобы два поля имели указанное значение только при наличии другого поля. Добавление этих правил валидации не должно вызывать затруднений. Сначала создайте экземпляр `Validator` со своими _статическими правилами_, которые будут неизменными:

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($request->all(), [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Предположим, что наше веб-приложение предназначено для коллекционеров игр. Если коллекционер игр регистрируется в нашем приложении и у него есть более 100 игр, мы хотим, чтобы он объяснил, почему у него так много игр. Например, возможно, они владеют магазином по перепродаже игр или, может быть, им просто нравится коллекционировать игры. Чтобы условно добавить это требование, мы можем использовать метод `sometimes` экземпляра `Validator`:

    $validator->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

Первый аргумент, переданный методу `sometimes` – это имя поля, которое мы условно проверяем. Второй аргумент – это список правил, которые мы хотим добавить. Если замыкание, переданное в качестве третьего аргумента, возвращает `true`, то правила будут добавлены. Этот метод упрощает создание сложных условных проверок. Вы даже можете добавить условные проверки сразу для нескольких полей:

    $validator->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} Параметр `$input`, переданный вашему замыканию, будет экземпляром `Illuminate\Support\Fluent` и может использоваться при валидации для доступа к вашим входящим данным и файлам запроса.

<a name="complex-conditional-array-validation"></a>
#### Комплексная условная валидация массива

Иногда требуется проверить поле на основе другого поля в том же вложенном массиве, индекс которого вам неизвестен. В этих ситуациях вы можете использовать второй аргумент вашего замыкания, который будет текущим отдельным элементом в проверяемом массиве:

    $input = [
        'channels' => [
            [
                'type' => 'email',
                'address' => 'abigail@example.com',
            ],
            [
                'type' => 'url',
                'address' => 'https://example.com',
            ],
        ],
    ];

    $validator->sometimes('channels.*.address', 'email', function ($input, $item) {
        return $item->type === 'email';
    });

    $validator->sometimes('channels.*.address', 'url', function ($input, $item) {
        return $item->type !== 'email';
    });

Подобно параметру `$input`, переданному в замыкание, параметр `$item` будет являться экземпляром `Illuminate\Support\Fluent`, если атрибут данных является массивом; в противном случае – это строка.

<a name="validating-arrays"></a>
## Валидация массивов

Как обсуждалось в [правилах валидации массива](#rule-array), правило `array` принимает список разрешенных ключей массива. Если в массиве присутствуют какие-либо дополнительные ключи, проверка не будет пройдена:

    use Illuminate\Support\Facades\Validator;

    $input = [
        'user' => [
            'name' => 'Taylor Otwell',
            'username' => 'taylorotwell',
            'admin' => true,
        ],
    ];

    Validator::make($input, [
        'user' => 'array:username,locale',
    ]);

В общем, вы всегда должны указывать ключи массива, которые могут присутствовать в вашем массиве. В противном случае методы валидатора `validate` и `validated` вернут все провалидированные данные, включая массив и все его ключи, даже если эти ключи не были провалидированы другими правилами валидации вложенных массивов.

<a name="validating-nested-array-input"></a>
### Валидация вложенных массивов

Проверка полей ввода формы на основе массива не должна быть проблемой. Вы можете использовать «точечную нотацию» для валидации атрибутов в массиве. Например, если входящий HTTP-запрос содержит поле `photos[profile]`, вы можете проверить его следующим образом:

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

Вы также можете проверить каждый элемент массива. Например, чтобы убедиться, что каждое электронное письмо в переданном поле ввода массива уникально, вы можете сделать следующее:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Точно так же вы можете использовать символ `*` при указании [пользовательских сообщений в ваших языковых файлах](#custom-messages-for-specific-attributes), что упрощает использование одного сообщения валидации для полей на основе массива:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique email address',
        ]
    ],

<a name="accessing-nested-array-data"></a>
#### Доступ к данным вложенного массива

Иногда требуется получить доступ к значению переданного вложенного элемента массива при назначении правил валидации для атрибута. Вы можете сделать это, используя метод `Rule::forEach`. Метод `forEach` принимает замыкание, которое будет вызываться для каждой итерации проверяемого атрибута массива, и будет получать значение атрибута и явное, полностью развернутое имя атрибута. Замыкание должно возвращать массив правил предназначенных элементу массива:

    use App\Rules\HasPermission;
    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    $validator = Validator::make($request->all(), [
        'companies.*.id' => Rule::forEach(function ($value, $attribute) {
            return [
                Rule::exists(Company::class, 'id'),
                new HasPermission('manage-company', $value),
            ];
        }),
    ]);

<a name="error-message-indexes-and-positions"></a>
### Индексы и позиции сообщений об ошибках

При валидации массивов вы можете указать индекс или положение определенного элемента, который не прошел проверку, в сообщении об ошибке, отображаемом вашим приложением. Для этого вы можете включить заполнители `:index` и `:position` в свое [собственное сообщение об ошибке](#manual-customizing-the-error-messages):

    use Illuminate\Support\Facades\Validator;

    $input = [
        'photos' => [
            [
                'name' => 'BeachVacation.jpg',
                'description' => 'A photo of my beach vacation!',
            ],
            [
                'name' => 'GrandCanyon.jpg',
                'description' => '',
            ],
        ],
    ];

    Validator::validate($input, [
        'photos.*.description' => 'required',
    ], [
        'photos.*.description.required' => 'Please describe photo #:position.',
    ]);

В приведенном выше примере валидация завершится ошибкой, и пользователю будет представлена ​​следующая ошибка: _"Please describe photo #2."_

<a name="validating-passwords"></a>
## Валидация паролей

Чтобы гарантировать, что пароли имеют адекватный уровень сложности, вы можете использовать класс правила `Password` Laravel:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rules\Password;

    $validator = Validator::make($request->all(), [
        'password' => ['required', 'confirmed', Password::min(8)],
    ]);

Класс правила `Password` позволяет вам легко форсировать требования к сложности пароля для вашего приложения, например указать, что для паролей требуется хотя бы одна буква, цифра, символ или символы со смешанным регистром:

    // Требуется не менее 8 символов ...
    Password::min(8)

    // Требуется хотя бы одна буква ...
    Password::min(8)->letters()

    // Требуется хотя бы одна заглавная и одна строчная буквы ...
    Password::min(8)->mixedCase()

    // Требуется хотя бы одна цифра ...
    Password::min(8)->numbers()

    // Требуется хотя бы один символ ...
    Password::min(8)->symbols()

Кроме того, вы можете гарантировать, что пароль не был [скомпрометирован](https://ru.wikipedia.org/wiki/Компрометация_(криптография)) при утечке данных, используя метод `uncompromised`:

    Password::min(8)->uncompromised()

Внутренне объект правила `Password` использует модель [k-Anonymity](https://en.wikipedia.org/wiki/K-anonymity), чтобы определить, не произошла ли утечка пароля через сервис [haveibeenpwned.com](https://haveibeenpwned.com) без ущерба для конфиденциальности или безопасности пользователя.

По умолчанию, если пароль фигурирует хотя бы один раз в утечке данных, он будет считаться скомпрометированным. Вы можете изменить этот критерий, используя первый аргумент метода `uncompromised`:

    // Убедиться, что пароль появляется менее 3 раз в утечке данных ...
    Password::min(8)->uncompromised(3);

Вы можете связать все методы в упомянутых выше примерах:

    Password::min(8)
        ->letters()
        ->mixedCase()
        ->numbers()
        ->symbols()
        ->uncompromised()

<a name="defining-default-password-rules"></a>
#### Определение правил валидации паролей по умолчанию

Возможно, вам будет удобно указать правила валидации паролей по умолчанию в одном месте вашего приложения. Вы можете легко сделать это, используя метод `Password::defaults`, который принимает замыкание. Замыкание, переданное методу `defaults`, должно вернуть конфигурацию правила пароля по умолчанию. Обычно вызов метода `defaults` осуществляется в методе `boot` одного из поставщиков служб вашего приложения:

```php
use Illuminate\Validation\Rules\Password;

/**
 * Загрузка любых служб приложения.
 *
 * @return void
 */
public function boot()
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
                    ? $rule->mixedCase()->uncompromised()
                    : $rule;
    });
}
```

Затем, когда вы хотите применить правила по умолчанию к конкретному паролю, проходящему валидацию, вы можете вызвать метод `defaults` без аргументов:

    'password' => ['required', Password::defaults()],

По желанию можно добавить дополнительные правила валидации к правилам валидации пароля по умолчанию. Для этого вы можете использовать метод `rules`:

    use App\Rules\ZxcvbnRule;

    Password::defaults(function () {
        $rule = Password::min(8)->rules([new ZxcvbnRule]);

        // ...
    });

<a name="custom-validation-rules"></a>
## Пользовательские правила валидации

<a name="using-rule-objects"></a>
### Использование класса Rule

Laravel предлагает множество полезных правил валидации; однако вы можете указать свои собственные. Один из методов регистрации собственных правил валидации – использование объектов правил. Чтобы сгенерировать новый объект правила, вы можете использовать команду `make:rule` [Artisan](artisan.md). Давайте воспользуемся этой командой, чтобы сгенерировать правило, которое проверяет, что строка состоит из прописных букв. Laravel поместит новый класс правила в каталог `app/Rules` вашего приложения. Если этот каталог не существует в вашем приложении, то Laravel предварительно создаст его, когда вы выполните команду Artisan для создания своего правила:

```shell
php artisan make:rule Uppercase
```

Как только правило создано, мы готовы определить его поведение. Объект правила содержит два метода: `passes` и `message`. Метод `passes` получает значение и имя атрибута и должен возвращать `true` или `false` в зависимости от того, является ли значение атрибута допустимым или нет. Метод `message` должен возвращать сообщение об ошибке валидации, которое следует использовать, если проверка оказалась не успешной:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class Uppercase implements Rule
    {
        /**
         * Определить, пройдено ли правило валидации.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }

        /**
         * Получить сообщение об ошибке валидации.
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }

Вы можете вызвать помощник `trans` в методе `message`, если хотите вернуть сообщение об ошибке из ваших файлов перевода:

    /**
     * Получить сообщение об ошибке валидации.
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

После определения правила вы можете отправить его валидатору, передав экземпляр объекта правила с другими вашими правилами валидации:

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', 'string', new Uppercase],
    ]);

#### Доступ к дополнительным данным

Если вашему пользовательскому классу правил валидации требуется доступ к другим валидируемым данным, то ваш класс правил может реализовать интерфейс `Illuminate\Contracts\Validation\DataAwareRule`. Этот интерфейс требует, чтобы ваш класс определял метод `setData`. Этот метод будет автоматически вызван Laravel (до того, как начнется валидация) со всеми проверяемыми данными:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;
    use Illuminate\Contracts\Validation\DataAwareRule;

    class Uppercase implements Rule, DataAwareRule
    {
        /**
         * Все валидируемые данные.
         *
         * @var array
         */
        protected $data = [];

        // ...

        /**
         * Установить валидируемые данные.
         *
         * @param  array  $data
         * @return $this
         */
        public function setData($data)
        {
            $this->data = $data;

            return $this;
        }
    }

Или, если вашему правилу валидации требуется доступ к экземпляру текущего валидатора, то вы можете реализовать интерфейс `ValidatorAwareRule`:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;
    use Illuminate\Contracts\Validation\ValidatorAwareRule;

    class Uppercase implements Rule, ValidatorAwareRule
    {
        /**
         * Экземпляр валидатора.
         *
         * @var \Illuminate\Validation\Validator
         */
        protected $validator;

        // ...

        /**
         * Установить текущий валидатор.
         *
         * @param  \Illuminate\Validation\Validator  $validator
         * @return $this
         */
        public function setValidator($validator)
        {
            $this->validator = $validator;

            return $this;
        }
    }

<a name="using-closures"></a>
### Использование замыканий

Если вам нужна функциональность собственного правила только один раз во всем приложении, то вы можете использовать анонимное правило вместо объекта правила. Анонимное правило получит имя атрибута, значение атрибута и замыкание `$fail`, которое должно быть вызвано при не успешной проверки:

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($request->all(), [
        'title' => [
            'required',
            'max:255',
            function ($attribute, $value, $fail) {
                if ($value === 'foo') {
                    $fail('The '.$attribute.' is invalid.');
                }
            },
        ],
    ]);

<a name="implicit-rules"></a>
### Неявные правила

По умолчанию, правила валидации, включая созданные вами, не применяются, если проверяемый атрибут отсутствует или содержит пустую строку. Например, правило [`unique`](#rule-unique) не будет выполнено для пустой строки:

    use Illuminate\Support\Facades\Validator;

    $rules = ['name' => 'unique:users,name'];

    $input = ['name' => ''];

    Validator::make($input, $rules)->passes(); // true

Чтобы ваше правило было применено, даже если атрибут пуст, то правило должно подразумевать, что атрибут является обязательным. Чтобы создать «неявное» правило, реализуйте интерфейс `Illuminate\Contracts\Validation\ImplicitRule`. Это «маркерный интерфейс» для валидатора; следовательно, он не содержит никаких дополнительных методов, которые вам нужно реализовать, помимо методов, требуемых типичным интерфейсом `Rule`.

Чтобы сгенерировать новый объект неявного правила, вы можете использовать команду `make:rule` Artisan с флагом `--implicit`:

```shell
php artisan make:rule Uppercase --implicit
```

> {note} «Неявное» правило только _подразумевает_, что атрибут является обязательным к валидации. В действительности, решать только вам, будет ли пустой или отсутствующий атрибут считаться невалидным.
