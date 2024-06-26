# SQL и реляционные СУБД. Введение в PostgreSQL 

Занятие от 4.04.2024

## Выполнение домашнего задания:

 - запустить везде psql из под пользователя postgres
 - выключить auto commit
 - сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
 - посмотреть текущий уровень изоляции: show transaction isolation level
 - начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
 - в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
 - сделать select from persons во второй сессии
 - видите ли вы новую запись и если да то почему?

 ### Комментарий:
Внесённые изменения в первой сессии не зафиксированы. Во второй сессии установлен режим изоляции read commited, которая не позволяет читать не зафиксированные данные, поэтому новая строка во второй сессии не видна.

 - завершить первую транзакцию - commit;
 - сделать select from persons во второй сессии
 - видите ли вы новую запись и если да то почему?

  ### Комментарий:

Во второй сессии вставленная строка видна, так как внесённые изменения в первой сессии зафиксированы.

 - завершите транзакцию во второй сессии
 - начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
 - в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
 - сделать select* from persons во второй сессии*
 - видите ли вы новую запись и если да то почему?

  ### Комментарий:

Внесённые изменения в первой сессии не зафиксированы. Во второй сессии установлен режим изоляции repeatable read, которая не позволяет читать не зафиксированные данные, поэтому новая строка во второй сессии не видна.

 - завершить первую транзакцию - commit;
 - сделать select from persons во второй сессии
 - видите ли вы новую запись и если да то почему?

  ### Комментарий:

Во второй сессии не видно новой строки. Режим изоляции repeatable read удерживает блокировки каждой затронутой строки до окончания транзакции. Такой режим изоляции гарантирует, что в рамках одной транзакции при повторном обращении данные не могут измениться другими транзакциями.

 - завершить вторую транзакцию
 - сделать select * from persons во второй сессии
 - видите ли вы новую запись и если да то почему?

  ### Комментарий:

Вторая сессия завершила транзакцию с режимом изоляции repeatable read и, блокировка с данных снимается. Новые данные видны.