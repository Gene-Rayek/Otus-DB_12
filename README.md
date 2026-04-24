## Транзакции, MVCC, ACID 


## **Задание:**
Транзакции

**Цель:**  
Заполнить свой проект данными.

**Описание/Пошаговая инструкция выполнения домашнего задания:**  
Описать пример транзакции из своего проекта с изменением данных в нескольких таблицах. Реализовать в виде хранимой процедуры.  
Загрузить данные из приложенных в материалах CSV.  
Реализовать загрузку следующими путями:

- `LOAD DATA`

**Задание со * :**  
Дополнительно выполнить загрузку через:

- `mysqlimport`


## **Выполнено:**

#### 1. Описать пример транзакции из своего проекта с изменением данных в нескольких таблицах. Реализовать в виде хранимой процедуры.

В проекте реализован пример транзакции оформления покупки.  
В рамках одной транзакции:

- выполняется поиск покупателя по email;
- если покупатель отсутствует, он добавляется в таблицу `customers`;
- затем создается запись о покупке в таблицу `purchases`.

Если возникает ошибка, выполняется `ROLLBACK`.  
Если все прошло успешно, выполняется `COMMIT`.

Создание хранимой процедуры:

```
USE otus;

DROP PROCEDURE IF EXISTS sp_create_purchase;
DELIMITER $$

CREATE PROCEDURE sp_create_purchase(
    IN p_customer_name VARCHAR(255),
    IN p_customer_email VARCHAR(100),
    IN p_customer_phone VARCHAR(20),
    IN p_customer_city VARCHAR(100),
    IN p_product_id INT UNSIGNED,
    IN p_price_id INT UNSIGNED,
    IN p_quantity SMALLINT UNSIGNED,
    IN p_purchase_date DATETIME
)
BEGIN
    DECLARE v_customer_id INT UNSIGNED;

    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
    END;

    START TRANSACTION;

    SELECT customer_id
      INTO v_customer_id
      FROM customers
     WHERE email = p_customer_email
     LIMIT 1;

    IF v_customer_id IS NULL THEN
        INSERT INTO customers(name, email, phone, city)
        VALUES (p_customer_name, p_customer_email, p_customer_phone, p_customer_city);

        SET v_customer_id = LAST_INSERT_ID();
    END IF;

    INSERT INTO purchases(customer_id, product_id, price_id, quantity, purchase_date)
    VALUES (v_customer_id, p_product_id, p_price_id, p_quantity, p_purchase_date);

    COMMIT;
END$$

DELIMITER ;
```

 Выполнение хранимой процедуры:
```
CALL sp_create_purchase(
    'Алексей Смирнов',
    'alexey@example.com',
    '+79990000007',
    'Москва',
    1,
    1,
    1,
    '2026-04-25 12:00:00'
);
```

 Проверка результата:
```
SELECT * FROM purchases;
```

<img width="902" height="232" alt="image" src="https://github.com/user-attachments/assets/8f0c1e42-5f94-4933-a146-3f1f95ddc524" />


 
##  2. Загрузить данные из приложенных в материалах CSV. Реализовать следующим путем: LOAD DATA

В рамках задания был использован приложенный CSV-файл с данными пользователей.
Структура файла:

- email
- city

 Пример строк из файла:
```
admin@example.com,Москва
user@example.com,Волгоград
guest@example.com,\N
```
 Так как файл содержит только email и city, для загрузки была создана промежуточная таблица.

 Сначала добавим поле city в таблицу customers:
```
USE otus;

ALTER TABLE customers
ADD COLUMN city VARCHAR(100) NULL;
```

 Создадим промежуточную таблицу для импорта:
```
DROP TABLE IF EXISTS customers_import;

CREATE TABLE customers_import (
    email VARCHAR(100),
    city VARCHAR(100) NULL
);
```

 CSV-файл был предварительно скопирован в контейнер MySQL в путь:
```
/tmp/users-39289-1025cc.csv
```

 Загрузка данных из CSV через LOAD DATA LOCAL INFILE:
```
LOAD DATA LOCAL INFILE '/tmp/users-39289-1025cc.csv'
INTO TABLE customers_import
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
(email, city);
```

## Проверим загруженные данные:
```
TRUNCATE TABLE customers_import;
```

После этого переносим данные в основную таблицу customers:
```
LOAD DATA LOCAL INFILE '/tmp/users-39289-1025cc.csv'
INTO TABLE customers_import
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
(email, city);
```
```
SELECT * FROM customers_import;
```
<img width="483" height="197" alt="image" src="https://github.com/user-attachments/assets/2ddc679b-768d-4ff0-a8b2-a1af9472cac1" />


Проверка результата:
```
SELECT * FROM customers;
```
<img width="1056" height="262" alt="image" src="https://github.com/user-attachments/assets/99920479-6a1f-4119-9f30-872cbf918e6f" />


## 3. Задание со * : загрузить используя mysqlimport

Для выполнения дополнительного задания тот же CSV-файл был подготовлен для импорта через `mysqlimport`.

Так как `mysqlimport` ожидает имя файла, совпадающее с именем таблицы, файл был переименован:

```bash
cp /home/user/users-39289-1025cc.csv /home/user/customers_import.csv
```


После этого импорт выполняется командой:
```
mysqlimport \
  --local \
  --default-character-set=utf8mb4 \
  -u root -p12345 \
  --host=127.0.0.1 \
  --port=3309 \
  otus /home/user/customers_import.csv
```

результата:

<img width="884" height="33" alt="image" src="https://github.com/user-attachments/assets/38f46faa-f0ae-4d9f-bde1-faf51bae2d83" />


В рамках домашнего задания выполнено:

- реализован пример транзакции из проекта с изменением данных в нескольких таблицах;
- создана хранимая процедура sp_create_purchase;
- загружены данные из приложенного CSV с помощью LOAD DATA;
- импортированные данные перенесены в таблицу customers;
- дополнительно подготовлена загрузка того же CSV через mysqlimport.






