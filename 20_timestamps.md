# Advent of Flink - Day 20 `TIMESTAMP` vs `TIMESTAMP_LTZ`

Flink comes with two different timestamp types: `TIMESTAMP` and `TIMESTAMP_LTZ` (or `TIMESTAMP WITH LOCAL TIME ZONE`). Both 
`TIMESTAMP` and `TIMESTAMP_LTZ` represent timestamps and can be used for time-based operations, i.e can serve as row time column, 
but beyond that they are quite different. 

* `TIMESTAMP_LTZ` represents a specific point, or instant, in the UTC timeline. It is internally stored as a UTC integer. When
  returned to the user as a string it is dynamically converted to the timezone specified in `sql.local-time-zone`. 

* `TIMESTAMP` does not store any information about the timezone and hence can not represent a specific moment in time and is
  inherently ambigious. It is internally stored as a string. 

Let's look at some consequences of this - hoping that this makes it clearer.


```sql
SET 'sql.local-time-zone' = 'Europe/Berlin';
SELECT 
  TIMESTAMP '2023-05-04 23:42:55'
```
returns 
```
EXP$0
2023-05-04 23:42:55
```
There is no timestamp information in the output. 
```sql
SET 'sql.local-time-zone' = 'America/Los_Angeles';
SELECT 
  TIMESTAMP '2023-05-04 23:42:55'
```
returns exactly the same
```
EXP$0
2023-05-04 23:42:55
```
`TIMESTAMP_LTZ` behaves differently. `$rowtime` of type `TIMESTAMP_LTZ`. So, 
```sql
SET 'sql.local-time-zone' = 'America/Los_Angeles';
SELECT `$rowtime` FROM `examples`.`marketplace`.`clicks`;
```
returns a different timetamp string (but same moment in time) than 
SET 'sql.local-time-zone' = 'Europe/Berlin';
SELECT `$rowtime` FROM `examples`.`marketplace`.`clicks`;
```

`TO_TIMESTAMP_LTZ` takes a unix epoch in seconds or milliseconds and coverts it to `TIMESTAMP_LTZ`. There is no equivalent function
for `TIMESTAMP`. Instead `TO_TIMESTAMP` takes a datetime string and a datatime format string. 

You can cast a `TIMESTAMP` to a `TIMESTAMP_LTZ` and vice-versa. In both directions, `sql.local-time-zone` is used for the conversion.
Consequently, 
```sql
SET 'sql.local-time-zone' = 'Europe/Berlin';
SELECT CAST(TIMESTAMP '2023-05-04 23:42:55' AS TIMESTAMP_LTZ)
```
and 
```sql
SET 'sql.local-time-zone' = 'America/Los_Angeles';
SELECT CAST(TIMESTAMP '2023-05-04 23:42:55' AS TIMESTAMP_LTZ)
```
both return 
```
EXPR$0
2023-05-04 23:42:55.000000
```
because the same timezone is used for the conversation from `TIMESTAMP` to `TIMESTAMP_LTZ` and to print the `TIMESTAMP_LTZ` to the screen.

In summary, you are better of using `TIMESTAMP_LTZ` because it actually represents a point in time, but it's important to understand the differences, because
one can easily be confused by it, specifically when the same query or table has timestamp columns using both types. And that's all for today. As always (so far), 
the examples in here are runnable out of the box on [Confluent Cloud](https://confluent.cloud).
