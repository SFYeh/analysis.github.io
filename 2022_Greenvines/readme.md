# 綠藤生機 - 新客回購率提升分析
本分析使用 SQLite Online 作為資料庫引擎

## 新客回購表現確認
~~~~sql
-- 新客數
 SELECT firsttransactionyear,
 		COUNT(customerid) AS `新客數`
  FROM Customers
  --WHERE str(firsttransactiondate,6,2) in ('01','02','03')
  GROUP by firsttransactionyear
~~~~
## 新客首購品項區隔
## 首購品項 × 通路
