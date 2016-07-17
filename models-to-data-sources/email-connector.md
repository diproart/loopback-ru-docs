# Email connector

Email connector - это встроенный компонента, его установка не требуется.

## создание источника данных для email

```bash
$ slc loopback:datasource
```

```bash
$ apic create --type datasource
```

## конфигурация источника данных

файл  `server/datasources.json`

```json
{
  "emailDS": {
    "connector": "mail",
    "transports": [{
      "type": "smtp",
      "host": "smtp.my-smtp.com",
      "secure": false,
      "port": 587,
      "tls": {
        "rejectUnauthorized": false
      },
      "auth": {
        "auth": {
          "user": "mailer@my-smtp.com",
          "pass": "myPassw0rd"
        }
      }
    }]
  }
}
```

Пример использования GMail

```json
{
  "Email": {
    "name": "mail",
    "defaultForType": "mail",
    "connector": "mail",
    "transports": [
      {
        "type": "SMTP",
        "host": "smtp.gmail.com",
        "secure": true,
        "port": 465,
        "auth": {
          "user": "your-name@gmail.com",
          "pass": "your-pass"
        }
      }
    ]
  }
}
```

## Подключение модели к источнику данных

После настройки источника данных нужно настроить модель в файле `server/model-config.json`. Вот пример:

```json
{  
  "Email": {
    "dataSource": "emailDS"
  },
}
```
* [Email model on Github](https://github.com/strongloop/loopback/blob/master/common/models/email.json)

## Отправка почты

Пример демонстрирует как отправлять почту из приложения. Например модель `Mailer` (`common/models/mailer.js`)

```javascript
module.exports = function(Mailer){
  Mailer.sendMail = function(cb){
    Mailer.app.models.Email.send({
      to: 'me@my.com',
      from: 'hello@from.com',
      subject: 'Welcome!',
      text: 'Welcome to Paradise!',
      html: '<h1>Welcome to Paradise!</h1>'
    }, function(err, mail){
      // if(err) throw err;
      cb(err);
    });
  }
};
```

`mail connector` использует [nodemailer](http://www.nodemailer.com/). Смотрите [документацию](http://www.nodemailer.com/) по nodemailer для более подробной информации.

* [верификация адреса электронной почты пользователя при регистрации](managing-users/registering-users.md)

