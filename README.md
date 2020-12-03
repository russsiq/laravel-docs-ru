## Документация Laravel

Вольный перевод [репозитория](https://github.com/laravel/docs/tree/8.x) документации **laravel/docs** ветка 8.x.

Ссылка на [лицензию](https://github.com/laravel/docs/blob/8.x/license.md) оригинала документации **laravel/docs**.

Перед чтением документации ознакомьтесь с [соглашением по переводу](#dictionary).

### Содержание документации

Перевод документации пополняется по мере обращения к соответствующим разделам.

- ## Пролог
    - [Примечания к выпуску](/docs/releases.md)
    - [Руководство по обновлению](/docs/upgrade.md)
    - [Рекомендации по участию](/docs/contributions.md)
- ## Начало
    - [Установка](/docs/installation.md)
    - [Конфигурирование](/docs/configuration.md)
    - [Структура каталогов](/docs/structure.md)
    - [Homestead](/docs/homestead.md)
    - [Valet](/docs/valet.md)
    - [Развертывание](/docs/deployment.md)
- ## Архитектурные концепции
    - [Жизненный цикл запроса](/docs/lifecycle.md)
    - [Контейнер служб](/docs/container.md)
    - [Поставщики служб](/docs/providers.md)
    - [Фасады](/docs/facades.md)
    - [Контракты](/docs/contracts.md)
- ## Основы
    - [Маршрутизация](/docs/routing.md)
    - [Посредник](/docs/middleware.md)
    - [Предотвращение атак CSRF](/docs/csrf.md)
    - [Контроллеры](/docs/controllers.md)
    - [HTTP-запросы](/docs/requests.md)
    - [HTTP-ответы](/docs/responses.md)
    - [Шаблоны HTML](/docs/views.md)
    - [Генерация URL-адресов](/docs/urls.md)
    - [Сессия](/docs/session.md)
    - [Валидация](/docs/validation.md)
    - [Обработка ошибок](/docs/errors.md)
    - [Логирование](/docs/logging.md)
- ## Клиентская часть
    - [Шаблонизатор Blade](/docs/blade.md)
    - [Локализация](/docs/localization.md)
    - [Компиляция исходников](/docs/mix.md)
- ## Безопасность
    - [Аутентификация](/docs/authentication.md)
    - [Авторизация](/docs/authorization.md)
    - [Подтверждение адреса электронной почты](/docs/verification.md)
    - [Шифрование](/docs/encryption.md)
    - [Хеширование](/docs/hashing.md)
    - [Сброс пароля](/docs/passwords.md)
- ## Углубленное изучение
    - [Консоль Artisan](/docs/artisan.md)
    - [Широковещание](/docs/broadcasting.md)
    - [Кэш](/docs/cache.md)
    - [Коллекции](/docs/collections.md)
    - [События](/docs/events.md)
    - [Файловое хранилище](/docs/filesystem.md)
    - [Помощники](/docs/helpers.md)
    - [HTTP-клиент](/docs/http-client.md)
    - [Почта](/docs/mail.md)
    - [Уведомления](/docs/notifications.md)
    - [Разработка пакетов](/docs/packages.md)
    - [Очереди](/docs/queues.md)
    - [Планирование задач](/docs/scheduling.md)
- ## База данных
    - [Начало](/docs/database.md)
    - [Построитель запросов](/docs/queries.md)
    - [Постраничная навигация](/docs/pagination.md)
    - [Миграции](/docs/migrations.md)
    - [Наполнение](/docs/seeding.md)
    - [Redis](/docs/redis.md)
- ## Eloquent ORM
    - [Начало](/docs/eloquent.md)
    - [Отношения](/docs/eloquent-relationships.md)
    - [Коллекции](/docs/eloquent-collections.md)
    - [Мутаторы](/docs/eloquent-mutators.md)
    - [Ресурсы API](/docs/eloquent-resources.md)
    - [Сериализация](/docs/eloquent-serialization.md)
- ## Тестирование
    - [Начало](/docs/testing.md)
    - [Тесты HTTP](/docs/http-tests.md)
    - [Консольные тесты](/docs/console-tests.md)
    - [Браузерные тесты](/docs/dusk.md)
    - [База данных](/docs/database-testing.md)
    - [Имитация](/docs/mocking.md)
- ## Пакеты
    - [Cashier (Stripe)](/docs/billing.md)
    - [Cashier (Paddle)](/docs/cashier-paddle.md)
    - [Cashier (Mollie)](https://github.com/laravel/cashier-mollie)
    - [Dusk](/docs/dusk.md)
    - [Envoy](/docs/envoy.md)
    - [Horizon](/docs/horizon.md)
    - [Jetstream](https://jetstream.laravel.com)
    - [Passport](/docs/passport.md)
    - [Sanctum](/docs/sanctum.md)
    - [Scout](/docs/scout.md)
    - [Socialite](/docs/socialite.md)
    - [Telescope](/docs/telescope.md)
- [Документация API](https://laravel.com/api/8.x/)

<a name="dictionary"></a>
### Соглашения по переводу

В следующей таблице перечислены варианты принятых переводов некоторых терминов и словосочетаний, используемых в переводе документации для общей согласованности.

**Оглавление**
0-9
[A](#dictionary-a)
B
[C](#dictionary-c)
[D](#dictionary-d)
[E](#dictionary-e)
[F](#dictionary-f)
[G](#dictionary-g)
[H](#dictionary-h)
I
[J](#dictionary-j)
K
[L](#dictionary-l)
[M](#dictionary-m)
[N](#dictionary-n)
[O](#dictionary-o)
[P](#dictionary-p)
[Q](#dictionary-q)
[R](#dictionary-r)
[S](#dictionary-s)
[T](#dictionary-t)
U
[V](#dictionary-v)
[W](#dictionary-w)
X
Y
Z

Исходный вариант  |  Вариации перевода  |  Замечания
----------------- | ------------------- | -------------
<a name="dictionary-a">**A**</a>
abilities  |  полномочия, компетенции  |  
accessors  |  аксессоры  |  
argument  |  аргумент  |  
asset  |  исходник  |  
assets  |  исходники  |  Директория исходных файлов проекта для фронтенда
attach a relation  |  назначить отношение  |  
attaching  |  прикрепление  |  
attempting  |  пытается  |  
authentication  |  аутентификация  |  
authorization  |  авторизация  |  
auto-incrementing ID  |  автоинкрементный идентификатор  |  
<a name="dictionary-b">**B**</a>
bag, message bag, attribute bag  |  коллекция, коллекция сообщений, коллекция атрибутов  |  
binding  |  связывание  |  
<a name="dictionary-c">**C**</a>
cache  |  кэш  |  
casting  |  типизация  |  
cast type  |  типизатор  |  
chunking  |  разбиение  |  
clauses  |  условия  |  
сlosure  |  замыкание  |  
command line interface (CLI)  |  интерфейс командной строки  |  
condition  |  условие  |  
conditional  |  кондиционный  |  
configuration  |  конфигурация, конфигурирование  |  
constraints  |  ограничения, условия  |  
contract  |  контракт  |  
custom  |  пользовательский, желаемый  |  
customization  |  настройка  |  
customizing  |  корректировка  |  
<a name="dictionary-d">**D**</a>
defining  |  определение  |  
deployment  |  развертывание  |  
detaching  |  отсоединение  |  
dispatch  |  выполнение, исполнение  |  
dump  |  вывод, отображение  |  
<a name="dictionary-e">**E**</a>
eager  |  нетерпеливый  |  
encrypter  |  шифровальщик  |  
encryption  |  шифрование  |  
environment  |  среда, окружение  |  
evaluated  |  проанализированы  |  
event  |  событие  |  
<a name="dictionary-f">**F**</a>
facade  |  фасад  |  
factory callbacks  |  хуки фабрики  |  
fake  |  фальсификация, фальшивка  |  
foreign key constraints  |  ограничения внешнего ключа  |  
flash data  |  флеш-данные  |  Данные, имеющие непродолжительный срок существования
flashed to the session  |  записаны (краткосрочно) в сессию  |  
<a name="dictionary-g">**G**</a>
given  |  переданный, указанный  |  
guard  |  охранник (аутентификатора)  |  
guarding  |  защита  |  
<a name="dictionary-h">**H**</a>
hashing  |  хеширование  |  
helpers  |  помощники, глобальные вспомогательные функции  |  
<a name="dictionary-j">**J**</a>
job  |  задание  |  
<a name="dictionary-l">**L**</a>
layer over  |  обертка над  |  
lazy  |  отложенный  |  
listener  |  слушатель  |  
log  |  журнал  |  
logger  |  регистратор  |  
logging  |  логирование  |  
localization  |  локализация  |  
<a name="dictionary-m">**M**</a>
making  |  инициализация  |  Например, метод фабрики `make` инициализирует, но не создает запись о модели в БД.
mass assignment  |  массовое присвоение  |  
middleware  |  посредник  |  
migration  |  миграция  |  
miscellaneous  |  разное  |  
mock  |  подставной объект  |  
mocking  |  имитация  |  
mutator  |  мутатор  |  
<a name="dictionary-n">**N**</a>
named error bag  |  массив именованных ошибок  |  
<a name="dictionary-o">**O**</a>
observers  |  наблюдатели  |  
options  |  параметры  |  
overriding  |  переопределение  |  
<a name="dictionary-p">**P**</a>
pagination  |  постраничная навигация  |  
placeholder  |  заполнитель, метка-заполнитель, символ-заполнитель, заменитель  |  
policy  |  политика  |  
production  |  эксплуатация  |  
<a name="dictionary-q">**Q**</a>
query builder  |  построитель запросов  |  
query constraints  |  ограничения запроса  |  
queue  |  очередь  |  
<a name="dictionary-r">**R**</a>
redirect  |  перенаправление  |  
redirector  |  перенаправитель  |  
refreshing  |  обновление  |  
reflection  |  рефлексия (отражение)  |  
relationships  |  отношения  |  
replicating  |  репликация, тиражирование  |  
request  |  запрос  |  
resource  |  ресурс  |  Источник информации, например, API-ресурс
resources  |  ресурсы  |  Директория исходных файлов проекта для фронтенда
response  |  ответ  |  
retrieving  |  получение, извлечение  |  
route  |  маршрут  |  
routing  |  маршрутизация  |  
<a name="dictionary-s">**S**</a>
scope  |  область, сфера, диапазон  |  То, что имеет ограничения
seeding  |  наполнение  |  
serialization  |  сериализация  |  
service container  |  контейнер служб  |  
service providers  |  поставщики служб  |  
session  |  сессия  |  
soft deleting  |  программное удаление  |  
spy on an object  |  наблюдать за объектом  |  
specific  |  конкретный, определенный  |  
statement  |  оператор  |  
stub  |  заглушка  |  
<a name="dictionary-t">**T**</a>
terms of service  |  условия использования  |  
timestamp  |  временная метка  |  
toggling  |  переключение  |  
<a name="dictionary-v">**V**</a>
validation  |  валидация, проверка  |  
validator  |  валидатор  |  
vendor  |  поставщик  |  Директория `vendor` с библиотеками от сторонних поставщиков
verb, HTTP  |  HTTP-метод  |  
verification  |  подтверждение  |  
view  |  шаблон, представление  |  
<a name="dictionary-w">**W**</a>
wildcard  |  метасимвол подстановки  |  
