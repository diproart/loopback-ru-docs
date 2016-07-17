# Managing users

original: https://docs.strongloop.com/display/public/LB/Registering+users

## Регистрация пользователей

Модель `User` предоставляет методы для регистрации новых пользователей и подтверждения их email-адресов. Вы также можете использовать модуль `loopback-component-passport` для интеграции авторизации с Facebook, Google и другими провайдерами.

### Регистрация пользователей с использованием `User`

#### Создание нового пользователя

Создание пользователя (или регистрация пользователя), это создание нового экземпляра модели User, ничем не отличается от создания экземпляров любых других моделей. Обязательными являются только свойства `email` и `password`.

Создание пользователя на этапе инициализации [используя скрипт](https://docs.strongloop.com/display/LB/Defining+boot+scripts) (`/boot/server/my-boot-script.js`):

```javascript
module.exports = function(app){
  var User = app.models.User;
  User.create({
    email: 'foo@bar.com',
    password: 'baz'
  }, function(err, userInstance){
    // if(err) throw err;
    console.log('created', userInstance);
  });
};
```

> пароль автоматически хешируется при сохранени в базу данных.

Создание пользователя, используя `REST`, отправляем `POST` запрос

```bash
curl -X POST -H "Content-Type:application/json" \
-d '{"email": "me@domain.com", "password": "secret"}' \
http://localhost:3000/api/users
```

#### Добавление специальных ограничений при регистрации пользователя

Обычно требуется добавить различные методы в процесс регистрации пользователя, для примера, проверять уникально ли имя пользователя или адрес электронной почты. Лучший путь для реализации подобных функций - это добавить методы как `beforeRemote` хуки к объекту `User`. [Более подробно о хуках](https://docs.strongloop.com/display/LB/Remote+hooks).

#### Проверка адреса электронной почты

Обычно в приложении требуется, чтобы пользователи подтверждали адрес свой электронной почты, перед тем как получить возможность авторизации. Приложение должно отправлять пользователю письмо содержащее ссылку для подтверждения указанного при регистрации адреса электронной почты. После перехода по ссылке (подтверждения email) пользователь перенаправляется на главную страницу ("/") и теперь сможет авторизоваться в приложении.

Для включения этого ограничения установите свойство модели `emailVerificationRequired` в значение `true` в файле `server/model-config.json`, например для модели `customer`:

```json
"customer": {
  "dataSource": "db",
  "public": true,
  "options": {
    "emailVerificationRequired": true
  }
}
```

Для подтверждения через `REST` будет использоваться `GET` запрос на `/customers/confirm`.

Следующий пример демонстрирует создание `remote hook` для модели `Customer`, который выполняется после метода`create()`.

> Для работы данного примера модель `Customer` и источник данных (datasource) `Mail` должны быть установлены и настроены.


```javascript
// /common/models/customer.js
var config = require('../../server/config.json');
var path = require('path');

module.exports = function(customer){
 // отправка письма
 customer.afterRemote('create', function(context, userInstance, next){
   var options = {
    type: 'email',
    to: userInstance.email,
    from: 'noreply@my-app.com',
    subject: 'Please confirm email address',
    template: path.resolve(__dirname, '../../server/views/verify.ejs'),
    redirect: '/verified',
    user: customer
   };
   
   userInstance.verify(options, function(err, response, next){
     if(err) return next(err);
     
     context.res.render('response', {
       title: 'Signed up successfully',
       content: 'Please check your email and click on the verification link ',
       redirectTo: '/',
       redirectToLinkText: 'Log in'
     });
   });   
 });
}
```

Полный пример можно посмотреть здесь [user.js](https://github.com/strongloop/loopback-faq-user-management/blob/master/common/models/user.js), [loopback-example-user-management](https://github.com/strongloop/loopback-example-user-management)

> Если ваша модель именуются по соглашении camelСase, например `MyUser`, то имена файлы будут: `common/models/my-user.{js,json}`

Шаблон `verify.ejs`

```html
This is the html version of your email.
<strong><%= text %></strong>
```

#### Регистрация через других провайдеров

Для организации регистрации через других провайдеров используется `loopback-component-passport`. Подробнее - [Third-party login using Passport](https://docs.strongloop.com/display/public/LB/Third-party+login+using+Passport)
