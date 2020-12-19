# Шифрование

- [Введение](#introduction)
- [Конфигурирование](#configuration)
- [Использование шифровальщика](#using-the-encrypter)

<a name="introduction"></a>
## Введение

Шифровальщик Laravel использует OpenSSL для обеспечения шифрования AES-256 и AES-128. Вам настоятельно рекомендуется использовать встроенные средства шифрования Laravel и не пытаться использовать свои собственные алгоритмы шифрования. Все зашифрованные значения Laravel подписываются с использованием кода аутентификации сообщения (MAC), поэтому их базовое значение не может быть изменено после шифрования.

<a name="configuration"></a>
## Конфигурирование

Перед использованием шифровальщика Laravel вы должны установить параметр `key` в конфигурационном файле `config/app.php`. Вы должны использовать команду `php artisan key:generate` для генерации этого ключа, поскольку эта Artisan-команда будет использовать безопасный генератор случайных байтов PHP для создания вашего ключа. Если это значение не установлено должным образом, все значения, зашифрованные Laravel, будут небезопасными.

<a name="using-the-encrypter"></a>
## Использование шифровальщика

<a name="encrypting-a-value"></a>
#### Шифрование значения

Вы можете зашифровать значение, используя метод `encryptString` фасада `Crypt`. Все значения будут зашифрованы с использованием OpenSSL и шифра `AES-256-CBC`. Кроме того, все зашифрованные значения подписываются кодом аутентификации сообщения (MAC) для обнаружения любых изменений зашифрованной строки:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class UserController extends Controller
    {
        /**
         * Сохраните зашифрованное сообщение пользователя.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => Crypt::encryptString($request->secret),
            ])->save();
        }
    }

<a name="decrypting-a-value"></a>
#### Расшифровка значения

Вы можете расшифровать значения, используя метод `decryptString` фасада `Crypt`. Если значение не может быть правильно расшифровано, например, когда MAC недействителен, будет выброшено исключение `Illuminate\Contracts\Encryption\DecryptException`:

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
