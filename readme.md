## Задачи и решения SQL

### 1. Подсчёт статистики накопления товаров с использованием пользовательской переменной  

На Sqlfiddle http://www.sqlfiddle.com/#!9/49756e/4/2

```sql
CREATE TABLE IF NOT EXISTS `products` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(150) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
```sql
INSERT INTO `products` (`id`, `name`) VALUES
(1, 'Процессор INTEL Core i5-7500'),
(2, 'Процессор INTEL Core i7-5000'),
(3, 'Оптическая мышь'),
(4, 'Видео карта MV10'),
(5, 'SSD диск Samsung 860 PRO 2 Тб'),
(6, 'SSD диск Samsung 860 PRO 5 Тб'),
(7, 'Материнская плата Supermicro X11DPL-I'),
(8, 'Материнская плата ASUS X99-E-10G WS'),
(9, 'Водяное охлаждение Corsair Hydro Series H115i');
```

```sql
CREATE TABLE IF NOT EXISTS `statistics` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `id_product` bigint(20) NOT NULL,
  `orders` int(11) NOT NULL,
  `date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `id_product` (`id_product`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

```sql
INSERT INTO `statistics` (`id`, `id_product`, `orders`, `date`) VALUES
(1, 1, 1, '2017-09-04 12:52:15'),
(3, 2, 1, '2017-09-04 10:22:44'),
(4, 3, 10, '2017-09-04 12:32:31'),
(5, 4, 2, '2017-09-04 15:24:51'),
(6, 5, 22, '2017-09-04 23:22:15'),
(7, 6, 5, '2017-09-04 11:22:11'),
(8, 7, 26, '2017-09-04 01:22:14'),
(9, 8, 8, '2017-09-04 20:22:12'),
(10, 9, 18, '2017-09-04 19:22:31'),
(11, 1, 1, '2017-09-12 12:22:41'),
(12, 2, 10, '2017-09-12 22:22:21'),
(13, 3, 3, '2017-09-12 10:22:51'),
(14, 4, 5, '2017-09-12 01:22:21'),
(15, 5, 18, '2017-09-12 06:22:31'),
(16, 6, 1, '2017-09-12 14:22:14'),
(17, 7, 12, '2017-09-12 12:22:51'),
(18, 8, 7, '2017-09-12 11:22:11'),
(19, 9, 30, '2017-09-12 18:22:51'),
(20, 1, 2, '2017-09-14 13:22:01'),
(21, 1, 2, '2017-09-15 09:22:02'),
(22, 2, 3, '2017-09-15 02:22:33');

```
```sql
SELECT 
      `id_product`, 
      SUM(`orders`) AS `orders`, 
      DATE(`date`) AS `date` 
FROM `statistics`
WHERE `id_product` = 1 GROUP BY `date`;
```

```sql
SET @count_value_order = 0;
```

```sql
SELECT 
       `products`.`name` AS `name`, 
       (@count_value_order := @count_value_order + `orders`) AS `orders`, 
       `date` 
FROM (
          SELECT 
                 `id_product`, 
                 SUM(`orders`) AS `orders`, 
                 DATE(`date`) AS `date` 
            FROM `statistics` 
           WHERE `id_product` = 3
        GROUP BY `date`) AS `statistics` 
JOIN `products` ON `statistics`.`id_product` = `products`.`id`;
```

### 2. Ранжирование и создание блоков данных фиксированного размера

На Sqlfiddle http://www.sqlfiddle.com/#!9/e7c70d/2/1

```sql
CREATE TABLE `people` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(100) CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

```sql
INSERT INTO `people` (`id`, `name`)
VALUES
	(1,'Иванов Иван'),
	(2,'Андрей Петров'),
	(3,'Марина Васильевна'),
	(4,'Фёдор Николаевич'),
	(5,'Дмитрий Игоревич'),
	(6,'Семён Николаевич'),
	(7,'Василий Алибабаевич'),
	(8,'Никита Петрович'),
	(9,'Зинаида Михайловна'),
	(10,'Ольга Николаевна'),
	(11,'Юлия Матвеева');
```

Ранжирование строк:

```sql
SELECT 
	`people`.`id`, 
	`people`.`name`, 
	(SELECT COUNT(*) FROM `people` AS `people_line` WHERE `people`.`id` > `people_line`.`id`) + 1 AS `rank` 
FROM `people`;
```

Или так:

```sql
SELECT 
COUNT(*) AS `rank`,
`p`.`id`,
`p`.`name`
FROM `people` AS `p` JOIN `people` ON `p`.`id` >= `people`.`id`
GROUP BY `p`.`id`, `p`.`name`;
```

