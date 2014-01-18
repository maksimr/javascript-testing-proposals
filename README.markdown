# The Good Parts


# The Bad Parts

## Люблю тестировать все возможные и невозможные случаи

Bad:
```javascript
    it('должен возвращать true на всякий разный юникод', function() {
        for (var i = 0x0374; i < 0x1000; i++) {
            (function(letter) {
                expect(password.containsInvalidCharacters(letter)).to.be(true);
            })(String.fromCharCode(i));
        }
    })
```
- Замедляет выполнение тестов
- Трата времени и ресурсов на похожие случаи

Good:
```javascript
    it('должен возвращать true на всякий разный юникод', function() {
        expect(password.containsInvalidCharacters(String.fromCharCode(0x0374))).to.be(true);
        expect(password.containsInvalidCharacters(String.fromCharCode(0x1000))).to.be(true);
    })
```
+ Необходимо тестировать:
  + Граничные условия
  + Случае где вероятнее всего может возникнуть ошибки (Пример: деление на ноль)
  + Найденные ошибки

## Генерация тестов

```javascript
    describe('onPasswordKeyup', function() {
        var methods = {
            'showError': {
                test: 'Должен показывать ошибку, если пароль невалидный',
                password: '~Inv@lid~'
            },
            'setHighlight': {
                test: 'Должен подсвечивать инпут с паролем, если пароль невалидный',
                password: '~Inv@lid~'
            },
            'hideError': {
                test: 'Должен убирать ошибку, если пароль валидный',
                password: 'Valid'
            },
            'unsetHighlight': {
                test: 'Должен убирать подсветку с поля с паролем, если пароль валидный',
                password: 'Valid'
            }
        };

        beforeEach(function() {
            var valStub = this.sinon.stub();
            this.valStub = valStub;

            password.$password = {
                val: valStub,
                selector: '123abcd'
            };

            for (var method in methods) {
                this.sinon.stub(promobar, method);
            }
        });

        for (var method in methods) {
            it(methods[method].test, function() {
                this.valStub.returns(methods[method].password);
                promobar.onPasswordKeyup();
                expect(promobar[method].called).to.be(true);
            });
        }
    });
```
- Замедляет выполнение тестов
- Затрудняет чтение тестов
- Затрудняет изменение тестов
- Может усложнить процесс поиска ошибки

+ Увеличивает количество случаев покрытых тестами

Если вам действительно не важно тестировать большое количество
случаев, то лучше не использовать технику для генерации тестов.
Вместо всех возможных случаев возьмите краевые и случае когда система может вести
себя не предсказуемо.

Good:
```javascript
    describe('onPasswordKeyup', function() {
        beforeEach(function() {
            password.$password = {
                val: this.sinon.stub(),
                selector: '123abcd'
            };

            this.sinon.stub(promobar, 'showError');
            this.sinon.stub(promobar, 'setHighlight');
            this.sinon.stub(promobar, 'hideError');
            this.sinon.stub(promobar, 'unsetHighlight');
        });

        it('Должен показывать ошибку, если пароль невалидный', function() {
            password.$password.val.returns('~Inv@lid~');

            promobar.onPasswordKeyup();

            expect(promobar.showError.called).to.be(true);
        });

        it('Должен подсвечивать инпут с паролем, если пароль невалидный', function() {
            password.$password.val.returns('~Inv@lid~');

            promobar.onPasswordKeyup();

            expect(promobar.setHighlight.called).to.be(true);
        });

        it('Должен убирать ошибку, если пароль валидный', function() {
            password.$password.val.returns('Valid');

            promobar.onPasswordKeyup();

            expect(promobar.hideError.called).to.be(true);
        });

        it('Должен убирать подсветку с поля с паролем, если пароль валидный', function() {
            password.$password.val.returns('Valid');

            promobar.onPasswordKeyup();

            expect(promobar.unsetHighlight.called).to.be(true);
        });
    });
```

```javascript
describe('onPasswordKeyup', function() {
    beforeEach(function() {
        password.$password = {
            val: this.sinon.stub(),
            selector: '123abcd'
        };

        this.sinon.stub(promobar, 'showError');
        this.sinon.stub(promobar, 'setHighlight');
        this.sinon.stub(promobar, 'hideError');
        this.sinon.stub(promobar, 'unsetHighlight');
    });

    describe('Неправильный пароль', function() {
        beforeEach(function() {
            password.$password.val.returns('~Inv@lid~');
        });

        it('Должен показывать ошибку', function() {
            promobar.onPasswordKeyup();
            expect(promobar.showError.called).to.be(true);
        });

        it('Должен подсвечивать инпут с паролем', function() {
            promobar.onPasswordKeyup();
            expect(promobar.setHighlight.called).to.be(true);
        });
    });

    describe('Правильный пароль', function() {
        beforeEach(function() {
            password.$password.val.returns('Valid');
        });

        it('Должен убирать ошибку', function() {
            promobar.onPasswordKeyup();
            expect(promobar.hideError.called).to.be(true);
        });

        it('Должен убирать подсветку с поля с паролем', function() {
            promobar.onPasswordKeyup();
            expect(promobar.unsetHighlight.called).to.be(true);
        });
    });
});
```

