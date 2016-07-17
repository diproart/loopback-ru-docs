# Managing users

original: https://docs.strongloop.com/display/public/LB/Managing+users

## Overview

Встроенная в `LoopBack` модель `User` предоставляет основную функциональность для управления пользователями приложения, это:

+ Регистрация и подтверждение электронной почты
+ Авторизация (login, logout)
+ Создание токенов доступа
+ Сброс (восстановление) пароля

> Важно! При разработке приложения вы должны создать свою собственную модель (с именем отличным от `User`, например `Customer` или `Client`), которая расширит встроенную модель `User`, а не использовать `User` напрямую. Модель `User` обеспечивает большую часть часто используемых функций.

### Создание пользователей и аутентификация

Основной процесс создания и аутентификации такой:

+ Регистрация нового пользователя используя метод `User.create()`, который унаследован от общей объекта `PersistedModel`. [Подробнее](https://docs.strongloop.com/display/LB/Registering+users)
+ Авторизация пользователя, через вызов метода `User.login()` для получения токена доступа
+ Выполнение последующих запросов к API с использованием токена доступа. Токен передается через HTTP-заголовки или как параметр запроса. [Подробнее](https://docs.strongloop.com/display/LB/Making+authenticated+requests#Makingauthenticatedrequests-Makingauthenticatedrequestswithaccesstokens) 

> Для повышения производительности функций регистрации и авторизации рекомендуется использовать "родной" `bcrypt` модуль. Установите его, как глобальный модуль для вашей OS.
> ```
> $ npm install -g bcrypt
> ```

## О встроенной модели `User`

По-умолчанию LoopBack приложение имеет встроенную модель `User` определенную в файле `user.json`(Это файл является частью фреймворка. Не изменяйте его, а следуйте [правилам расширения встроенных моделей](https://docs.strongloop.com/display/LB/Extending+built-in+models))

#### Контроль доступа по-умолчанию

Встроенная модель `User` имеет следующий набор настроек доступа (ACL)

```json

{
 "name": "User",
 "properties": {
   "acls": [
     {
       "principalType": "ROLE",
       "principalId": "$everyone",
       "permission": "DENY"
     },
      {
       "principalType": "ROLE",
       "principalId": "$everyone",
       "permission": "ALLOW",
       "property": "create"
     },
     {
       "principalType": "ROLE",
       "principalId": "$owner",
       "permission": "ALLOW",
       "property": "deleteById"
     },
     {
       "principalType": "ROLE",
       "principalId": "$everyone",
       "permission": "ALLOW",
       "property": "login"
     },
     {
       "principalType": "ROLE",
       "principalId": "$everyone",
       "permission": "ALLOW",
       "property": "logout"
     },
     {
       "principalType": "ROLE",
       "principalId": "$owner",
       "permission": "ALLOW",
       "property": "findById"
     },
     {
       "principalType": "ROLE",
       "principalId": "$owner",
       "permission": "ALLOW",
       "property": "updateAttributes"
     },
     {
       "principalType": "ROLE",
       "principalId": "$everyone",
       "permission": "ALLOW",
       "property": "confirm"
     },
     {
       "principalType": "ROLE",
       "principalId": "$everyone",
       "permission": "ALLOW",
       "property": "resetPassword",
       "accessType": "EXECUTE"
     }
   ]
 }
}
```

Представленый список доступа (ACL) запрещает все операции для всех, а затем селективно разрешает некоторые:

+ Кто угодно может создать нового пользователя
+ Кто угодно может авторизоваться, "выйти", "подтвердить" свою запись и "сбросить" пароль своей учетной записи

> Вы не можете напрямую изменять списки доступа встроенных моделей используя ACL генератор `slc loopback:acl`.
> Однако, вы можете и должны создавать собсвенные моделит расширяющие встроенные и затем используя [ACL-генератор](https://docs.strongloop.com/display/LB/ACL+generator) определять необходимый уровень доступа. Например можно создать модели Customer или Client расширяющие модель User, затем установить необходимые уровни доступа используя `slc loopback:acl`. Помните, что модель не наследует списки доступа (ACL) из своей базовой модели, вы должны установить все необходимые списки доступа для своей новой модели.