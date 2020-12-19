# Хеширование

- [Введение](#introduction)
- [Конфигурирование](#configuration)
- [Основы использования](#basic-usage)

<a name="introduction"></a>
## Введение

[Фасад](facades.md) `Hash` Laravel обеспечивает безопасное хеширование Bcrypt и Argon2 для хранения паролей пользователей. Если вы используете каркас аутентификации [Laravel Jetstream](https://jetstream.laravel.com), то по умолчанию для регистрации и аутентификации будет использоваться Bcrypt.

> {tip} Bcrypt – отличный выбор для хеширования паролей, потому что его «коэффициент работы» регулируется, а это означает, что время, необходимое для генерации хеш-кода, может быть увеличено по мере увеличения мощности оборудования.

<a name="configuration"></a>
## Конфигурирование

Драйвер хеширования по умолчанию для вашего приложения настраивается в файле конфигурации `config/hashing.php`. В настоящее время существует три поддерживаемых драйвера: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) и [Argon2](https://en.wikipedia.org/wiki/Argon2) (вариации Argon2i и Argon2id).

> {note} Для драйвера Argon2i требуется PHP 7.2.0 или выше, а для драйвера Argon2id требуется PHP 7.3.0 или выше.

<a name="basic-usage"></a>
## Основы использования

Вы можете хэшировать пароль, вызвав метод `make` фасада `Hash`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class UpdatePasswordController extends Controller
    {
        /**
         * Обновить пароль пользователя.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Проверить длину нового пароля ...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

<a name="adjusting-the-bcrypt-work-factor"></a>
#### Регулировка коэффициента работы Bcrypt

Если вы используете алгоритм Bcrypt, метод `make` позволяет вам управлять коэффициентом работы алгоритма с помощью параметра `rounds`; однако значение по умолчанию приемлемо для большинства приложений:

    $hashed = Hash::make('password', [
        'rounds' => 12,
    ]);

<a name="adjusting-the-argon2-work-factor"></a>
#### Регулировка коэффициента работы Argon2

Если вы используете алгоритм Argon2, метод `make` позволяет вам управлять коэффициентом работы алгоритма с помощью параметров `memory`, `time` и `threads`; однако значения по умолчанию приемлемы для большинства приложений:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> {tip} Дополнительную информацию об этих параметрах можно найти в [официальной документации PHP](https://secure.php.net/manual/en/function.password-hash.php).

<a name="verifying-a-password-against-a-hash"></a>
#### Проверка пароля по хешу

Метод `check` позволяет проверить, что указанная текстовая строка соответствует заданному хешу:

    if (Hash::check('plain-text', $hashedPassword)) {
        // Пароли совпадают ...
    }

<a name="checking-if-a-password-needs-to-be-rehashed"></a>
#### Проверка необходимости повторного хеширования пароля

Метод `needsRehash` позволяет определить, изменился ли коэффициентом работы, используемый хешером, с момента хеширования пароля:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