Создание групп из людей. Количество персон в каждой группе 3. Количество групп не фиксированно, установлено требование 
на количество элементов в группах. 

```sql
SELECT CEIL(`rank` / 3) AS `group`, `id`, `name`
FROM 
      (SELECT 
          `people`.`id`, 
          `people`.`name`, 
          (SELECT COUNT(*) FROM `people` AS `people_line` WHERE `people`.`id` > `people_line`.`id`) + 1 AS `rank` 
      FROM `people`) AS `table_rank`;
ORDER BY `group`  
```

Создание групп из людей. Количество персон в каждой не опроеделено. Количество групп фиксированно и рано 3 (Обратная задача).

http://www.sqlfiddle.com/#!9/e7c70d/47

```sql
SELECT 
MOD(COUNT(*), 3) + 1 AS `group`,
`p`.`id`,
`p`.`name`
FROM `people` AS `p` 
JOIN `people` ON `p`.`id` <= `people`.`id`
GROUP BY `p`.`id`, `p`.`name`
ORDER BY `group` ASC;
```

### 3. Сумма из связаных таблиц. 
 
Есть связанные таблицы - Процедуры (1) <--> (n) Лоты (1) <--> (n) Заказчики. Написать запрос, который должен выводить наименование процедуры и суммарный 
НДС из лотов, суммарный НДС, уплаченый заказчиками. 

```sql
CREATE TABLE IF NOT EXISTS `procedures` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) CHARACTER SET latin1 NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 AUTO_INCREMENT=3 ;
```

```sql
INSERT INTO `procedures` (`id`, `name`) VALUES
(1, 'Procedure one'),
(2, 'Other procedure');
```

```sql
CREATE TABLE IF NOT EXISTS `lots` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `procedure_id` int(11) NOT NULL,
  `vat` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 ;
```

```sql
INSERT INTO `lots` (`id`, `procedure_id`, `vat`) VALUES
(1, 1, 320),
(2, 1, 100),
(3, 2, 40),
(4, 2, 10);
```

```sql
CREATE TABLE IF NOT EXISTS `customers` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `lot_id` int(11) NOT NULL,
  `vat_customer` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 ;
```

```sql
INSERT INTO `customers` (`id`, `lot_id`, `vat_customer`) VALUES
(1, 1, 10),
(2, 1, 35),
(3, 2, 190),
(4, 2, 120),
(5, 3, 1000),
(6, 3, 10),
(7, 4, 50),
(8, 4, 150);
```

##### Способ 1

http://www.sqlfiddle.com/#!9/325ada/1

```sql
SELECT 
  `p`.`name` AS `name`, 
  `l`.`vat_sum`, 
  `table_vat_customer`.`vat_customer_sum`
FROM `procedures` AS `p` 
LEFT JOIN (
    SELECT 
      `lots`.`id`, 
      `lots`.`procedure_id` AS `procedure_id`,
      SUM(`lots`.`vat`) AS `vat_sum`
      FROM `lots` 
      GROUP BY `lots`.`procedure_id`
) AS `l` ON `l`.`procedure_id` = `p`.`id` 
JOIN (
  SELECT 
      `tbl`.`procedure_id`, 
      SUM(`vat_customer_by_lots`) AS 
      `vat_customer_sum` 
   FROM (
     SELECT `lots`.`procedure_id`, 
     SUM(`customers`.`vat_customer`) AS `vat_customer_by_lots` 
     FROM `lots` LEFT JOIN `customers` ON `lots`.`id` = `customers`.`lot_id` 
     GROUP BY `customers`.`lot_id`) AS `tbl` GROUP BY `tbl`.`procedure_id`
) AS `table_vat_customer` ON `table_vat_customer`.`procedure_id` = `l`.`procedure_id`
```

##### Способ 2

http://www.sqlfiddle.com/#!9/325ada/15

```sql
SELECT 
`procedures`.`name`,
SUM(`vat`) AS `vat_sum`,  
SUM(`vat_customer_sum`) AS `vat_customer_sum`
FROM (
       SELECT `procedure_id`, `vat`, (
            SELECT SUM(`vat_customer`) 
            FROM `customers` 
            WHERE `customers`.lot_id = `l`.`id`) AS `vat_customer_sum`
        FROM `lots` AS `l`) AS `table_sum`
        JOIN `procedures` ON `procedures`.`id` = `table_sum`.`procedure_id`
GROUP BY `procedure_id`;
```