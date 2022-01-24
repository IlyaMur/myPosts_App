# MyPosts

![CodeSniffer-PSR-12](https://github.com/IlyaMur/myposts_app/workflows/CodeSniffer-PSR-12/badge.svg)
![PHPUnit-Tests](https://github.com/IlyaMur/myposts_app/workflows/PHPUnit-Tests/badge.svg)
[![Maintainability](https://api.codeclimate.com/v1/badges/0273c0de648a6f356cf3/maintainability)](https://codeclimate.com/github/IlyaMur/myposts_app/maintainability)

**[🇬🇧 English readme](https://github.com/IlyaMur/myposts_app/blob/master/README_en.md)**

**Содержание**
  - [О приложении](#о-приложении)
  - [Установка](#установка)
    - [Использованные библиотеки](#использованные-библиотеки)
  - [Как работает приложение](#как-работает-приложение)
    - [Системы аутентификации и авторизации](#система-аутентификации-и-авторизации)
    - [Работа с постами](#работа-с-постами)
    - [Работа с комментариями](#работа-с-комментариями)
    - [Хэштеги](#хэштеги)
    - [Хранение изображений](#хранение-изображений)
    - [Шаблоны](#шаблоны)
    - [Административная составляющая](#административная-составляющая)
    - [Ошибки](#ошибки)

## О Приложении  

**MyPosts** - приложение-блог созданное на базе фреймворка [PHP On Rails](https://github.com/IlyaMur/php_on_rails_mvc).  
Блог создан в процессе обучения, но обладает богатым функционалом.  

Деплой приложения осуществлен на сервис Heroku.   
MyPosts доступен по адресу - http://myposts-app.herokuapp.com.

В приложении реализованы:
- Полноценные системы аутентификации и авторизации пользователей.
- Активация аккаунта и сброс пароля по почте (рассылка писем осуществлена на базе сервиса MailJet).
- Хранение данных пользователей, как локально, так и на стороннем сервисе (AWS S3).
- Система комментариев.
- Хэштеги.
- Профили пользователей.
- CAPTCHA и базовая защита от спама.
- Административная составляющая.

Большая часть функционала написана с нуля, готовые решение по возможности избегались.  
Основой для представлений был выбран шаблонизатор [Twig](https://twig.symfony.com/), как гибкое и безопасное решение с лаконичным синтаксисом.

![Главная страница](https://i.imgur.com/AUtFld3.png)   

## Установка  

- Версия PHP >= 8.0 (приложение использует именованные аргументы и другие нововведения PHP).
- `$ git clone` данный репозиторий.
- `$ make install` для установки зависимостей.
- В настройках веб-сервера установить root в директории `public/`. 
- В выбранную СУБД импортировать SQL из файла `myposts.sql`. 
- В `config/config.php` заполнить данные для доступа к БД, настройки хранения и хэширования.

### Использованные библиотеки  

Не смотря на то, что в процессе написания приложения в первую очередь стояла цель создания функционала с нуля, в MyPosts присутствуют некоторые зависимости:
- Twig Template Engine.
- SDK Mailjet.
- SDK AWS.
- Gregwar/Captcha

## Как Работает Приложение

### Конфигурация  

Настройки конфигурации доступны в файле [config.php](config/config.php)

Настройки по умолчанию включают в себя:
- Данные подключения к серверу БД. 
- Логирование ошибок.
- Возможность выбора между локальным и удаленным хранением пользовательских изображений.
- Установки для вывода/скрытия детализации ошибок.
- Настройка секретного ключа для хэширования токенов.

Для переопределения настроек в конфигурационном файле доступны соответствующие константы.

### Система аутентификации и авторизации

В MyTasks с нуля реализована система регистрации и аутентификации пользователей.  

Форма аутентификации валидируется как на стороне сервера, так и клиента.  
После регистрации на почту пользователя высылается письмо с ссылкой для активации.  
Создана возможность сброса существущего пароля на почту. У токена сброса имеется срок годности.

Рассылка писем осуществлена посредством сервиса MailJet и служебного класса [Mail](src/Service/Mail.php), шаблоны для работы с паролями и токенами пользователей доступны в [src/Views/Password](src/Views/Password) 

Логин пользователя доступен через [RememberedLogin](src/Models/RememberedLogin.php). Токен логина так же имеет срок давности.  
Для взаимодействия с сессиями и куки используется служебный класс [Auth](src/Service/Auth.php), предоставляющий инструменты для работы с авторизациями пользователей.

### Работа с постами

Основой блога являются посты пользователей. Они удобно выведены на главной странице, так же доступна пагинация.
Пользователи могут добавлять изображения к постам, редактировать свои посты и комментировать чужие и свои посты.  
Так же внимание уделено безопасности и в частности XSS-атакам.  
Валидация содержимого поста происходит как на клиенте (посредством библиотеки JS и HTML5-валидации), так и на сервере в модели [Post](src/Models/Post.php), ошибки возвращаются пользователю.

### Работа с комментариями

Пользователи могут оставлять комментарии под своими и чужими постами. Для работы с комментариями создана модель [Comment](src/Models/Comment.php) и контроллер [Comments](src/Controllers/Comments.php).   
Реализована и анонимная отправка комментариев, но для этого необходимо пройти проверку через ввод CAPTCHA.  
Комментарии пользователя доступны в его профиле, где удобно выведены через пагинацию.
Комментарии так же валидируются и проверяются на XSS.

### Хэштеги

Как и любой современный блог MyPosts поддерживает хэштеги. Хэштеги доступны в постах и автоматически закрепляются за ними.
Логика по работе с хэштегами находится в модели [Hashtag](src/Controllers/Hashtag.php).   
Последние 10 добавленных хэштегов доступны на главной странице. При клике по хэштегу выводятся связанные с ними посты.

### Хранение изображений

Для хранения изображений доступны два варианта.
- Хранение изображений на базе сервиса AWS S3. С помощью AWS SDK.
- Хранение изображений локально.

Для использования AWS S3 необходимо ввести данные своего аккаунта в файле [config.php](config/config.php) и установить в константу `AWS_STORING` в значение `true`.  
Для работы воспользоваться классом [S3Helper](src/Service/S3Helper.php), который использует SDK и предоставляет интерфейс для взаимодействия с облачным хранилищем.

Для локального хранения данных необходимо установить `AWS_STORING` в `false`. Картинки будут сохраняться в директорию `public/upload/`.

Все изображения перед загрузкой на сервер валидируются.

### Шаблоны

Представления организованы посредством шаблонизатора [Twig](https://twig.symfony.com/), который поддерживает наследование шаблонов.  
Шаблоны находятся в директории `src/Views`, базовый шаблон - [base.html.twig](src/Views/base.html.twig).  
Для переиспользования кода и улучшения читаемости в директорию `src/Views/partials` вынесены часто используемые элементы Представлений.

Для приятного и лаконичного внешнего вида был выбран CSS Framework - Bootstrap.

### Административная составляющая

Администратор блога имеет свою панель управления, взаимодействие с которой происходит в отдельном неймспейсе `Ilyamur\PhpMvc\Controllers\Admin`.
Администратор видит подробную информацию о созданных постах и может модерировать их.

### Ошибки

Ошибки преобразуются в исключения. Обработчиками обозначены:
```
set_error_handler('Ilyamur\PhpMvc\Service\ErrorHandler::errorHandler');
set_exception_handler('Ilyamur\PhpMvc\Service\ErrorHandler::exceptionHandler');
```

При константе `SHOW_ERRORS` (настраивается в [config.php](config/config.php)) равной `true`, в случае исключения или ошибки в браузер будет выведена полная детализация.   
Если `SHOW_ERRORS` присвоено значение `false` будет показано лишь общее сообщение из шаблонов [404.html.twig](src/Views/404.html.twig) или [500.html.twig](src/Views/500.html.twig) в зависимости от ошибки.  
Детализированная информация в данном случае будет логироваться в директории `logs/`.
