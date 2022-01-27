
1.1996年に3回以上注文した（Ordersが3つ以上紐づいている）CustomerのIDと、注文回数を取得してみてください

```SQL
SELECT
 C.CustomerID,
 COUNT(*) AS OrderCount
FROM
 Customers C
 LEFT JOIN
  Orders O
  USING(CustomerID)
WHERE
 O.OrderDate BETWEEN '1996-01-01' AND '1996-12-31'
GROUP BY
 C.CustomerID
HAVING
 OrderCount >= 3
```

2. 最もよく注文してくれたのは、どのCustomerでしょうか？

```SQL
SELECT
 CustomerID,
 COUNT(*) MaxOrder
FROM
 Orders
GROUP BY
  CustomerID
HAVING
  COUNT(*) = (SELECT MAX(CNT)
              FROM
                (SELECT COUNT(*) CNT
                 FROM Orders
                 GROUP BY CustomerID));
```

3. 「一度の注文で、最大どれぐらいの注文詳細が紐づく可能性があるのか」調べる必要が生じました。過去最も多くのOrderDetailが紐づいたOrderを取得してください。何個OrderDetailが紐づいていたでしょうか？

```sql
SELECT
  OrderID, COUNT(*) OrderDetailCount
FROM
  OrderDetails
GROUP BY
  OrderID
HAVING
  COUNT(*) = (SELECT MAX(CNT)
              FROM
                (SELECT COUNT(*) CNT
                 FROM OrderDetails
                 GROUP BY OrderID));
```

4. 「一番お世話になっている運送会社を教えて欲しい」と頼まれました。過去最も多くのOrderが紐づいたShipperを特定してみてください

```sql
SELECT
 ShipperID,
 count(*) AS ShipperCount
FROM
 Orders
GROUP BY
 ShipperID
ORDER BY
 ShipperCount DESC
```

5. 「重要な市場を把握したい」と頼まれました。売上が高い順番にCountryを並べてみましょう

```sql
SELECT
    C.Country,
    ROUND(SUM(od.quantity * p.Price)) AS Sales
FROM
 Customers C
 LEFT JOIN
  Orders O
   USING(CustomerID)
 LEFT JOIN
  OrderDetails OD
   USING(OrderID)
 LEFT JOIN
  Products P
   USING(ProductID)
GROUP BY
 Country
ORDER BY
 Sales DESC
```

6. 国ごとの売上を年毎に（1月1日~12月31日の間隔で）集計してください

```sql
SELECT
 c.Country,
  strftime('%Y', O.OrderDate) AS OrderYear,
  ROUND(SUM(od.Quantity * p.Price)) AS Sales
FROM
 Customers C
 INNER JOIN
  Orders O
  USING(CustomerID)
 INNER JOIN
  OrderDetails OD
   USING(OrderID)
 INNER JOIN
  Products P
   USING(ProductID)
GROUP BY
 Country,
 OrderYear
```

7. Employeeテーブルに「Junior（若手）」カラム（boolean）を追加

```sql
ALTER TABLE
 Employees
ADD
 Junior boolean;

UPDATE
  Employees
SET
 Junior = CASE WHEN BirthDate >= '1960-01-01' THEN true
        ELSE false
        END
```

8. Shipperにlong_relationカラムを追加

```sql
ALTER TABLE
  Shippers
ADD
  long_relation boolean;

UPDATE
  Shippers
SET
  long_relation = CASE
    WHEN y.OrderCount >= 70 THEN TRUE
    ELSE false
  END
FROM
  Shippers s
  LEFT JOIN (
    SELECT
      ShipperID,
      COUNT(OrderID) AS orderCount
    FROM
      Shippers
      LEFT JOIN Orders USING(ShipperID)
    GROUP BY
      ShipperID
  ) y USING(ShipperID)
```

```sql
- Employeeが最後に担当したOrderと、その日付を取得してほしい
    - OrderID, EmployeeID, 最も新しいOrderDate
  SELECT
    E.EmployeeID,
    O.OrderID,
    MAX(O.OrderDate) AS LatestOrderDate
  FROM
    Employees AS E
    JOIN [Orders] AS O ON E.EmployeeID = O.EmployeeID
  GROUP BY
    E.EmployeeID
```

Null

```sql
-- Customerテーブルで任意の１レコードのCustomerNameをNULLにしてください
UPDATE
 Customers
SET
 CustomerName = NULL 
WHERE
 CustomerID = 1

-- CustomerNameが存在するユーザを取得するクエリを作成してください
SELECT
 *
FROM
 Customers
WHERE
 CustomerName IS NOT NULL

-- CustomerNameが存在しない（NULLの）ユーザを取得するクエリを変えてください
SELECT
 *
FROM
 Customers
WHERE
 CustomerName IS NULL

-- もしかすると、CustomerNameが存在しないユーザーを取得するクエリを、このように書いた方がいるかもしれません
-- SELECT * FROM Customers WHERE CustomerName = NULL;
-- しかし残念ながら、これでは期待した結果は得られません。なぜでしょうか？
CustomerNameがNULLの場合、WHERE以下は'NULL = NULL'となる。
返り値はTRUEでなくNULLであるため、knownとなり、比較できなくなる。
```

- JOIN

```sql
-- EmployeeId=1の従業員のレコードを、Employeeテーブルから削除してください
DELETE FROM
 Employees
WHERE
 EmployeeID = 1

-- OrdersとEmployeesをJOINして、注文と担当者を取得してください。
-- （削除された）EmployeeId=1が担当したOrdersを表示しないクエリを書いてください
SELECT
 OrderID,
  LastName
FROM
 Orders
    INNER JOIN
   Employees
        USING(EmployeeID)

-- （削除された）EmployeeId=1が担当したOrdersを表示する（Employeesに関する情報はNULLで埋まる）クエリを書いてください
SELECT
  OrderID,
  E.FirstName
FROM
  [Orders] AS O
  LEFT JOIN Employees AS E ON O.EmployeeID = E.EmployeeID
ORDER BY
  O.EmployeeID
```

## GROUP BYした上で絞り込みを行う際「WHERE」と「HAVING」二つのクエリを使えますが、それぞれの違いを教えてください。

- WHERE：グループ化する前のデータに抽出条件を設定
- HAVING：グループ化した後のデータに抽出条件を設定

## どのような時にどちらを使うべきでしょうか？

- グルーピングを行わなくても指定できる条件の時：WHERE
- 集計関数を使っているときなどグルーピング後しか指定できない条件の時：HAVING

- 課題2

```
- GROUP BYした上で絞り込みを行う際「WHERE」と「HAVING」二つのクエリを使えますが、それぞれの違いを教えてください。
    - GroupByでグルーピングする前に抽出するのが、where
    - GroupByでグルーピングした後に抽出するのが、having
    
- SQLの文脈においてDDL、DML、DCL、TCLとは何でしょうか？それぞれ説明してください
    - DML データ操作言語
        - select , insert, update, delete, explainなど。
    - TCL トランザクション制御言語
        - commit , rollback , BEGINなど
    - DDL データ定義言語
        - テーブルの作成削除、設定など。
        - create, alter, drop, truncate
    - DCL データ制御言語
        - DMLやDDLの利用の許可、禁止を設定
        - grant, revoke
```

- 課題3
先ほどのデータベースで商品カテゴリ別に売上を取得するクエリを考えてみる
