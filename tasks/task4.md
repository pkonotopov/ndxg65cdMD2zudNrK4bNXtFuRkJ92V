# Внешние ключи (foreign keys), написание и оптимизация запросов

## Задача

На нагруженной БД необходимо:

1. Cоздать внешние ключи на таблицах
   1. Создайте внешние ключи между таблицами `users` и `votes` и любой другой таблицой в БД
   2. Опишите возможные проблемы при создании внешних ключей на нагруженной БД
   3. Преложите вариант действий для минимизации издержек при таких изменениях
   4. Придумайте вариант, что делать с данными, которые добавятся к БД во время создания FK
2. Необходимо составить SQL-запросы для получения следующей информации:
   1. 25 наиболее активных пользователей, вывод должен содержать `display_name` и суммарное количество постов каждого пользователя  (то есть сколько постов написал каждый за все время активности БД), отсортированные по убыванию количества постов.
   2. Топ 10 постов с наибольшим количеством дискуссионных сообщений в них (`answers`) отсортированные по убыванию количества дискуссионных сообщений.
   3. Топ 10 постов с наибольшим количеством голосов (`votes`) отсортированные по убыванию количества голосов.
3. Улучшите время выполнения следующего запроса:
```sql
SELECT
    u.display_name
   ,string_agg(b.name, ',') 
FROM users u 
JOIN badges b on (u.id = b.user_id) 
GROUP BY u.display_name;
```
Если выполнить этот запрос с `EXPLAIN (ANALYZE,BUFFERS,VERBOSE)`, то мы получим примерно следующий план выполнения:
```text
 GroupAggregate  (cost=64975.39..69307.51 rows=126416 width=42) (actual time=4694.458..6429.433 rows=112247 loops=1)
   Output: u.display_name, string_agg((b.name)::text, ','::text)
   Group Key: u.display_name
   Buffers: shared hit=2907 read=5735, temp read=3311 written=3319
   ->  Sort  (cost=64975.39..65892.70 rows=366922 width=21) (actual time=4694.279..5782.348 rows=366922 loops=1)
         Output: u.display_name, b.name
         Sort Key: u.display_name
         Sort Method: external merge  Disk: 11560kB
         Buffers: shared hit=2907 read=5735, temp read=3311 written=3319
         ->  Hash Join  (cost=11467.01..23537.42 rows=366922 width=21) (actual time=414.372..1589.905 rows=366922 loops=1)
               Output: u.display_name, b.name
               Inner Unique: true
               Hash Cond: (b.user_id = u.id)
               Buffers: shared hit=2907 read=5735, temp read=1866 written=1866
               ->  Seq Scan on public.badges b  (cost=0.00..6512.22 rows=366922 width=15) (actual time=0.158..258.171 rows=366922 loops=1)
                     Output: b.name, b.user_id
                     Buffers: shared hit=2843
               ->  Hash  (cost=7868.78..7868.78 rows=206978 width=14) (actual time=409.453..409.457 rows=206978 loops=1)
                     Output: u.display_name, u.id
                     Buckets: 131072  Batches: 4  Memory Usage: 3475kB
                     Buffers: shared hit=64 read=5735, temp written=683
                     ->  Seq Scan on public.users u  (cost=0.00..7868.78 rows=206978 width=14) (actual time=0.288..181.848 rows=206978 loops=1)
                           Output: u.display_name, u.id
                           Buffers: shared hit=64 read=5735
 Planning:
   Buffers: shared hit=8
 Planning Time: 4.292 ms
 Execution Time: 6469.260 ms
(28 rows)
```

Результат запроса должен отображать `display_name` и агрегированные бейджы (`badges`).

## Комментарий

Можно создавать любые индексы, которые на ваш взгляд необходимы для ускорения выполнения запроса.

## Результат

Файл в формате Markdown содержащий
1. Изложение подхода к проблеме в п.1, возможный SQL-код миграции данных и иных возможных действий
2. SQL-код запросов из п.2 задания
3. Перечень и обоснование созданных индексов п.3
4. Новый план запроса п.3 с комментарием, что именно дало снижение времени выполнения запроса 
