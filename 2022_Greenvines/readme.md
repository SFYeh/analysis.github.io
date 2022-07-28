# 綠藤生機 - 新客回購率提升分析

本分析使用 SQLite Online 作為資料庫引擎，資料庫的 ER Diagram 如下圖所示：
!() [https://github.com/SFYeh/sfyeh.github.io/blob/main/2022_Greenvines/ER%20Diagram.png]

## 新客回購表現確認

- 資料提取、彙整 ( SQL 語法) 
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

比例檢定(R語言)
~~~~r
data=data.frame(New_N=c(8284, 9636), repurchase_N=c(1276,1621))
rownames(data)= c(2021,2022)
prop.test( data$repurchase_N,data$New_N, alternative="two.sided") 
~~~~

## 新客首購品項區隔
## 首購品項 × 通路
