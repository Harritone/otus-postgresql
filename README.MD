# Домашнее задание по теме SQL и реляционные СУБД. Введение в Postgresql

## Инструкция по запуску PostgreSQL в докер-контейнере:

 1. Клонировать репозиторий
 2. Создать файл `.env` в корне проекта.
 3. Добавить переменные среды:
  - `pg_database` - Имя бд для подключения по умолчанию.
  - `pg_host` - Хост бд (posgres)
  - `pg_port` - Порт бд
  - `pg_user` - Пользователь бд
  - `pg_password` - пароль пользователя бд
4. Для запуска выполнить `docker-compose up -d`
5. Чтобы запустить клиента выполнить `docker-compose run --rm pg_client`


## Ход выполнения домашнего задания:

1. выключить auto commit
  ```sql
    >\SET AUTOCOMMIT OFF
    >\ECHO :AUTOCOMMIT
  ```

2. сделать в первой сессии новую таблицу и наполнить ее данными:
  ```sql
  CREATE TABLE persons(id SERIAL, first_name TEXT, second_name TEXT);
  INSERT INTO persons(first_name, second_name) VALUES('ivan', 'ivanov');
  INSERT INTO persons(first_name, second_name) VALUES('petr', 'petrov');
  COMMIT;
  ```
3. посмотреть текущий уровень изоляции: `show transaction isolation level`:
  ```sql
    otusdb=# show transaction isolation level;
    transaction_isolation
    -----------------------
    read committed
    (1 row)
  ```
4. начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
в первой сессии добавить новую запись
  ```sql
    INSERT INTO persons(first_name, second_name) VALUES('sergey', 'sergeev');
    INSERT 0 1
  ```
5. сделать `select * from persons` во второй сессии
  ```sql
    SELECT * FROM persons;
    id | first_name | second_name
    ----+------------+-------------
      6 | ivan       | ivanov
      7 | petr       | petrov
    (2 rows)
  ```
6. видите ли вы новую запись и если да то почему?
  > Новой записи нет, потому что автокоммит выключен,
  > при уровне изоляции read committed незакомиченные изменения не видны в других сессиях,
  > и для того чтобы увидеть новую запись во второй сессии нужно завершить транзакцию в первой сессии.
7. завершить первую транзакцию - `commit;` сделать select * from persons во второй сессии видите ли вы новую запись и если да то почему?

  >1 сессия:
  ```sql
    COMMIT;
    COMMIT
  ```

  > 2 сессия:
  ```sql
    SELECT * FROM persons;
     id | first_name | second_name
    ----+------------+-------------
      6 | ivan       | ivanov
      7 | petr       | petrov
      8 | sergey     | sergeev
    (3 rows)
    END;
    COMMIT
  ```
  > Да, после завершения транзакции изменения стали доступными и видны для всех сессиий.

8. начать новые но уже repeatable read транзации - `set transaction isolation level repeatable read;` в первой сессии добавить новую запись `insert into persons(first_name, second_name) values('sveta', 'svetova');` сделать `select * from persons` во второй сессии видите ли вы новую запись и если да то почему?

  ```sql
    set transaction isolation level repeatable read;
    SET
  ```
  > Нет. В repeatable read уровне так же не закомиченные изменения не будут видны в других сессиях.

  > 1 сессия:
  ```sql
    INSERT INTO persons(id, first_name, second_name) VALUES(7, 'oksy', 'petrova');
    INSERT 0 1
  ```

  > 2 сессия:
  ```sql
    SELECT * FROM persons;
      id | first_name | second_name
    ----+------------+-------------
      6 | ivan       | ivanov
      7 | petr       | petrov
      8 | sergey     | sergeev
    (3 rows)
  ```

9. завершить первую транзакцию - `commit;` сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?

  > 1 сессия:
  ```sql
    COMMIT;
    COMMIT
  ```

  > 2 сессия:
  ```sql
    SELECT * FROM persons;
     id | first_name | second_name
    ----+------------+-------------
      6 | ivan       | ivanov
      7 | petr       | petrov
      8 | sergey     | sergeev
    (3 rows)
  ```
  > Нет. В repeatable read уровне при первом запроса зафиксированные строки на выборку или изменения
  > не будут меняться в рамках текущей транзакции, поэтому результат выборки будет одним и тем же в рамках этой транзакции.

10. завершить вторую транзакцию, сделать `select * from persons` во второй сессии, видите ли вы новую запись и если да то почему?

  > 2 сессия:
  ```sql
    END;
    COMMIT
    SELECT * FROM persons;
     id | first_name | second_name
    ----+------------+-------------
      6 | ivan       | ivanov
      7 | petr       | petrov
      8 | sergey     | sergeev
      7 | oksy       | petrova
    (4 rows)
  ```

  > Да. Перед выборкой в новой транзакции изменения уже были закомичены,
  > соответственно они выбрались и зафиксировались в рамках текущей транзакции.