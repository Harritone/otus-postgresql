# Домашнее задание по теме Физический уровень PostgteSQL

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
4. Проверил что контейнер не развёрнут `docker ps`
5. Изменил `docker-compose.yml`, переименовал подмонтированную папку из `pg_data` в `pg_data_1`
6. Выполнил `docker-compose up -d`
7. Запустил клиента `docker-compose run --rm pg_client`
8. Удовстоверился что таблицы `persons` нет.
  ```psql
    otusdb=# \d
    Did not find any relations.
  ```
9. Остановил контейнер.
10. Перенёс содержимое `pg_data` в `pg_data_1`
11. Развернул контейнер.
12. Запустил клиент.
13. Удостоверился что данные на месте:
  ```psql
    otusdb=# \dt
              List of relations
    Schema |  Name   | Type  |  Owner
    --------+---------+-------+----------
    public | persons | table | postgres
    (1 row)
  ```