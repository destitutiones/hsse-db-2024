# Материалы

- [Лекция 7](https://drive.google.com/file/d/1NySY7H8nHOg7dHpZx1-zyo3WU-XF7q4z/view?usp=sharing)
- Документация по  [view](https://postgrespro.ru/docs/postgresql/14/sql-createview)

# Теория

## View

**Представление (view)** – это виртуальная таблица, содержимое которой (столбцы и строки) определяется запросом.

**Свойства view:**
* Не является самостоятельной частью набора данных
* Вычисляется динамически на основании данных, хранящихся в реальных таблицах
* Изменение данных в таблицах немедленно отражается в содержимом представлений

Представление можно использовать в следующих целях:
* Для направления, упрощения и настройки восприятия информации в базе данных каждым пользователем.
* В качестве механизма безопасности, позволяющего пользователям обращаться к данным через представления, но не дающего
  им разрешений на непосредственный доступ к базовым таблицам.

**Достоинства:**
* *Безопасность:* можно искусственно ограничивать информацию, к которой у пользователя есть доступ.
* *Простота запросов:* при написании запросов обращаемся к вью, как и к обычной таблице.
* *Защита от изменений:* пользователю не обязательно знать, что структуры / имена таблиц поменялись. Достаточно обновить представление.

**Недостатки:**
* *Производительность:* кажущийся простым запрос с использованием вью на деле может оказаться очень сложным из-за логики, “зашитой” во вью.
* *Управляемость:* вью может быть основана на вью, которая в свою очередь тоже основана на другой вью и т.д.

**Синтаксис:**

```sql
CREATE
[ OR REPLACE ] [ TEMP | TEMPORARY ] [ RECURSIVE ] 
  VIEW name [ ( column_name [, ...] ) ]
  [
WITH (view_option_name [= view_option_value] [, ...]) ]
  AS query
  [
WITH [ CASCADED | LOCAL ] CHECK
OPTION ]
```

* `CREATE VIEW` – создание нового представления.
* `CREATE OR REPLACE VIEW` – создание или замена уже существующего представления.
    * В случае замены в новом представлении должны присутствовать все поля старого представления (имена, порядок, тип данных). Допустимо только добавление новых полей.
* `TEMPORARY | TEMP` – временное представление, будет существовать до конца сессии.
* *view_name* – название представления.
* *column_name* – список полей представления. Если не указан, используются поля запроса.
* *query* – `SELECT` или `VALUES` команды.

[Изменения представления](https://www.postgresql.org/docs/current/sql-alterview.html):

```sql
ALTER VIEW [IF EXISTS] name ALTER [COLUMN] column_name SET DEFAULT
expression
ALTER VIEW [IF EXISTS] name ALTER [COLUMN] column_name DROP DEFAULT
ALTER VIEW [IF EXISTS] name OWNER TO new_owner
ALTER VIEW [IF EXISTS] name RENAME TO new_name
ALTER VIEW [IF EXISTS] name SET SCHEMA new_schema
ALTER VIEW [IF EXISTS] name SET ( view_option_name [=
view_option_value] [, ... ] )
ALTER VIEW [IF EXISTS] name RESET ( view_option_name [, ... ] )
DROP VIEW [IF EXISTS] name [, ...] [ CASCADE | RESTRICT ]
```

Примеры:

1. Создание представления

```sql
CREATE VIEW greeting AS
SELECT 'Hello World';

CREATE VIEW greeting AS
SELECT text 'Hello World' AS hello;


CREATE VIEW comedy AS
SELECT 
    *
FROM 
    films
WHERE 
    kind = 'Comedy';
```

Если после создания представления добавить столбцы в таблицу, в представлении их не будет.

 <details>
   <summary>Почему так?</summary>

При парсинге запроса при создании view `*` преобразуется в перечисление всех имеющихся на момент создания колонок отношения.

 </details>


**TEMP или TEMPORARY**

Представление создаётся как временное. Удаляется при окончании сессии

```sql
CREATE TEMP VIEW greeting AS
SELECT 'Hello World';
```

**RECURSIVE**

Представление создаётся как рекурсивное. Эквивалентные формы:

```sql
CREATE RECURSIVE VIEW [ schema. ] view_name (column_names) AS
SELECT ...;
```

```sql
CREATE VIEW [ schema.] view_name AS
WITH RECURSIVE view_name (column_names) AS (SELECT ...)
SELECT column_names
FROM view_name;
```

**Пример рекурсивного представления:**

```sql
CREATE RECURSIVE VIEW public.nums_1_100 (n) AS
VALUES (1)
UNION ALL
SELECT
    n + 1
FROM 
    nums_1_100
WHERE 
    n < 100;
```

**Типы представлений:**
1. *горизонтальное* — ограничение данных по строкам:
```sql
CREATE VIEW V_IT_EMPLOYEE AS
SELECT 
    *
FROM 
    EMPLOYEE
WHERE 
    DEPARTMENT_NM = 'IT';
```

2. *вертикальное* — ограничение данных по столбцам:
```sql
CREATE VIEW V_EMP AS
SELECT 
    EMP_NM, 
    DEPARTMENT_NM
FROM 
    EMPLOYEE;
```

**Обновляемые представления**

Представление называется *обновляемым*, если к нему применимы операции `UPDATE` и `DELETE` для изменения данных в таблицах, на которых построено это представление.

**Требования:**

* Ровно 1 источник в предложении `FROM`, являющийся таблицей или обновляемым представлением
* Запрос не должен содержать `WITH`, `DISTINCT`, `GROUP BY`, `HAVING`, `LIMIT` или `OFFSET`
* Запрос не должен содержать операторов `UNION`, `INTERSECT` или `EXCEPT`
* select-list запроса не должен содержать агрегатных, оконных, а также функций, возвращающих множества.

```postgresql
WITH [ CASCADED | LOCAL ] CHECK
OPTION
```

Задаёт поведение обновляемым представлениям: проверки, не позволяющие записывать данные, невидимые через представление

* `LOCAL` – проверки выполняются только на самом представлении
* `CASCADED` – проверки выполняются и на самом представлении, и на источнике, и так далее по цепочке обращений

Обновляемые представления – пример 1:

```postgresql
CREATE VIEW universal_comedy AS
SELECT 
    *
FROM 
    comedy
WHERE 
    classification = 'U'
WITH LOCAL CHECK OPTION;
```

Попытка вставить или отредактировать ряд с `classification <> 'U'` приведёт к ошибке. \
Но при этом вставка или редактирование ряда с `kind <> 'Comedy'` будет успешной.

Обновляемые представления – пример 2:

```postgresql
CREATE VIEW universal_comedies AS
SELECT 
    *
FROM 
    comedy
WHERE 
    classification = 'U'
WITH CASCADED CHECK OPTION;
```

Попытка вставить или отредактировать ряд с classification <> 'U' или kind <> 'Comedy' приведёт к ошибке.

Столбцы в обновляемом представлении могут быть как обновляемые, так и не обновляемые.

Обновляемые представления – пример 3:

```postgresql
CREATE VIEW comedy AS
SELECT 
    f.*,
    country_code_to_name(f.country_code) AS country,
    (SELECT avg(r.rating) FROM user_ratings r WHERE r.film_id = f.id) AS avg_rating
FROM 
    films f
WHERE 
    f.kind = 'Comedy';
```

Все столбцы таблицы `films` – обновляемые. Столбцы `country` и `avg_rating` – `readonly`.

Если представление не удаётся сделать обновляемым, но в этом есть потребность – используйте `INSTEAD OF` триггер.

Это такая функция, которая будет обрабатывать операции модификации данных – рассмотрим позже.


## Практика

### View

Будем работать с таблицами `seminar_7.film` и `seminar_7.film_translation`.

1. Создать view – полную копию таблицы `seminar_7.film`;
2. Создать view – копию таблицы `seminar_7.film` для фильмов, которые представлены на двух или более языках.
3. Создать view с полным списком фильмов и колонкой-списком языков, на которые он переведен. Не добавляйте колонку `code`.
4. Создать view со статисткой по жанрам: для каждого жанра выведите количество фильмов этого типа в таблице `seminar_7.film`, а также дату выхода самого раннего фильма такого типа.
5. Написать вставку записи (на своё усмотрение) во view из пункта (1). Проверить, что новая запись появилась в исходной
   таблице;
6. Написать удаление записи, вставленной в пункте (5), через view из пункта (1). Проверить, что запись удалилась из
   исходной таблицы;
7. Обновить дату выхода фильма с `code='00008'` на '1995-05-01' через view из пункта (1);
8. Удалить информацию о переводах фильма с `code='00008'` и убедиться, что фильм пропал из view из пункта (2);
9. Пересоздать view и пункта (2) с условием [with local check option]. Попробовать проделать те же манипуляции, что в
   пункте (8) на фильме `code='00010'`.
