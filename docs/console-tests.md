# Консольные тесты

- [Введение](#introduction)
- [Ожидание ввода / вывода](#expecting-input-and-output)

<a name="introduction"></a>
## Введение

Помимо упрощения HTTP-тестирования, Laravel предоставляет простой API для тестирования консольных приложений, которые запрашивают ввод данных пользователем.

<a name="expecting-input-and-output"></a>
## Ожидание ввода / вывода

Laravel позволяет вам легко «имитировать» ввод пользователем в консольных командах, используя метод `expectsQuestion`. Кроме того, вы можете указать код выхода / возврата и текст, который вы ожидаете получить от консольной команды, используя методы `assertExitCode` и `expectsOutput`. Например, рассмотрим следующую консольную команду:

    Artisan::command('question', function () {
        $name = $this->ask('What is your name?');

        $language = $this->choice('Which language do you program in?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Your name is '.$name.' and you program in '.$language.'.');
    });

Вы можете протестировать эту команду с помощью следующего теста, который использует методы `expectsQuestion`,` expectsOutput` и `assertExitCode`:

    /**
     * Тестирование консольной команды.
     *
     * @return void
     */
    public function testConsoleCommand()
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you program in?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you program in PHP.')
             ->assertExitCode(0);
    }

При написании команды, которая ожидает подтверждения в виде ответа «да» или «нет», вы можете использовать метод `expectsConfirmation`:

    $this->artisan('module:import')
        ->expectsConfirmation('Do you really wish to run this command?', 'no')
        ->assertExitCode(1);
