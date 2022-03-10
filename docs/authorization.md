# Laravel 9 · Авторизация

- [Введение](#introduction)
- [Шлюзы](#gates)
    - [Написание шлюзов](#writing-gates)
    - [Авторизация действий через шлюзы](#authorizing-actions-via-gates)
    - [Ответы шлюза](#gate-responses)
    - [Хуки шлюзов](#intercepting-gate-checks)
    - [Упрощенная авторизация](#inline-authorization)
- [Создание политик](#creating-policies)
    - [Генерация политик](#generating-policies)
    - [Регистрация политик](#registering-policies)
- [Написание политик](#writing-policies)
    - [Методы политики](#policy-methods)
    - [Ответы политики](#policy-responses)
    - [Методы политики без моделей](#methods-without-models)
    - [Гостевые пользователи](#guest-users)
    - [Фильтры политики](#policy-filters)
- [Авторизация действий с помощью политик](#authorizing-actions-using-policies)
    - [Через модель User](#via-the-user-model)
    - [Через помощников контроллера](#via-controller-helpers)
    - [Через посредника](#via-middleware)
    - [Через шаблоны Blade](#via-blade-templates)
    - [Предоставление дополнительного контекста политики](#supplying-additional-context)

<a name="introduction"></a>
## Введение

Помимо встроенных служб [аутентификации](authentication.md), Laravel также предлагает простой способ авторизации действий пользователя с конкретными ресурсами. Например, даже если пользователь аутентифицирован, то он может быть не авторизован для обновления или удаления определенных моделей Eloquent или записей базы данных вашего приложения. Функционал авторизации Laravel обеспечивает простой и организованный способ управления этими проверками авторизации.

Laravel предлагает два основных способа авторизации действий: [шлюзы](#gates) и [политики](#creating-policies). <!--Think of gates and policies like routes and controllers.--> Шлюзы обеспечивают простой подход к авторизации, основанный на замыкании, в то время как политики, <!--like controllers,--> группируют логику вокруг конкретной модели или ресурса. В этой документации мы сначала рассмотрим шлюзы, а затем рассмотрим политики.

Вам не нужно выбирать между использованием исключительно шлюзов или исключительно политик при создании приложения. Большинство приложений, скорее всего, будут содержать смесь шлюзов и политик, и это нормально! Шлюзы наиболее применимы к действиям, не связанным с какой-либо моделью или ресурсом, например к просмотру панели администратора. Напротив, политики следует использовать, когда вы хотите разрешить действие для конкретной модели или ресурса.

<a name="gates"></a>
## Шлюзы

<a name="writing-gates"></a>
### Написание шлюзов

> {note} Шлюзы – отличный способ изучить основы функционала авторизации Laravel; однако при создании надежных приложений Laravel, вам следует рассмотреть возможность использования [политик](#creating-policies) для организации ваших правил авторизации.

Шлюз – это просто замыкание, которое определяет, имеет ли пользователь право выполнять указанное действие. Как правило, шлюзы определяются в методе `boot` поставщика `App\Providers\AuthServiceProvider` с использованием фасада `Gate`. Шлюзы всегда получают экземпляр пользователя в качестве своего первого аргумента и могут получать дополнительные аргументы, например, модель Eloquent.

В этом примере мы определим шлюз, решающий, может ли пользователь обновить указанную модель `App\Models\Post`. Шлюз выполнит это, сравнив идентификатор пользователя с идентификатором `user_id` пользователя, создавшего пост:

    use App\Models\Post;
    use App\Models\User;
    use Illuminate\Support\Facades\Gate;

    /**
     * Регистрация любых служб аутентификации / авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id;
        });
    }

Шлюзы также могут быть определены с использованием callback-массива:<!--Like controllers, gates may also be defined using a class callback array:-->

    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;

    /**
     * Регистрация любых служб аутентификации / авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', [PostPolicy::class, 'update']);
    }

<a name="authorizing-actions-via-gates"></a>
### Авторизация действий через шлюзы

Чтобы авторизовать действие с помощью шлюзов, вы должны использовать методы `allows` или `denies` фасада `Gate`. Обратите внимание, что вам не требуется передавать в эти методы аутентифицированного в данный момент пользователя. Laravel автоматически позаботится о передаче пользователя в замыкание шлюза. Обычно методы авторизации шлюза вызываются в контроллерах вашего приложения перед выполнением действия, требующего авторизации:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Gate;

    class PostController extends Controller
    {
        /**
         * Обновить переданный пост.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \App\Models\Post  $post
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request, Post $post)
        {
            if (! Gate::allows('update-post', $post)) {
                abort(403);
            }

            // Обновление поста ...
        }
    }

Если вы хотите определить, авторизован ли другой (не аутентифицированный в настоящий момент) пользователь для выполнения действия, то вы можете использовать метод `forUser` фасада `Gate`:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // Пользователь может обновить пост ...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // Пользователь не может обновить пост ...
    }

Вы можете определить авторизацию нескольких действий одновременно, используя методы `any` или `none`:

    if (Gate::any(['update-post', 'delete-post'], $post)) {
        // Пользователь может обновить или удалить пост ...
    }

    if (Gate::none(['update-post', 'delete-post'], $post)) {
        // Пользователь не может обновить или удалить пост ...
    }

<a name="authorizing-or-throwing-exceptions"></a>
#### Авторизация или выброс исключений

Если вы хотите попытаться авторизовать действие и автоматически выбросить исключение `Illuminate\Auth\Access\AuthorizationException` при не авторизованном действии пользователя, то вы можете использовать метод `authorize` фасада `Gate`. Экземпляры `AuthorizationException` автоматически преобразуются в `403` HTTP-ответ обработчиком исключений Laravel:

    Gate::authorize('update-post', $post);

    // Действие разрешено ...

<a name="gates-supplying-additional-context"></a>
#### Предоставление дополнительного контекста шлюзам

Методы шлюза для авторизации полномочий (`allows`, `denies`, `check`, `any`, `none`, `authorize`, `can`, `cannot`) и [директивы авторизации Blade](#via-blade-templates) (`@can`, `@cannot`, `@canany`) могут получать массив в качестве второго аргумента. Эти элементы массива передаются в качестве параметров замыканию шлюза и могут использоваться как дополнительный контекст при принятии решений об авторизации:

    use App\Models\Category;
    use App\Models\User;
    use Illuminate\Support\Facades\Gate;

    Gate::define('create-post', function (User $user, Category $category, $pinned) {
        if (! $user->canPublishToGroup($category->group)) {
            return false;
        } elseif ($pinned && ! $user->canPinPosts()) {
            return false;
        }

        return true;
    });

    if (Gate::check('create-post', [$category, $pinned])) {
        // Пользователь может создать пост ...
    }

<a name="gate-responses"></a>
### Ответы шлюза

До сих пор мы рассматривали шлюзы, возвращающие простые логические значения. По желанию можно вернуть более подробный ответ, содержащий также сообщение об ошибке. Для этого вы можете вернуть экземпляр `Illuminate\Auth\Access\Response` из вашего шлюза:

    use App\Models\User;
    use Illuminate\Auth\Access\Response;
    use Illuminate\Support\Facades\Gate;

    Gate::define('edit-settings', function (User $user) {
        return $user->isAdmin
                    ? Response::allow()
                    : Response::deny('Вы должны быть администратором.');
    });

Даже когда вы возвращаете ответ авторизации из вашего шлюза, метод `Gate::allows` все равно будет возвращать простое логическое значение; однако вы можете использовать метод `Gate::inspect`, чтобы получить полный возвращенный шлюзом ответ авторизации:

    $response = Gate::inspect('edit-settings');

    if ($response->allowed()) {
        // Действие разрешено ...
    } else {
        echo $response->message();
    }

При использовании метода `Gate::authorize`, который генерирует исключение `AuthorizationException` при не авторизованном действие, сообщение об ошибке ответа авторизации будет передано в HTTP-ответ:

    Gate::authorize('edit-settings');

    // Действие разрешено ...

<a name="intercepting-gate-checks"></a>
### Хуки шлюзов

Иногда бывает необходимо предоставить все полномочия конкретному пользователю. Вы можете использовать метод `before` для определения замыкания, которое выполняется перед всеми другими проверками авторизации:

    use Illuminate\Support\Facades\Gate;

    Gate::before(function ($user, $ability) {
        if ($user->isAdministrator()) {
            return true;
        }
    });

Если замыкание `before` возвращает результат, отличный от `null`, то этот результат и будет считаться результатом проверки авторизации.

Вы можете использовать метод `after` для определения замыкания, которое будет выполнено после всех других проверок авторизации:

    Gate::after(function ($user, $ability, $result, $arguments) {
        if ($user->isAdministrator()) {
            return true;
        }
    });

Подобно методу `before`, если замыкание `after` возвращает результат, отличный от `null`, то этот результат и будет считаться результатом проверки авторизации.

<a name="inline-authorization"></a>
### Упрощенная авторизация

Вы можете определить авторизован ли текущий аутентифицированный пользователь для выполнения конкретного действия без написания специального шлюза, соответствующего этому действию. Laravel позволяет вам выполнять такие типы «упрощенных» проверок авторизации с помощью методов `Gate::allowIf` и `Gate::denyIf`:

```php
use Illuminate\Support\Facades\Gate;

Gate::allowIf(fn ($user) => $user->isAdministrator());

Gate::denyIf(fn ($user) => $user->banned());
```

Если действие не авторизовано или пользователь в настоящее время не аутентифицирован, то Laravel автоматически выбросит исключение `Illuminate\Auth\Access\AuthorizationException`. Экземпляры `AuthorizationException` автоматически преобразуются в `403` HTTP-ответ обработчиком исключений Laravel.

<a name="creating-policies"></a>
## Создание политик

<a name="generating-policies"></a>
### Генерация политик

Политики – это классы, которые организуют логику авторизации для конкретной модели или ресурса. Например, если ваше приложение является блогом, то у вас может быть модель `App\Models\Post` и соответствующая политика `App\Policies\PostPolicy` для авторизации действий пользователя, например, создание или обновление постов.

Чтобы сгенерировать новую политику, используйте команду `make:policy` [Artisan](artisan.md). Эта команда поместит новый класс политики в каталог `app/Policies` вашего приложения. Если этот каталог не существует в вашем приложении, то Laravel предварительно создаст его:

```shell
php artisan make:policy PostPolicy
```

Команда `make:policy` сгенерирует пустой класс политики. Если вы хотите создать класс с заготовками методов политики, связанных с просмотром, созданием, обновлением и удалением ресурса, то вы можете указать параметр `--model` при выполнении команды:

```shell
php artisan make:policy PostPolicy --model=Post
```

<a name="registering-policies"></a>
### Регистрация политик

После создания класса политики его необходимо зарегистрировать. Регистрация политик – это то, как мы можем сообщить Laravel, какую политику использовать при авторизации действий для конкретного типа модели.

Поставщик `App\Providers\AuthServiceProvider` содержит свойство `$policies`, которое сопоставляет ваши модели Eloquent с соответствующими политиками. Регистрация политики укажет Laravel, какую политику использовать при авторизации действий для конкретной модели Eloquent:

    <?php

    namespace App\Providers;

    use App\Models\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Карта политик приложения.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Регистрация любых служб аутентификации / авторизации.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="policy-auto-discovery"></a>
#### Автообнаружение политики

Вместо того, чтобы вручную регистрировать политики модели, Laravel может автоматически обнаруживать политики, если модель и политика соответствуют стандартным соглашениям об именах Laravel. <!--В частности, политики должны находиться в каталоге `Policies` в или выше каталога, содержащего ваши модели.--> Так, например, модели могут быть помещены в каталог `app/Models`, а политики – в каталог `app/Policies`. <!--В этой ситуации Laravel проверит наличие политик в app / Models / Policies, затем app / Policies.--> Кроме того, имя политики должно совпадать с названием модели и иметь суффикс `Policy`. Итак, модель `User` будет сопоставлена классу политики `UserPolicy`.

Если вы хотите определить свою собственную логику обнаружения политики, то вы можете зарегистрировать замыкание обнаружения политики, используя метод `Gate::guessPolicyNamesUsing`. Как правило, вызов этого метода осуществляется в методе `boot` поставщика `App\Providers\AuthServiceProvider`:

    use Illuminate\Support\Facades\Gate;

    Gate::guessPolicyNamesUsing(function ($modelClass) {
        // Возвращаем имя класса политики для переданной модели ...
    });

> {note} Любые политики, которые явно отображены в вашем `AuthServiceProvider`, будут иметь **приоритет** над любыми потенциально автоматически обнаруженными политиками.

<a name="writing-policies"></a>
## Написание политик

<a name="policy-methods"></a>
### Методы политики

После регистрации класса политики вы можете добавить методы для каждого из авторизуемых действий. Например, давайте определим метод `update` в нашем классе `PostPolicy`, который решает, может ли пользователь обновить указанный экземпляр поста.

Метод `update` получит в качестве аргументов экземпляры `User` и `Post` и должен вернуть `true` или `false`, которые будут указывать, авторизован ли пользователь обновлять указанный пост. Итак, в этом примере мы проверим, что идентификатор пользователя совпадает с `user_id` поста:

    <?php

    namespace App\Policies;

    use App\Models\Post;
    use App\Models\User;

    class PostPolicy
    {
        /**
         * Определить, может ли пользователь обновить пост.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

Вы можете продолжить определение в политике необходимых методов дополнительных авторизуемых действий. Например, вы можете определить методы `view` или `delete` для авторизации различных действий, связанных с `Post`. Помните, что вы можете дать своим методам политики любые желаемые имена.

Если вы использовали опцию `--model` при создании своей политики через Artisan, то она уже будет содержать методы для следующих действий: `viewAny`, `view`, `create`, `update`, `delete`, `restore` и `forceDelete`.

> {tip} Все политики извлекаются через [контейнер служб](container.md) Laravel, что позволяет вам объявлять любые необходимые зависимости в конструкторе политики для их автоматического внедрения.

<a name="policy-responses"></a>
### Ответы политики

До сих пор мы рассматривали методы политики, возвращающие простые логические значения. По желанию можно вернуть более подробный ответ, содержащий также сообщение об ошибке. Для этого вы можете вернуть экземпляр `Illuminate\Auth\Access\Response` из вашего метода политики:

    use App\Models\Post;
    use App\Models\User;
    use Illuminate\Auth\Access\Response;

    /**
     * Определить, может ли пользователь обновить пост.
     *
     * @param  \App\Models\User  $user
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Auth\Access\Response
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id
                    ? Response::allow()
                    : Response::deny('You do not own this post.');
    }

При возврате ответа авторизации из вашей политики метод `Gate::allows` все равно будет возвращать простое логическое значение; однако вы можете использовать метод `Gate::inspect`, чтобы получить полный возвращенный шлюзом ответ авторизации:

    use Illuminate\Support\Facades\Gate;

    $response = Gate::inspect('update', $post);

    if ($response->allowed()) {
        // Действие разрешено ...
    } else {
        echo $response->message();
    }

При использовании метода `Gate::authorize`, который генерирует исключение `AuthorizationException` при не авторизованном действие, сообщение об ошибке ответа авторизации будет передано в HTTP-ответ:

    Gate::authorize('update', $post);

    // Действие разрешено ...

<a name="methods-without-models"></a>
### Методы политики без моделей

Некоторые методы политики получают только экземпляр аутентифицированного в данный момент пользователя. Эта ситуация наиболее распространена при авторизации действий `create`. Например, если вы создаете блог, то вы можете определить, имеет ли пользователь право вообще создавать какие-либо посты. В этих ситуациях ваш метод политики должен рассчитывать только на получение экземпляра пользователя:

    /**
     * Определить, может ли пользователь создать пост.
     *
     * @param  \App\Models\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        return $user->role == 'writer';
    }

<a name="guest-users"></a>
### Гостевые пользователи

По умолчанию все шлюзы и политики автоматически возвращают `false`, если входящий HTTP-запрос был инициирован не аутентифицированным пользователем. Однако вы можете разрешить прохождение этих проверок авторизации к вашим шлюзам и политикам, пометив в аргументе метода объявленный тип `User` как [обнуляемый](https://www.php.net/manual/ru/language.types.declarations.php#language.types.declarations.nullable), путём добавления префикса в виде знака вопроса (`?`). Это означает, что значение может быть как объявленного типа `User`, так и быть равным `null`:

    <?php

    namespace App\Policies;

    use App\Models\Post;
    use App\Models\User;

    class PostPolicy
    {
        /**
         * Определить, может ли пользователь обновить пост.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Post  $post
         * @return bool
         */
        public function update(?User $user, Post $post)
        {
            return optional($user)->id === $post->user_id;
        }
    }

<a name="policy-filters"></a>
### Фильтры политики

Для определенных пользователей вы можете разрешить все действия в рамках конкретной политики. Для этого определите в политике метод `before`. Метод `before` будет выполнен перед любыми другими методами в политике, что даст вам возможность авторизовать действие до фактического вызова предполагаемого метода политики. Этот функционал чаще всего используется для авторизации администраторов приложения на выполнение любых действий:

    use App\Models\User;

    /**
     * Выполнить предварительную авторизацию.
     *
     * @param  \App\Models\User  $user
     * @param  string  $ability
     * @return void|bool
     */
    public function before(User $user, $ability)
    {
        if ($user->isAdministrator()) {
            return true;
        }
    }

Если вы хотите отклонить все проверки авторизации для определенного типа пользователей, вы можете вернуть `false` из метода `before`. Если возвращается `null`, то проверка авторизации перейдет к методу политики.

> {note} Метод `before` класса политики не будет вызываться, если класс не содержит метода с именем, совпадающим с именем проверяемого полномочия.

<a name="authorizing-actions-using-policies"></a>
## Авторизация действий с помощью политик

<a name="via-the-user-model"></a>
### Авторизация действий с помощью политик через модель User

Модель `App\Models\User` приложения Laravel включает два полезных метода авторизации действий: `can` и `cannot`. Методы `can` и `cannot` получают имя действия, которое вы хотите авторизовать, и соответствующую модель. Например, давайте определим, авторизован ли пользователь для обновления переданной модели `App\Models\Post`. Обычно это делается в методе контроллера:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Обновить переданный пост.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \App\Models\Post  $post
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request, Post $post)
        {
            if ($request->user()->cannot('update', $post)) {
                abort(403);
            }

            // Обновление поста ...
        }
    }

Если [политика зарегистрирована](#registering-policies) для данной модели, то метод `can` автоматически вызовет соответствующую политику и вернет логический результат. Если для модели не зарегистрирована политика, то метод `can` попытается вызвать шлюз, основанный на замыкании, соответствующий переданному имени действия.

<a name="user-model-actions-that-dont-require-models"></a>
#### Авторизация действий, не требующих моделей, с помощью политик через модель User

Помните, что некоторые действия могут соответствовать методам политики, например `create`, которые не требуют экземпляра модели. В этих ситуациях вы можете передать имя класса методу `can`. Имя класса будет использоваться для определения того, какую политику использовать при авторизации действия:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Сохранить пост.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            if ($request->user()->cannot('create', Post::class)) {
                abort(403);
            }

            // Сохранение поста ...
        }
    }

<a name="via-controller-helpers"></a>
### Авторизация действий с помощью политик через помощников контроллера

В дополнение к методам модели `App\Models\User`, контроллеры Laravel, расширяющие базовый класс `App\Http\Controllers\Controller`, содержат полезный метод `authorize`.

Подобно методу `can`, этот метод принимает имя действия, которое вы хотите авторизовать, и соответствующую модель. Если действие не авторизовано, то метод `authorize` выбросит исключение `Illuminate\Auth\Access\AuthorizationException`, которое обработчик исключений Laravel автоматически преобразует в `403` HTTP-ответ:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Обновить переданный пост.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \App\Models\Post  $post
         * @return \Illuminate\Http\Response
         *
         * @throws \Illuminate\Auth\Access\AuthorizationException
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // Текущий пользователь может обновить пост в блоге ...
        }
    }

<a name="controller-actions-that-dont-require-models"></a>
#### Авторизация действий, не требующих моделей, с помощью политик через помощников контроллера

Как обсуждалось ранее, некоторые методы политики, например `create`, не требуют экземпляра модели. В таких ситуациях вы должны передать имя класса методу `authorize`. Имя класса будет использоваться для определения того, какую политику использовать при авторизации действия:

    use App\Models\Post;
    use Illuminate\Http\Request;

    /**
     * Создайте новый пост в блоге.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // Текущий пользователь может создавать посты в блоге ...
    }

<a name="authorizing-resource-controllers"></a>
#### Авторизация ресурсных контроллеров с помощью политик

Если вы используете [ресурсные контроллеры](controllers.md#resource-controllers), то вы можете использовать метод `authorizeResource` в конструкторе вашего контроллера. Этот метод добавит соответствующие определения посредника `can` к методам ресурсного контроллера.

Метод `authorizeResource` принимает имя класса модели в качестве своего первого аргумента и имя параметра маршрута / запроса, который будет содержать идентификатор модели, в качестве второго аргумента. Вы должны убедиться, что ваш ресурсный контроллер создан с использованием флага `--model`, чтобы он имел необходимые сигнатуры методов и объявления типов:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * Создать экземпляр контроллера.
         *
         * @return void
         */
        public function __construct()
        {
            $this->authorizeResource(Post::class, 'post');
        }
    }

Следующие методы контроллера будут сопоставлены соответствующим методам политики. Когда запросы направляются к методу контроллера, тогда перед выполнением метода контроллера будет автоматически вызываться соответствующий метод политики:

| Метод контроллера | Метод политики |
| --- | --- |
| index | viewAny |
| show | view |
| create | create |
| store | create |
| edit | update |
| update | update |
| destroy | delete |

> {tip} Вы можете использовать команду `make:policy` с параметром `--model`, чтобы сгенерировать класс политики для указанной модели: `php artisan make:policy PostPolicy --model=Post`.

<a name="via-middleware"></a>
### Авторизация действий с помощью политик через посредника

Laravel содержит посредника, который может авторизовать действия до того, как входящий запрос достигнет ваших маршрутов или контроллеров. По умолчанию посреднику `Illuminate\Auth\Middleware\Authorize` назначается ключ `can` в вашем классе `App\Http\Kernel`. Давайте рассмотрим пример использования посредника `can` для авторизации того, что пользователь может обновлять пост:

    use App\Models\Post;

    Route::put('/post/{post}', function (Post $post) {
        // Текущий пользователь может обновить пост ...
    })->middleware('can:update,post');

В этом примере мы передаем посреднику `can` два аргумента. Первый – это имя действия, которое мы хотим авторизовать, а второй – параметр маршрута, который мы хотим передать методу политики. В этом случае, поскольку мы используем [неявную привязку модели](routing.md#implicit-binding), то методу политики будет передана модель `App\Models\Post`. Если пользователь не авторизован для выполнения указанного действия, то посредник вернет ответ HTTP с кодом состояния `403`.

Для удобства вы также можете назначить посредник `can` вашему маршруту, используя метод `can`:

    use App\Models\Post;

    Route::put('/post/{post}', function (Post $post) {
        // Текущий пользователь может обновить пост ...
    })->can('update', 'post');

<a name="middleware-actions-that-dont-require-models"></a>
#### Авторизация действий, не требующих моделей, с помощью политик через посредника

Опять же, некоторые методы политики, например `create`, не требуют экземпляра модели. В этих ситуациях вы можете передать имя класса посреднику. Имя класса будет использоваться для определения того, какую политику использовать при авторизации действия:

    Route::post('/post', function () {
        // Текущий пользователь может создавать посты ...
    })->middleware('can:create,App\Models\Post');

Указание полного имени класса строкой в определении посредника может стать громоздким. По этой причине вы можете назначить посредник `can` вашему маршруту, используя метод `can`:

    use App\Models\Post;

    Route::post('/post', function () {
        // Текущий пользователь может создавать посты ...
    })->can('create', Post::class);

<a name="via-blade-templates"></a>
### Авторизация действий с помощью политик через шаблоны Blade

При написании шаблонов Blade бывает необходимо отобразить часть страницы только в том случае, если пользователь авторизован для выполнения конкретного действия. Например, вы можете показать форму обновления поста в блоге, только если пользователь действительно уполномочен обновить сообщение. В этой ситуации вы можете использовать директивы `@can` и `@cannot`:

```blade
@can('update', $post)
    <!-- Текущий пользователь может обновить пост ... -->
@elsecan('create', App\Models\Post::class)
    <!-- Текущий пользователь может создавать новые посты ... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- Текущий пользователь не может обновить пост ... -->
@elsecannot('create', App\Models\Post::class)
    <!-- Текущий пользователь не может создавать новые посты ... -->
@endcannot
```

Эти директивы являются удобными ярлыками выражений `@if` и `@unless`. Приведенные выше директивы `@can` и `@cannot` эквивалентны следующим выражениям:

```blade
@if (Auth::user()->can('update', $post))
    <!-- Текущий пользователь может обновить пост ... -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- Текущий пользователь не может обновить пост ... -->
@endunless
```

Вы также можете определить, авторизован ли пользователь для выполнения любого из указанных в массиве действия. Для этого используйте директиву `@canany`:

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- Текущий пользователь может обновить, просмотреть или удалить пост ... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- Текущий пользователь может создать пост ... -->
@endcanany
```

<a name="blade-actions-that-dont-require-models"></a>
#### Авторизация действий, не требующих моделей, с помощью политик через шаблоны Blade

Как и большинство других методов авторизации, вы можете передать имя класса в директивы `@can` и `@cannot`, если для действия не требуется экземпляр модели:

```blade
@can('create', App\Models\Post::class)
    <!-- Текущий пользователь может создавать посты ... -->
@endcan

@cannot('create', App\Models\Post::class)
    <!-- Текущий пользователь не может создавать посты ... -->
@endcannot
```

<a name="supplying-additional-context"></a>
### Предоставление дополнительного контекста политики

При авторизации действий с использованием политик вы можете передать массив в качестве второго аргумента различным функциям авторизации и помощникам. Первый элемент в массиве будет использоваться для определения того, какая политика должна быть вызвана, в то время как остальные элементы массива передаются как параметры методу политики и могут использоваться как дополнительный контекст при принятии решений об авторизации. Например, рассмотрим `PostPolicy` и следующее определение метода, содержащего дополнительный параметр `$category`:

    /**
     * Определить, может ли пользователь обновить пост.
     *
     * @param  \App\Models\User  $user
     * @param  \App\Models\Post  $post
     * @param  int  $category
     * @return bool
     */
    public function update(User $user, Post $post, int $category)
    {
        return $user->id === $post->user_id &&
               $user->canUpdateCategory($category);
    }

При попытке определить, может ли аутентифицированный пользователь обновить указанный пост, мы можем вызвать этот метод политики следующим образом:

    /**
     * Обновить конкретный пост.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', [$post, $request->category]);

        // Текущий пользователь может обновить пост в блоге ...
    }
