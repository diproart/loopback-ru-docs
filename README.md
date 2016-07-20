# Работа с LoopBack объектами 

## Описание

Главные объекты LoopBack

* App
* Models
* Data sources

Посмотрим как правильно получить ссылки на эти объекты из разных частей приложения ( из загрузочных скриптов, файлов моделей `common/models/*.js`.

## Получение объекта приложения

Получение сссылки на объект приложения имеет ключевое, решающее значение, так как от него вы может получить ссылки на другие объекты, такие как модели и источники данных. Обычно ссылка на объект приложения неоходима в следующих случаях:

* в файлах моделей `common/models/*.js`
* в загрузочных скриптах `server/boot/*`,
* при использовании `middleware` (`sersver/boot/*`, `server/server.js`
* в ваших кастомных скриптах

`app` предоставляет контекст в многих частях `loopback` приложения.

### В скриптах загрузки

Асинхронная версия `/server/boot/your-script.js`

```javascript
// LoopBack передает ссылку в параметрах
module.exports = function(app, cb){

}
```
Синхронная версия `/server/boot/your-script.js`
```javascript
// LoopBack передает ссылку в параметрах
module.exports = function(app, cb){

}
```

Как вы видите в примерах LoopBack автоматически предоставлят объект `app` в виде 
параметра.

See [Defining boot scripts](https://docs.strongloop.com/display/LB/Defining+boot+scripts) for more information about boot scripts.

### В middleware

LoopBack автоматически устанваивает ссылку на объект `app` в параметре `request` (реально, за сценой это делает `express` на основе которого и построен `loopback`).

Доступ в файле `server/server.js`
```javascript
app.use(function(req, res, next) {
  var app = req.app;
  // do stuff
});
```

See [Defining middleware](https://docs.strongloop.com/display/LB/Defining+middleware) for more information on middleware.

### В пользовательском скрипте

Для доступа нужно получить ссылку на объект явно, используя `require`, так же, 
как и для любых других модулей Node.js

Например: `/server/your-script.js`
```javascript
var app = require('/server/server');
```

### В скрипте модели

