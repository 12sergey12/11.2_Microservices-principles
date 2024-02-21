### Домашнее задание к занятию «Микросервисы: принципы» Баранов Сергей

Вы работаете в крупной компании, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps-специалисту необходимо выдвинуть предложение по организации инфраструктуры для разработки и эксплуатации.

### Задача 1: API Gateway 

Предложите решение для обеспечения реализации API Gateway. Составьте сравнительную таблицу возможностей различных программных решений. На основе таблицы сделайте выбор решения.

Решение должно соответствовать следующим требованиям:
- маршрутизация запросов к нужному сервису на основе конфигурации,
- возможность проверки аутентификационной информации в запросах,
- обеспечение терминации HTTPS.

Обоснуйте свой выбор.


На рынке сейчас присутствует большой выбор различных программных решений для организации API Gateway, сравнительные характеристики некоторых (не всех) я вывел в следующую таблицу:


| Возможности | Ingress от Kubernetes | NGINX  | Kong | Traefik   | HAProxy | Contour | 
|-|---------|--------------------|------|-----------|---------|---------|
| маршрутизация запросов к нужному сервису на основе конфигурации | + | + | + | + | + | - |
| возможность JWT-валидации | - | + в платной версии | +| - | + | -|
| возможность проверки аутентификационной информации в запросах | + | Только Basic | + | + | +| - |
| обеспечение терминации HTTPS | + | + | +    | + | + | + |
| Количество звёзд на GitHub  | 4338 | 1469 | 434 | 21554 | 396 | 1523 |


Видно, что возможности у всех решений практически одинаковые. Ввиду популярности и наиболее активного развития я бы использовал продукт Traefik.

----

### Задача 2: Брокер сообщений

Составьте таблицу возможностей различных брокеров сообщений. На основе таблицы сделайте обоснованный выбор решения.

Решение должно соответствовать следующим требованиям:
- поддержка кластеризации для обеспечения надёжности,
- хранение сообщений на диске в процессе доставки,
- высокая скорость работы,
- поддержка различных форматов сообщений,
- разделение прав доступа к различным потокам сообщений,
- простота эксплуатации.

Обоснуйте свой выбор.


| Параметр | RabbitMQ | Apache Kafka | ActiveMQ | NATS | Redis |
|:---------|:--------:|:------------:|:--------:|:----:|:-----:|
| Поддержка кластеризации для обеспечения надежности | + | + | + | + | + |
| Хранение сообщений на диске в процессе доставки | + | + | + | + | + |
| Высокая скорость работы | + - | + | - | + | + |
| Поддержка различных форматов сообщений | AMQP, MQTT, STOMP | Только binary через TCP Socket | 10 форматов, включая AMQP, AUTO, MQTT, REST | Только NATS Streaming Protocol | Только RESP |
| Разделение прав доступа к различным потокам сообщений | + | + | + | + | + |
| Простота эксплуатации | + | - | + | + | + |


Из поставленной задачи в начале не очень понятен характер нагрузки:

- Видно что компания большая, но сопоставимо ли это с объёмом сообщений, которые будут литься на брокера?
- Хватит ли RabbitMQ, потому что он популярный и его просто настроить, или лучше озадачиться чем-то помощней, вроде Apache Kafka?
- Действительно важна "всеядность" или это требование включили "про запас" с запасом на будущее?
- Очередь сообщений нужна только для связи микросервисов, или планируется подключать какие-то внешние системы и монолитные системы?

Как универсальное решение, возможно использовать RabbitMQ - он усреднённый по всем параметрам, очень распространён, проще найти специалистов, которые с ним работали.

---

## Задача 3: API Gateway * (необязательная)

### Есть три сервиса:

**minio**
- хранит загруженные файлы в бакете images,
- S3 протокол,

**uploader**
- принимает файл, если картинка сжимает и загружает его в minio,
- POST /v1/upload,

**security**
- регистрация пользователя POST /v1/user,
- получение информации о пользователе GET /v1/user,
- логин пользователя POST /v1/token,
- проверка токена GET /v1/token/validation.

### Необходимо воспользоваться любым балансировщиком и сделать API Gateway:

**POST /v1/register**
1. Анонимный доступ.
2. Запрос направляется в сервис security POST /v1/user.

**POST /v1/token**
1. Анонимный доступ.
2. Запрос направляется в сервис security POST /v1/token.

**GET /v1/user**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис security GET /v1/user.

**POST /v1/upload**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис uploader POST /v1/upload.

**GET /v1/user/{image}**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис minio GET /images/{image}.

### Ожидаемый результат

Результатом выполнения задачи должен быть docker compose файл, запустив который можно локально выполнить следующие команды с успешным результатом.
Предполагается, что для реализации API Gateway будет написан конфиг для NGinx или другого балансировщика нагрузки, который будет запущен как сервис через docker-compose и будет обеспечивать балансировку и проверку аутентификации входящих запросов.
Авторизация
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token

**Загрузка файла**

curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @yourfilename.jpg http://localhost/upload

**Получение файла**
curl -X GET http://localhost/images/4e6df220-295e-4231-82bc-45e4b1484430.jpg

---

#### [Дополнительные материалы: как запускать, как тестировать, как проверить](https://github.com/netology-code/devkub-homeworks/tree/main/11-microservices-02-principles)

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
