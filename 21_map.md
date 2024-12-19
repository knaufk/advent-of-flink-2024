# Advent of Flink - Day #21 `MAP`

Another day, another data type. `MAP`s. Let's start by creating a `MAP`. 

```sql
SELECT MAP['Java', 5, 'SQL', 4, 'Python', 3] AS my_map;
```
returns
```
{Java=5, Python=3, SQL=4}
```
You can retrieve a specific value from a map via square brackets. 
```
CREATE VIEW maps AS SELECT MAP['Java', 5, 'SQL', 4, 'Python', 3] AS my_map;
```
and 
```
SELECT my_map['Java'] FROM maps;
```
which retunrs
```
EXPR$0
5
```
Flink has a couple of related built-in functions, too. First, there are three functions to retrieve the content of a map as an array: 

* `MAP_ENTRIES` returns an `ARRAY` of `ROW`s with the key and value as the two columns of these rows.
* `MAP_VALUES` returns an `ARRAY` of the values of the map.
* `MAP_KEY` returns an `ARRAY` of the keys of the map.

The other way around, you can use `MAP_FROM_ARRAYS` to create a map from two arrays: one for the keys, one for the arrays. Or you can use
`STR_TO_MAP` to create a `MAP` from `STRING` `MAP_UNION` you can use to merge two maps. As an example, in the following I use `MAP_FROM_ARRAYS` 
to get to table that contains user_id and for each user_id a map from urls to the number of clicks from this user on that url. 
```sql
SELECT 
 user_id, 
 MAP_FROM_ARRAYS(ARRAY_AGG(url), ARRAY_AGG(cnt))
FROM 
( 
  SELECT 
    user_id, 
    url,
    COUNT(*) cnt
  FROM `examples`.`marketplace`.`clicks`
  GROUP BY user_id, url
)
GROUP BY user_id;
```
As always (so far), the examples in here are runnable out of the box on [Confluent Cloud](https://confluent.cloud).



