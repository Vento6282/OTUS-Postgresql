# Хранимые функции и процедуры.

Занятие от 4.07.2024

## Выполнение домашнего задания:

 - Скрипт и развернутое описание задачи – в ЛК (файл hw_triggers.sql) или по ссылке: https://disk.yandex.ru/d/l70AvknAepIJXQ
    В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales).

 - Есть запрос для генерации отчета – сумма продаж по каждому товару.

 - БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.

 - Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину) ***Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE***

Перед созданием триггера и триггерной функции создал процедуру для очистки и заполнения таблицы (витрины) актуальными данными **good_sum_mart**:
```
CREATE OR REPLACE PROCEDURE fill_table_good_sum_mart()
LANGUAGE plpgsql
AS $$
BEGIN
   TRUNCATE TABLE good_sum_mart;
   INSERT INTO good_sum_mart 
   SELECT G.good_name AS good_name, sum(G.good_price * S.sales_qty) AS sum_sale
   FROM goods G
      INNER JOIN sales S ON S.good_id = G.goods_id
   GROUP BY G.good_name;
END;
$$;
```

Создание триггерной функции:
```
CREATE OR REPLACE FUNCTION update_insert_delete_table_sales() 
RETURNS TRIGGER AS 
$$
   DECLARE
      good_sum_mart_row good_sum_mart%ROWTYPE;
   BEGIN
      CASE TG_OP
         WHEN 'INSERT'
            THEN 
               IF (SELECT 1 
                  FROM goods g 
                     INNER JOIN good_sum_mart gsm ON g.good_name = gsm.good_name 
                  WHERE g.goods_id = new.good_id) 
               THEN 
                  SELECT good_name, new.sales_qty * good_price
                  INTO good_sum_mart_row
                  FROM goods g
                  WHERE g.goods_id = new.good_id;
                  UPDATE good_sum_mart gsm
                  SET sum_sale = sum_sale + good_sum_mart_row.sum_sale
                  WHERE gsm.good_name = good_sum_mart_row.good_name;
               ELSE 
                  SELECT good_name, new.sales_qty * good_price
                  INTO good_sum_mart_row
                  FROM goods g
                  WHERE g.goods_id = new.good_id;
                  INSERT INTO good_sum_mart (good_name, sum_sale)
                     VALUES (good_sum_mart_row.good_name, good_sum_mart_row.sum_sale);
               END IF;
         WHEN 'DELETE'
            THEN 
               IF (SELECT 1 AS "1"
                  FROM sales s
                  WHERE sales_id != old.sales_id and good_id = old.good_id
                  GROUP BY "1") 
               THEN 
                  SELECT good_name, old.sales_qty * good_price
                  INTO good_sum_mart_row
                  FROM goods g
                  WHERE g.goods_id = old.good_id;
                  UPDATE good_sum_mart gsm
                  SET sum_sale = sum_sale - good_sum_mart_row.sum_sale
                  WHERE gsm.good_name = good_sum_mart_row.good_name;
               THEN
                  DELETE 
                  FROM good_sum_mart
                  WHERE good_name = (
                     SELECT good_name
                     FROM goods
                     WHERE goods_id = old.good_id);
               END IF;								
         WHEN 'UPDATE'
            THEN 
               IF new.good_id != old.good_id
               THEN
                  IF (SELECT 1 
                     FROM goods g 
                        INNER JOIN good_sum_mart gsm ON g.good_name = gsm.good_name 
                     WHERE g.goods_id = new.good_id) 
                  THEN 
                     SELECT good_name, new.sales_qty * good_price
                     INTO good_sum_mart_row
                     FROM goods g
                     WHERE g.goods_id = new.good_id;
                     UPDATE good_sum_mart gsm
                     SET sum_sale = sum_sale + good_sum_mart_row.sum_sale
                     WHERE gsm.good_name = good_sum_mart_row.good_name;
                  ELSE 
                     SELECT good_name, new.sales_qty * good_price
                     INTO good_sum_mart_row
                     FROM goods g
                     WHERE g.goods_id = new.good_id;
                     INSERT INTO good_sum_mart (good_name, sum_sale)
                        VALUES (good_sum_mart_row.good_name, good_sum_mart_row.sum_sale);
                  END IF;
                  IF (SELECT 1 AS "1"
                     FROM sales s
                     WHERE sales_id != old.sales_id and good_id = old.good_id
                     GROUP BY "1") 
                  THEN 
                     SELECT good_name, old.sales_qty * good_price
                     INTO good_sum_mart_row
                     FROM goods g
                     WHERE g.goods_id = old.good_id;
                     UPDATE good_sum_mart gsm
                     SET sum_sale = sum_sale - good_sum_mart_row.sum_sale
                     WHERE gsm.good_name = good_sum_mart_row.good_name;
                  ELSE
                     DELETE 
                     FROM good_sum_mart
                     WHERE good_name = (
                        SELECT good_name
                        FROM goods
                        WHERE goods_id = old.good_id);
                  END IF;	
               ELSEIF (new.good_id = old.good_id and new.sales_qty != old.sales_qty)
               THEN 
                  SELECT good_name, (new.sales_qty - old.sales_qty) * good_price
                  INTO good_sum_mart_row
                  FROM goods g
                  WHERE g.goods_id = new.good_id;
                  UPDATE good_sum_mart gsm
                  SET sum_sale = sum_sale + good_sum_mart_row.sum_sale
                  WHERE gsm.good_name = good_sum_mart_row.good_name;	
               END IF;
      END CASE;
      RETURN NEW;
   END;
$$ LANGUAGE plpgsql;
```
Создание триггера:
```
CREATE OR REPLACE TRIGGER trigger_table_sales
   AFTER INSERT OR UPDATE OR DELETE 
   ON "sales"
   FOR EACH ROW
   EXECUTE PROCEDURE update_insert_delete_table_sales();
```
### Комментарий
Работа триггера предусматривает следующие ситуации:
 1. Если в таблице **sales** в результате удаления или изменения записей не окажется ниодной записи по продажам относящимся к какому-либу товару, то этот товар удаляется из таблицы **good_sum_mart**.
 2. Если в таблице **sales** появляется запись о товаре, который не содержится в таблице **good_sum_mart**, то соответствующий товар добавляется в эту таблицу.
 3. В таблице **sales** учитываются изменения в записях не только количества товара (*sales_qty*), но и если изменяют id товара (*good_id*), допустим, ошиблись при указании товара при продаже.

В задании (ТЗ) не указано, но также можно добавить триггер на таблицу **goods**, что учитывалось изменения цены или названия товара. В случае с ценой, скорее потребуется новая таблица, что-то вроде реестра изменения цен. В случае названия товара, при изменении названия товара потеряется сопоставление с таблицей **good_sum_mart**.

Задание со звездочкой*
 - Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)? ***Подсказка: В реальной жизни возможны изменения цен.***

### Ответ
Не уверен, что правильно понимаю значение "по требованию", но предположу, что при обращении к витрине результат будет получаться быстрее, чем строить отчёт. В схеме витрина+триггер будут храниться всегда актуальные данные, но это будет зависеть от того, на сколько продумали реализацию триггеров для всех случаев изменения связанных данных.