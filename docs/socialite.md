# Laravel 9 · Пакет Laravel Socialite

- [Введение](#introduction)
- [Установка](#installation)
- [Обновление пакета Socialite](#upgrading-socialite)
- [Конфигурирование](#configuration)
- [Аутентификация](#authentication)
    - [Маршрутизация](#routing)
    - [Аутентификация и хранение пользователей](#authentication-and-storage)
    - [Права доступа](#access-scopes)
    - [Необязательные параметры](#optional-parameters)
- [Получение сведений о пользователе](#retrieving-user-details)

<a name="introduction"></a>
## Введение

Помимо типичной аутентификации на основе форм, Laravel также предлагает простой и удобный способ аутентификации через провайдеров OAuth с помощью [Laravel Socialite](https://github.com/laravel/socialite). Socialite в настоящее время поддерживает аутентификацию через Facebook, Twitter, LinkedIn, Google, GitHub, GitLab, и Bitbucket.

> {tip} Адаптеры для других платформ перечислены на веб-сайте [Socialite Providers](https://socialiteproviders.com/), управляемом сообществом.

<a name="installation"></a>
## Установка

Для начала установите Socialite с помощью менеджера пакетов Composer в свой проект:

```shell
composer require laravel/socialite
```

<a name="upgrading-socialite"></a>
## Обновление пакета Socialite

При обновлении Socialite важно внимательно изучить [руководство по обновлению](https://github.com/laravel/socialite/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Конфигурирование

Перед использованием Socialite вам нужно будет добавить учетные данные для провайдеров OAuth, которые использует ваше приложение. Как правило, эти учетные данные можно получить, создав «приложение разработчика» на панели инструментов службы, с которой вы будете проходить аутентификацию.

Эти учетные данные должны быть размещены в файле конфигурации вашего приложения `config/services.php` и должны использовать ключ `facebook`, `twitter` (OAuth 1.0), `twitter-oauth-2` (OAuth 2.0), `linkedin`, `google`, `github`, `gitlab` или `bitbucket`, в зависимости от провайдеров, которые требуются вашему приложению:

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => 'http://example.com/callback-url',
    ],

> {tip} Если параметр `redirect` содержит относительный путь, то он будет автоматически преобразован в абсолютный URL.

<a name="authentication"></a>
## Аутентификация

<a name="routing"></a>
### Маршрутизация

Для аутентификации пользователей с помощью провайдера OAuth вам понадобятся два маршрута: один для перенаправления пользователя к провайдеру OAuth, а другой для получения обратного вызова от провайдера после аутентификации. Пример ниже демонстрирует реализацию обоих маршрутов:

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/redirect', function () {
        return Socialite::driver('github')->redirect();
    });

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // $user->token
    });

Метод `redirect` фасада `Socialite` обеспечивает перенаправление пользователя к провайдеру OAuth, а метод `user` обрабатывает входящий запрос и извлекает информацию о пользователе от провайдера после того, как он подтвердил запрос на аутентификацию.

<a name="authentication-and-storage"></a>
### Аутентификация и хранение пользователей

После того, как пользователь был получен от провайдера OAuth, вы можете определить, существует ли пользователь в базе данных вашего приложения, и [аутентифицировать пользователя](authentication.md#authenticate-a-user-instance). Если пользователь не существует в базе данных вашего приложения, то вы можете создать новую запись в своей базе данных для представления пользователя:

    use App\Models\User;
    use Illuminate\Support\Facades\Auth;
    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $githubUser = Socialite::driver('github')->user();

        $user = User::updateOrCreate([
            'github_id' => $githubUser->id,
        ], [
            'name' => $githubUser->name,
            'email' => $githubUser->email,
            'github_token' => $githubUser->token,
            'github_refresh_token' => $githubUser->refreshToken,
        ]);

        Auth::login($user);

        return redirect('/dashboard');
    });

> {tip} О том, какая информация о пользователе доступна у конкретных провайдеров OAuth, ознакомьтесь с документацией по [получению сведений о пользователе](#retrieving-user-details).

<a name="access-scopes"></a>
### Права доступа

Перед перенаправлением пользователя вы можете использовать метод `scopes`, чтобы добавить дополнительные «права» к запросу аутентификации. Этот метод объединит все существующие права с указанными:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

Вы можете перезаписать все существующие права в запросе аутентификации, используя метод `setScopes`:

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="optional-parameters"></a>
### Необязательные параметры

Некоторые провайдеры OAuth поддерживают другие необязательные параметры запроса перенаправления. Чтобы включить в запрос любые необязательные параметры, вызовите метод `with` с ассоциативным массивом:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> {note} При использовании метода `with` будьте осторожны, чтобы не передавать какие-либо зарезервированные ключевые слова, такие как `state` или `response_type`.

<a name="retrieving-user-details"></a>
## Получение сведений о пользователе

После того, как пользователь будет перенаправлен обратно на маршрут `callback` аутентификации вашего приложения, вы можете получить данные пользователя, используя метод `user` Socialite. Объект пользователя, возвращаемый методом `user`, содержит множество свойств и методов, которые вы можете использовать для сохранения информации о пользователе в вашей собственной базе данных.

Для этого объекта могут быть доступны различные свойства и методы в зависимости от версии провайдера OAuth, с которым вы выполняете аутентификацию:

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // Провайдер OAuth 2.0 ...
        $token = $user->token;
        $refreshToken = $user->refreshToken;
        $expiresIn = $user->expiresIn;

        // Провайдер OAuth 1.0 ...
        $token = $user->token;
        $tokenSecret = $user->tokenSecret;

        // Все провайдеры ...
        $user->getId();
        $user->getNickname();
        $user->getName();
        $user->getEmail();
        $user->getAvatar();
    });

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### Получение сведений о пользователе из токена (OAuth2)

Если у вас уже есть действительный токен доступа пользователя, то вы можете получить данные о пользователе с помощью метода `userFromToken` пакета Socialite:

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('github')->userFromToken($token);

<a name="retrieving-user-details-from-a-token-and-secret-oauth1"></a>
#### Получение сведений о пользователе из токена и секретного ключа (OAuth1)

Если у вас уже есть действительный токен и секретный ключ пользователя, то вы можете получить данные о пользователе с помощью метода `userFromTokenAndSecret` пакета Socialite:

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);

<a name="stateless-authentication"></a>
#### Аутентификация без сохранения состояния

Метод `stateless` может использоваться для отключения проверки состояния сессии. Это полезно при добавлении социальной аутентификации в API без сохранения состояния, который не использует сессии на основе файлов куки:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')->stateless()->user();

> {note} Аутентификация без сохранения состояния недоступна для драйвера Twitter OAuth 1.0.
