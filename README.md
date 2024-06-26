**Case Study Introduction**

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat — the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

**Problem Statement**

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

sales

menu

members

**Case Study Questions**

Each of the following case study questions can be answered using a single SQL statement:

1.What is the total amount each customer spent at the restaurant?

2.How many days has each customer visited the restaurant?

3.What was the first item from the menu purchased by each customer?

4.What is the most purchased item on the menu and how many times was it purchased by all customers?

5.Which item was the most popular for each customer?

6.Which item was purchased first by the customer after they became a member?

7.Which item was purchased just before the customer became a member?

8.What is the total items and amount spent for each member before they became a member?

9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

**Bonus Questions**

Join All The Things
The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.



**Schema (PostgreSQL v14)**

    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
    VALUES
      ('A', '2021-01-01', '1'),
      ('A', '2021-01-01', '2'),
      ('A', '2021-01-07', '2'),
      ('A', '2021-01-10', '3'),
      ('A', '2021-01-11', '3'),
      ('A', '2021-01-11', '3'),
      ('B', '2021-01-01', '2'),
      ('B', '2021-01-02', '2'),
      ('B', '2021-01-04', '1'),
      ('B', '2021-01-11', '1'),
      ('B', '2021-01-16', '3'),
      ('B', '2021-02-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-07', '3');
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');
     
     
    
    
    

---

**Query #1**

    select distinct s.customer_id, sum(m.price) over (partition by s.customer_id ) as total_spent 
    from dannys_diner.sales s
    join dannys_diner.menu m on s.product_id = m.product_id
    order by s.customer_id asc;

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---
**Query #2**

    select customer_id, count(distinct (order_date)) as visit   
    from dannys_diner.sales
    group by customer_id;

| customer_id | visit |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

---
**Query #3**

    with t1 as
    (select customer_id, order_date, product_id,dense_rank() over (Partition by customer_id order by order_date) as RN
    from dannys_diner.sales)
    
    select t1.customer_id, t1.product_id
    from t1
    where RN = 1
    group by t1.customer_id, t1.product_id;

| customer_id | product_id |
| ----------- | ---------- |
| A           | 1          |
| A           | 2          |
| B           | 2          |
| C           | 3          |

---
**Query #4**

    select product_name as most_purchased_item , count(s.product_id) over (partition by s.product_id) as repeat
    from dannys_diner.sales s
    join dannys_diner.menu m on s.product_id = m.product_id
    order by repeat desc
    limit 1;

| most_purchased_item | repeat |
| ------------------- | ------ |
| ramen               | 8      |

---
**Query #5**

    with t1 as
    (select customer_id, product_name, count(s.product_id) as quantity, dense_rank() over (partition by customer_id order by count(customer_id) desc) as rank
    from dannys_diner.sales s
    join dannys_diner.menu m on s.product_id = m.product_id
    group by customer_id, product_name)
    
    select t1.customer_id, t1.product_name, quantity
    from t1
    where rank = 1
    ;

| customer_id | product_name | quantity |
| ----------- | ------------ | -------- |
| A           | ramen        | 3        |
| B           | ramen        | 2        |
| B           | curry        | 2        |
| B           | sushi        | 2        |
| C           | ramen        | 3        |

---
**Query #6**

    with t1 as
    (select s.customer_id, s.product_id, join_date, order_date, product_name, row_number() over (Partition by s.customer_id order by order_date) as rn 
    from dannys_diner.sales s
    join dannys_diner.members m on s.customer_id = m.customer_id
    join dannys_diner.menu n on s.product_id = n.product_id
    where order_date >= join_date)
    
    select t1.customer_id, t1.product_id, t1.product_name
    from t1
    where rn = 1;

| customer_id | product_id | product_name |
| ----------- | ---------- | ------------ |
| A           | 2          | curry        |
| B           | 1          | sushi        |

---
**Query #7**

    with t1 as
    (select s.customer_id, s.product_id, join_date, order_date, product_name, row_number() over (Partition by s.customer_id order by order_date desc) as rn 
    from dannys_diner.sales s
    join dannys_diner.members m on s.customer_id = m.customer_id
    join dannys_diner.menu n on s.product_id = n.product_id
    where join_date > order_date)
    
    select t1.customer_id, t1.product_id,t1.product_name
    from t1
    where rn = 1;

| customer_id | product_id | product_name |
| ----------- | ---------- | ------------ |
| A           | 1          | sushi        |
| B           | 1          | sushi        |

---
**Query #8**

    with t1 as
    (select s.customer_id, s.product_id,price 
    from dannys_diner.sales s
    join dannys_diner.menu m on s.product_id = m.product_id
    join dannys_diner.members t on s.customer_id = t.customer_id
    where join_date > order_date)
    
    select t1.customer_id,count(t1.product_id) AS total_items, sum(t1.price) AS amount_spent
    from t1
    group by t1.customer_id
    ORDER by t1.customer_id;

| customer_id | total_items | amount_spent |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 3           | 40           |

---
**Query #9**

    with t1 as
    (select s.customer_id, s.product_id, m.product_name, CASE WHEN m.product_id = 1 THEN m.price * 20 else m.price * 10 end as total_poin
    from dannys_diner.sales s
    join dannys_diner.menu m on s.product_id = m.product_id)
    
    select t1.customer_id, sum(total_poin) as total_poin
    from t1
    group by t1.customer_id
    order by t1.customer_id
    ;

| customer_id | total_poin |
| ----------- | ---------- |
| A           | 860        |
| B           | 940        |
| C           | 360        |

---
**Query #10**

    with t1 as
    (select s.customer_id, s.product_id, m.product_name, CASE WHEN 
    order_date between join_date and (join_date + 6) then m.price * 20 else case when m.product_id = 1 THEN m.price * 20 else m.price * 10 end end as total_poin
    from dannys_diner.sales s
    join dannys_diner.menu m on s.product_id = m.product_id
    join dannys_diner.members b on s.customer_id = b.customer_id
    where order_date <= '2021-01-31')
    
    select t1.customer_id, sum(total_poin) as total_poin
    from t1
    group by t1.customer_id
    order by t1.customer_id;

| customer_id | total_poin |
| ----------- | ---------- |
| A           | 1370       |
| B           | 820        |

---
**Query #11**

    select s.customer_id, s.order_date, m.product_name, m.price, case when s.order_date >= b.join_date then 'Y' 
    when s.order_date < b.join_date then 'N'
    else 'N'
    end as member
    from dannys_diner.sales s
    join dannys_diner.menu m on s.product_id = m.product_id
    join dannys_diner.members b on s.customer_id = b.customer_id
    order by s.customer_id, s.order_date
    ;

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/o7UjiuCkus1TiKeQdeYSar/2)
