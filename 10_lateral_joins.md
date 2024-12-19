# Advent of Flink - Day #10 Lateral Joins

There is more to be said and more to write about event time and watermarks, but I need a break from this. So, today I 
cover a more fun topic: lateral joins. 

Flink SQL supports lateral joins. The `LATERAL` keyword allows a subquery to reference columns from a table expression 
that precedes that subquery. You can think of it like a loop: For each row in the left-hand table, execute the 
right-hand-subquery using the values from the current row in the left-hand table. Clear? Let's look at an example:

```sql
WITH departments AS (
  SELECT DISTINCT(p.department) FROM `examples`.`marketplace`.`products` AS p
)
SELECT * FROM departments, LATERAL (
  SELECT 1
)
```
returns
```
department EXPR$0
Grocery    1
Baby       1
Automotive 1
...
```
So, for each department the inner query `SELECT 1` was executed. Now, let's look at a more interesting example. We use 
a lateral join to build a query that let's you dynamically filter orders for a dynamically changing set of customers. First, we 
create a new table that contains all customers we are interestested in, and since we can not easily insert a tombstone into the
table via Flink SQL. I'll add a flag that let's us disable a customer that we previously inserted
```sql
CREATE TABLE enabled_customers (
  customer_id INT,
  is_enabled BOOLEAN,
  PRIMARY KEY (customer_id) NOT ENFORCED
);
```
Then, we start the following query and keep it running in a Workspace cell. 
```sql
SELECT o.* FROM enabled_customers AS ec, LATERAL (
  SELECT 
   *  
  FROM `examples`.`marketplace`.`orders`AS o
  WHERE o.customer_id = ec.customer_id
) AS o
WHERE ec.is_enabled = TRUE;
```
Since we haven't enabled any customers yet, there is no output. Now, in a different workspace cell, we insert some customers into 
`enabled_customers`. 
```sql
INSERT INTO enabled_customers VALUES
  (3000, TRUE),
  (3001, TRUE),
  (3002, TRUE);
```
And now, you should see all orders for these three customers returned by the previous query. If you now, run: 
```sql
INSERT INTO enabled_customers VALUES
  (3000, TRUE),
  (3001, FALSE),
  (3002, FALSE);
```
You should see that the orders for customers "3001" and "3002" are retracted again from the output. Nice, right? We have 
essentially implemented the control stream pattern with Flink SQL. 

This query by the way has unbounded state, because it keeps all `orders` in state - just in case you enabled the corresponding 
customer at a later point in time. So, you definitely want to use `sql.state-ttl` as described on [Day 6](./06_statettl.md) to
bound the state of the `orders` table. On the other hand, you don't want any state time-to-live on `enabled_customers`
presumably, because you never want a customer to be enabled or disabled by state expiration - on the other hand it seems reasonable 
not to get all orders from all eternity replayed if a customer is enabled. 

That's all for today. As always (so far), the examples in here are runnable out of the box on Confluent Cloud.