+ Тесты проще читаются(как документация)
+ Проще изменять и понять код тестов

- Не покрывает большое количество тестовых случаев
- Если слишком сложная настройка окружения для тестирования функциональности
то может привести к плохо читающимся тестам. В этом случае надо обратить внимание
на структуру кода, который тестируете.

## Тестирование времени

Bad source code:
```javascript
date.fullDate = function(timestamp){
    var userDate = new Date();
    var nowDate = new Date();

    ...
    userDate.setTime(timestamp);
    ...
}
```

Bad:
```javascript
it('Возвращает короткую форму записи для даты текущего года', function() {
    var timestamp = '1371813596000';
    expect(this.date.fullDate(timestamp)).to.be('21 июн. в 15:19');
});
```
Привязываемся к времени на компьютере, которое мы не контролируем.
Тест становится не детерминированный.

- Тест может сломаться из за времени на машине, хотя исходных код работает правильно.

Нам необходимо протестировать наш код, а не внешний.
Все системные и внешние объекты необходимо заменять на mock/stub объекты.
Это позволит контролировать объект не зависимо от его реального поведения.

Good source code:
```javascript
date.fullDate = function(timestamp){
    var userDate = new Date();
    var nowDate = Date.getDateNow();

    ...
    userDate.setTime(timestamp);
    ...
}
```

Good:
```javascript
it('Возвращает короткую форму записи для даты текущего года', function() {
    var timestamp = '1371813596000';
    var nowDate = new Date();
    nowDate.setTime(1371813600000);

    this.sinon.stub(Date, 'getDateNow').returns(nowDate);

    expect(this.date.fullDate(timestamp)).to.be('21 июн. в 15:19');
});
```
+ Тест не зависит от времени на машине, где исполняются тесты
+ Детерминированный

## Тестирование неопределенного исключения

Всегда определяйте тип исключения.

Bad:
```javascript
    describe('Валидация телефона', function() {
        it('Должен бросать эксепшен, если нужного поля у телефона нет', function() {
            var phone = new UserPhone(mock['UserPhone.registeredSecurePhone']);

            expect(phone.onResendCodeTimeout.bind(phone, $.noop)).to.throwError();
        });
    });
```
В примере выше код тестирует что функция `onResendCodeTimeout` бросит исключение,
при этом не проверяя тип исключения. Такой тест **может давать не правильный результат**.
Например при данных условиях функция может бросить `TypeError` вместо пользовательского
исключения, но тест при этом будет проходить, хотя это явно не правильное поведение.

Good:
```javascript
    describe('Валидация телефона', function() {
        it('Должен бросать эксепшен, если нужного поля у телефона нет', function() {
            var phone = new UserPhone(mock['UserPhone.registeredSecurePhone']);

            expect(phone.onResendCodeTimeout.bind(phone, $.noop)).to.throwError(/No field resendCodeTimeout/);
        });
    });
```

Тут вы жестко определили какое исключение вы ожидаете.

## Асинхронные тесты

Bad:
```javascript
    it('Должен отщищать все обработчики', function(done) {
        var phone = new UserPhone(mock['UserPhone.validSecurePhone']);

        phone.remove().done(function() {
            expect(jQuery._data(phone, "events")).to.be(undefined);

            done();
        });

        setTimeout(function() {
            phone._deferred.resolve();
        }, 0);
    });
```
- Если в асинхронном тесте есть ошибка, то все остальные тесты будут ждать пока
он не упадет по timeout-у.
- Асинхронный тест читается и понимается хуже чем синхронных
- Зависимость от реализации асинхронности внутри теста (setTimeout)
- Сложнее изменять, добавлять новые проверки
- Сложнее читать. Фаза проверки находиться не в конце теста, а внутри обработчика.

Good:
```javascript
    it('Должен отщищать все обработчики', function() {
        var phone = new UserPhone(mock['UserPhone.validSecurePhone']);

        phone.remove();
        phone._deferred.resolve();

        expect(jQuery._data(this.phone, "events")).to.be(undefined);
    });
```
+ Проще читать
+ Проще проверять
+ Проще изменять
+ Так как код выполняется последовательно, тест падает сразу
+ Приводит к тому, что реализация обработчиков не полагается на асинхронное
выполнение(могут выполняться как синхронно так и асинхронно)
