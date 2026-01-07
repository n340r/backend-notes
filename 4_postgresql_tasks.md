## 1. 📚 PostgreSQL Database Design

- You need to **design a database** (describe entities, their attributes, and relationships between them).

- Business domain: **"User Identification"**.

- Users must complete identification after registration.

- To do this, they must provide minimum personal data: `full name` and `date of birth`.

- The user must also provide a `document number`.

- Documents can be different: `passport`, `INN`, `SNILS`, `visa`, `migration card`.

## 2. 📚 PostgreSQL complicated task

SQL Задача: Гистограмма по числу твитов в 2022 году

### Условие

Вам дана таблица `tweets`, содержащая информацию о твитах пользователей. Напишите SQL-запрос, который вернет гистограмму числа твитов, опубликованных пользователями в 2022 году.

Группируйте пользователей по количеству твитов, которые они опубликовали в 2022 году, и посчитайте количество пользователей в каждой такой группе.

### Структура таблицы `tweets`

| Column Name | Type      |
| ----------- | --------- |
| tweet_id    | integer   |
| user_id     | integer   |
| msg         | string    |
| tweet_date  | timestamp |

### Пример данных

| tweet_id | user_id | msg                                                      | tweet_date |
| -------- | ------- | -------------------------------------------------------- | ---------- |
| 214252   | 111     | Am considering taking Tesla private at $420. Funding...  | 12/30/2021 |
| 739252   | 111     | Despite the constant negative press covfefe              | 01/01/2022 |
| 846402   | 111     | Following @NickSinghTech on Twitter changed my life!     | 02/14/2022 |
| 241425   | 254     | If the salary is so competitive why won’t you tell me... | 03/01/2022 |
| 231574   | 148     | I no longer have a manager. I can't be managed           | 03/23/2022 |

### Ожидаемый результат

| tweet_bucket | users_num |
| ------------ | --------- |
| 1            | 2         |
| 2            | 1         |

### Объяснение

- Пользователь `111` написал **2 твита** в 2022 году.
- Пользователи `254` и `148` написали по **1 твиту** в 2022 году.

Таким образом:

- 2 пользователя попали в "бакет" 1 (по 1 твиту),
- 1 пользователь — в "бакет" 2 (2 твита).

### SQL-решение (PostgreSQL)

```sql
SELECT
  tweet_count AS tweet_bucket,
  COUNT(*) AS users_num
FROM (
  SELECT
    user_id,
    COUNT(*) AS tweet_count
  FROM tweets
  WHERE EXTRACT(YEAR FROM tweet_date) = 2022
  GROUP BY user_id
) AS user_tweet_counts
GROUP BY tweet_count
ORDER BY tweet_bucket;
```

## SQL bonus questions:

- What is `UNION`, `UNION ALL` ? Can you give me an example ?
- What is the difference between `WHERE` and `HAVING` ?
  Can we do `WHERE COUNT(*) > 10` in SQL ?
- Difference between `SERIAL` and `uuid`. Which one is better for sharding ?  
  Is `UUID v4` better than `UUID v7` and why ?
- Can you referency by **non-primary** key ?
- What is `selectivity`. What is `high selectivity` and `low selectivity`
