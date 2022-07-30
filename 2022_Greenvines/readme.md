# 綠藤生機 - 新客回購率提升分析

本分析使用 SQLite Online 作為資料庫引擎，資料庫的 ER Diagram 如下圖所示：

![image](https://github.com/SFYeh/sfyeh.github.io/blob/main/2022_Greenvines/ER%20Diagram.png)

## 新客回購表現確認

- 資料提取、彙整 ( SQL 語法) ：
~~~~sql

WITH `repurchase` AS (
  -- 算出每位客戶 2021 年與 2022 年各別有幾個訂單, 篩選出有回購的人
  SELECT customerid,
  	      transactionyear,
         COUNT(customerid) AS `order_cnt`
  FROM `Orders`
  GROUP BY `customerid`,`transactionyear`
  HAVING `order_cnt`>1
)

-- 計算 2022 與 2021 年各別的新客數、新客回購數
SELECT Customers.FirstTransactionYear,
       COUNT(DISTINCT Customers.CustomerId) as `new_customer_cnt`,
       COUNT(DISTINCT repurchase.CustomerId) as `repurchase_customer_cnt`
   FROM Customers
   LEFT JOIN repurchase
     ON Customers.CustomerId=repurchase.CustomerId
    AND firsttransactionyear=repurchase.transactionyear --鎖定 "新客" 回購
  GROUP BY firsttransactionyear
~~~~


## 新客首購品項區隔

~~~~sql
WITH first_orders AS (
  --首購訂單資料
 	SELECT Customers.CustomerId, Orders.OrderId,Orders.TransactionYear, Products.ProductType
  	FROM Customers
  	JOIN Orders
  	  ON Customers.CustomerId=Orders.CustomerId AND Customers.FirstTransactionDate=Orders.TransactionDate
  	JOIN OrderDetails
  	  ON Orders.OrderId=OrderDetails.OrderId
    JOIN Products
      ON OrderDetails.ProductId =Products.ProductId
  ),
  p_core as ( 
   --首購含核心產品
	SELECT *
  	  FROM first_orders
      WHERE producttype ="核心產品"),
  p_lead AS (
   --首購含帶路產品
    SELECT *
  	  FROM first_orders
      WHERE producttype ="帶路產品"),
  p_others AS (
   --首購含其他產品
    SELECT *
  	  FROM first_orders
      WHERE producttype ="其他產品"),
      
  seg_core_only AS(
   --首購只買核心產品的分組
    SELECT* 
      FROM p_core
     WHERE OrderId NOT IN 
    	(
         SELECT OrderId FROM p_lead
         UNION
         SELECT OrderId FROM p_others
        )
  ),
  seg_lead_only AS(
   --首購只買帶路產品的分組
     SELECT* 
      FROM p_lead
     WHERE OrderId NOT IN 
    	(
         SELECT OrderId FROM p_core
         UNION
         SELECT OrderId FROM p_others
        )
  ),
  seg_others_only AS(
   -- 首購只買其他的分組
    SELECT* 
      FROM p_others
     WHERE OrderId NOT IN 
    	(
         SELECT OrderId FROM p_core
         UNION
         SELECT OrderId FROM p_lead  --要用 UNION 因為兩者會有重複的部分
        )
  ),
  seg_core_and_lead AS(
    --首購同時買核心產品與帶路產品的分組
    SELECT*
      FROM p_core
     WHERE OrderId IN (SELECT OrderId FROM p_lead) 
       AND OrderId NOT IN (SELECT OrderId FROM p_others)
  ),
  seg_core_and_others AS(
    --首購同時買核心產品與其他產品的分組
    SELECT*
      FROM p_core
     WHERE OrderId IN (SELECT OrderId FROM p_others) 
     AND OrderId NOT IN (SELECT OrderId FROM p_lead)
  ),
  seg_lead_and_others AS(
    --首購同時買帶路產品與其他產品的分組
    SELECT*
      FROM p_lead
     WHERE OrderId IN (SELECT OrderId FROM p_others) 
     AND OrderId NOT IN (SELECT OrderId FROM p_core)
  ),   
  seg_core_lead_others AS(
   --首購三種產品都購買的分組
  	SELECT*
      FROM p_core
     WHERE OrderId IN (SELECT OrderId FROM p_lead)
       AND OrderId IN (SELECT OrderId FROM p_others)
  ),
  
  repurchase AS(
  --算出每位客戶各年份的訂單次數, 篩選出有回購的人
  SELECT customerid,
  	   transactionyear,
       COUNT(customerid) AS `order_cnt`
  FROM Orders
  GROUP BY customerid,transactionyear
  HAVING order_cnt>1
  )
  
-- 提取各個 Segment 的新客數、回購數，以只買核心產品為例
SELECT segment.TransactionYear,
       COUNT(DISTINCT segment.customerid) as `new_customer_cnt`,
       COUNT(DISTINCT repurchase.CustomerId) as `repurchase_customer_cnt`
  FROM  seg_core_only AS segment   --以只買核心產品 (seg_core_only) 為例
  LEFT JOIN repurchase
    ON  segment.CustomerId=repurchase.CustomerId
   AND segment.TransactionYear=repurchase.transactionyear
 GROUP BY segment.TransactionYear

~~~~

## 首購品項 × 通路

~~~~sql
~~~~
