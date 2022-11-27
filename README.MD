# Домашнее задание по теме SQL и реляционные СУБД. Введение в Postgresql

## Инструкция по запуску PostgreSQL в докер-контейнере:

 1. Клонировать репозиторий
 2. Создать файл `.env` в корне проекта.
 3. Добавить переменные среды в .env :
  - `pg_database` - Имя бд для подключения по умолчанию.
  - `pg_host` - Хост бд (posgres)
  - `pg_port` - Порт бд
  - `pg_user` - Пользователь бд
  - `pg_password` - пароль пользователя бд
4. Для запуска выполнить `docker-compose up -d`
5. Чтобы запустить клиента выполнить `docker-compose run --rm pg_client`


## Ход выполнения домашнего задания:

1. Выполнил инструкции описанные выше.
2. Создал таблицу `persons`. Заполнил её данными.
3. Выполнил `docker-compose down && docker container prune && docker image prune`
4. Проверил что контейнер не равёрнут `docker ps`
5. Выполнил `docker-compose up -d`
6. Выполнил `docker-compose run --rm pg_client`
7. Выполнил:
  ```sql
    SELECT * FROM persons;
   id | first_name | second_name
  ----+------------+-------------
    6 | ivan       | ivanov
    7 | petr       | petrov
    8 | sergey     | sergeev
    7 | oksy       | petrova
  (4 rows)
  ```
  > Данные остались, потому что подмонтирован volume на диске где хранятся данные.