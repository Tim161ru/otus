1)создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере
 -- новый проект Яндекс облако.png

2) далее создать инстанс виртуальной машины с дефолтными параметрами
 -- инстанс виртуальной машины.png
 
3) добавить свой ssh ключ в metadata ВМ
 -- добавить свой ssh ключ в metadata ВМ.png
 
4) зайти удаленным ssh (первая сессия), не забывайте про ssh-add
 -- tkhandadashev@test:~$ --я на машине

5) поставить PostgreSQL

tkhandadashev@test:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-11
~~~~
~~~~
postgres@test:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
11  main    5432 online postgres /var/lib/postgresql/11/main /var/log/postgresql/postgresql-11-main.log

6) зайти вторым ssh (вторая сессия)

7)запустить везде psql из под пользователя postgres
  выключить auto commit
  postgres=# \echo :AUTOCOMMIT
  OFF
  
8) сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
--INSERT 0 1
--INSERT 0 1
--COMMIT
9) посмотреть текущий уровень изоляции: show transaction isolation level
 postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)

10) начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
       в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
		postgres=# insert into persons(first_name, second_name) values('sergey', 'sergeev');
		INSERT 0 1

	сделать select * from persons во второй сессии
		postgres=# select * from persons
		все так же 2 строки
		
11)видите ли вы новую запись и если да то почему? -- нет, изоляция read committed
12)завершить первую транзакцию - commit; -- postgres=# commit;
13)сделать select * from persons во второй сессии -- 3 строки показано.
14)видите ли вы новую запись и если да то почему? -- вижу т.к транзакция в перовй сессии закончелась команой  commit;
15)завершите транзакцию во второй сессии -- done

16)начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
--postgres=# set transaction isolation level repeatable read;
--SET

в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
--postgres=# insert into persons(first_name, second_name) values('sveta', 'svetova');
--INSERT 0 1

 сделать select * from persons во второй сессии
 видите ли вы новую запись и если да то почему? -- нет не вижу, не было коммита.
 завершить первую транзакцию - commit; -- done
 сделать select * from persons во второй сессии -- вижу 4 строки
 видите ли вы новую запись и если да то почему? -- прошел коммит, все ограничения сняты.
