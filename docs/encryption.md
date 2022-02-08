# Laravel 9 · Шифрование

- [Введение](#introduction)
- [Конфигурирование](#configuration)
- [Использование шифровальщика](#using-the-encrypter)

<a name="introduction"></a>
## Введение

Сервисы шифрования Laravel предоставляют простой и удобный интерфейс для шифрования и дешифрования текста через OpenSSL с использованием шифрования AES-256 и AES-128. Все зашифрованные значения Laravel подписываются с использованием кода аутентификации сообщения (MAC), поэтому их базовое значение не может быть изменено или подделано после шифрования.

<a name="configuration"></a>
## Конфигурирование

Перед использованием шифровальщика Laravel вы должны установить параметр `key` в конфигурационном файле `config/app.php`. Это значение конфигурации управляется переменной окружения `APP_KEY`. Вы должны использовать команду `php artisan key:generate` для генерации значения этой переменной, поскольку команда `key:generate` будет использовать безопасный генератор случайных байтов PHP для создания криптографически безопасного ключа для вашего приложения. Обычно значение переменной среды `APP_KEY` генерируется для вас во время [установки Laravel](installation.md).

<a name="using-the-encrypter"></a>
## Использование шифровальщика

<a name="encrypting-a-value"></a>
#### Шифрование значения

Вы можете зашифровать значение, используя метод `encryptString` фасада `Crypt`. Все значения будут зашифрованы с использованием OpenSSL и шифра `AES-256-CBC`. Кроме того, все зашифрованные значения подписываются кодом аутентификации сообщения (MAC). Встроенный код аутентификации сообщений предотвратит расшифровку любых значений, которые были подделаны злоумышленниками:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class DigitalOceanTokenController extends Controller
    {
        /**
         * Сохраните DigitalOcean API-токен пользователя.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function storeSecret(Request $request)
        {
            $request->user()->fill([
                'token' => Crypt::encryptString($request->token),
            ])->save();
        }
    }

<a name="decrypting-a-value"></a>
#### Расшифровка значения

Вы можете расшифровать значения, используя метод `decryptString` фасада `Crypt`. Если значение не может быть правильно расшифровано, например, когда код аутентификации сообщения недействителен, будет выброшено исключение `Illuminate\Contracts\Encryption\DecryptException`:

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
