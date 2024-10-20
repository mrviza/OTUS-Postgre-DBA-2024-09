1) Поднимаем докер в контейнере
2) Подключаемся через ```docker exec -it container_id bash```
3) Запускаем транзакцию без ```autocommit``` командой ``` begin; ```
4) Выполняем:
    > create table persons(id serial, first_name text, second_name text);<br>
    insert into persons(first_name, second_name) values('ivan', 'ivanov');<br>
    insert into persons(first_name, second_name) values('petr', 'petrov');<br>
    commit;

   Получаем:
   > CREATE TABLE<br>
   INSERT 0 1<br>
   INSERT 0 1<br>
   COMMIT

   <br>
5) Выполняем:
   ```show transaction isolation level;```<br>
   Получаем:
   >transaction_isolation<br>
   -----------------------<br>
   read committed<br>
   (1 row)

    <br>
6) Запускаем вторую сессию
7) В обоих сессиях запускаем транзакцию без ```autocommit``` командой ```begin;```
8) В первой сессии выполняем:
   >insert into persons(first_name, second_name) values('sergey', 'sergeev');

   Получаем:
   >INSERT 0 1<br>
9) Во второй сессии выполняем:
   >select * from persons;

    Получаем:
    >id | first_name | second_name<br>
    --+------------+-------------<br>
    1 | ivan       | ivanov<br>
    2 | petr       | petrov<br>
    (2 rows)
10) Новая запись не появилась, потому что установлен уровень изоляции ```read_commited```, 
который не позволяет выполнять ```грязное чтение```,
т.е мы не видим записи, которые не были закомичены
11) В первой сессии выполняем ```commit;``` 
12) Во второй сессии выполняем запрос 
    >select * from persons;

    Получаем: 

    >id | first_name | second_name<br>
     --+------------+-------------<br>
     1 | ivan       | ivanov<br>
     2 | petr       | petrov<br>
     3 | sergey     | sergeev<br>
     (3 rows)
13) Новая запись появила ввиду того, что транзакция в первой сессии была закомичена и данные стали доступны для чтения
14) В обоих сессиях запускаем новые транзакции коммандой ```begin;```.
15) В обоих сессиях выполняем:
    > set transaction isolation level repeatable read;
16) В первой сессии выполняем:  
    > insert into persons(first_name, second_name) values('sveta', 'svetova');
17) Во второй сессии выполняем: 
    >select * from persons;

    Получаем:

    >id | first_name | second_name<br>
    --+------------+-------------<br>
    1 | ivan       | ivanov<br>
    2 | petr       | petrov<br>
    3 | sergey     | sergeev<br>
    (3 rows)
18) Новая запись не видна, потому что ```repeatable_read``` позволяет видить только те данные, что были доступны на момент начала транзакции
19) В первой сессии выполняем : ```commit;```
20) Во второй сессии выполняем: 
    > select * from persons;

    Получаем:

    > id | first_name | second_name<br>
    --+------------+-------------<br>
    1 | ivan       | ivanov<br>
    2 | petr       | petrov<br>
    3 | sergey     | sergeev<br>
    (3 rows)
21) Новая запись не видна, потому что ```repeatable_read``` позволяет видить только те данные, что были доступны на момент начала транзакции
22) Во сторой сессии завершаем транзакцию
23) Во второй сессии выполняем: 
    > select * from persons;

    Получаем:

    >id | first_name | second_name<br>
    --+------------+-------------<br>
    1 | ivan       | ivanov<br>
    2 | petr       | petrov<br>
    3 | sergey     | sergeev<br>
    4 | sveta      | svetova<br>
    (4 rows)
24) Новая запись видна, потому что запрос был выполнен после завершения транзакции в первой сессии.