# Материалы

- [Лекция 10](https://drive.google.com/file/d/15cZwLmy78rqin-2CPSeXU0t608QvqRBE/view?usp=sharing)
- Туториалы:
  - [Пошаговое руководство по работе с SQLite (немного древнее)](https://metanit.com/python/database/)
  - [Как работать с базами данных SQL в Python](https://selectel.ru/blog/tutorials/working-with-sql-databases-in-python/)
- Документация:
  - [Документация по SQLAlchemy The Python SQL Toolkit and Object Relational Mapper](https://www.sqlalchemy.org/)

# Теория

## Работа с базами данных в Python

[Python DB API 2.0](https://www.python.org/dev/peps/pep-0249/) - стандарт
интерфейсов для пакетов, работающих с БД. "Набор правил", которым подчиняются
отдельные модули, реализующие работу с конкретными базами данных.

Для PostgreSQL наиболее популярный адаптер –
[psycopg2](http://initd.org/psycopg/).

| Database   | Interface                                                 |
| ---------- | --------------------------------------------------------- |
| SQLite     | [sqlite3](https://docs.python.org/3/library/sqlite3.html) |
| PostgreSQL | [psycopg2](http://initd.org/psycopg/)                     |
| MySQL      | [PyMySQL](https://pymysql.readthedocs.io/en/latest/)      |
| Oracle     | [cx_Oracle](https://oracle.github.io/python-cx_Oracle/)   |

### 2. Курсоры

**Курсор** – специальный объект, выполняющий запрос и получающий его результат.

Вместо того чтобы сразу выполнять весь запрос, есть возможность настроить
**курсор**, инкапсулирующий запрос, и затем получать результат запроса по
нескольку строк за раз. Одна из причин так делать заключается в том, чтобы
избежать переполнения памяти, когда результат содержит большое количество строк.

В PL/pgSQL:

```sql
name [ [ NO ] SCROLL ] CURSOR [ ( arguments ) ] FOR query;
```

```sql
DECLARE
    curs1 refcursor;
    curs2 CURSOR FOR SELECT * FROM tenk1;
    curs3 CURSOR (key integer) FOR SELECT * FROM tenk1 WHERE unique1 = key;
```

# Практика

- [SQL Alchemy Core](https://colab.research.google.com/drive/1HStnVeJ4W6avWjvxwNsvGq8zahvRCUgK?usp=sharing)
- [SQL Alchemy ORM SQLite](https://colab.research.google.com/drive/1GBsbFqtyhuzDcP9Gnk8wruCB7eqya_NP?usp=sharing)
- [SQL Alchemy PostgreSQL](https://colab.research.google.com/drive/1zK9lAccGqQeysgAA308yIOKrE62O7pvJ?usp=sharing)
- [SQL Alchemy Pandas](https://colab.research.google.com/drive/1fvu8VHjHeSm5LfjEDl2DE3fCoRGFYwJu?usp=sharing)
