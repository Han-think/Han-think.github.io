# SQL 20문제 정답 채움판



Colab에서 `run_sql(""" ... """)` 그대로 실행하면 됩니다.


## S01. customers 전체 조회

```python
run_sql("""
SELECT *
FROM customers;
""")
```


## S02. 국가별 고객 수

```python
run_sql("""
SELECT
    country,
    COUNT(*) AS customer_count
FROM customers
GROUP BY country
ORDER BY customer_count DESC;
""")
```


## S03. 2004년 주문일별 주문 수

```python
run_sql("""
SELECT
    orderDate,
    COUNT(*) AS order_count_2004
FROM orders
WHERE strftime('%Y', orderDate) = '2004'
GROUP BY orderDate
ORDER BY orderDate;
""")
```


## S04. 제품 라인별 평균 MSRP

```python
run_sql("""
SELECT
    productLine,
    ROUND(AVG(MSRP), 2) AS avg_msrp
FROM products
GROUP BY productLine
ORDER BY avg_msrp DESC;
""")
```


## S05. 국가별 총 주문 금액

```python
run_sql("""
SELECT
    c.country,
    ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS total_sales
FROM customers AS c
JOIN orders AS o
    ON c.customerNumber = o.customerNumber
JOIN orderdetails AS od
    ON o.orderNumber = od.orderNumber
GROUP BY c.country
ORDER BY total_sales DESC;
""")
```


## S06. 고객별 주문 수

```python
run_sql("""
SELECT
    c.customerName,
    COUNT(o.orderNumber) AS order_count
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName
ORDER BY order_count DESC;
""")
```


## S07. 주문 수가 2건 이상인 고객

```python
run_sql("""
SELECT
    c.customerName,
    COUNT(o.orderNumber) AS order_count
FROM customers AS c
JOIN orders AS o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName
HAVING COUNT(o.orderNumber) >= 2
ORDER BY order_count DESC;
""")
```


## S08. 주문한 적 있는 고객만 조회 - IN 서브쿼리

```python
run_sql("""
SELECT
    customerNumber,
    customerName,
    country
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber
    FROM orders
)
ORDER BY customerNumber;
""")
```


## S09. 주문이 없는 고객 찾기 - LEFT JOIN

```python
run_sql("""
SELECT
    c.customerNumber,
    c.customerName,
    c.country
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customerNumber = o.customerNumber
WHERE o.orderNumber IS NULL;
""")
```


## S10. 고객 신용한도 순위 - RANK

```python
run_sql("""
SELECT
    customerName,
    country,
    creditLimit,
    RANK() OVER (ORDER BY creditLimit DESC) AS credit_rank
FROM customers
ORDER BY credit_rank;
""")
```


## S11. 상품라인별 주문수량 합계

```python
run_sql("""
SELECT
    p.productLine,
    SUM(od.quantityOrdered) AS total_quantity
FROM products AS p
JOIN orderdetails AS od
    ON p.productCode = od.productCode
GROUP BY p.productLine
ORDER BY total_quantity DESC;
""")
```


## S12. 주문별 총액

```python
run_sql("""
SELECT
    orderNumber,
    ROUND(SUM(quantityOrdered * priceEach), 2) AS order_total
FROM orderdetails
GROUP BY orderNumber
ORDER BY order_total DESC;
""")
```


## S13. 평균 주문총액보다 큰 주문

```python
run_sql("""
WITH order_totals AS (
    SELECT
        orderNumber,
        SUM(quantityOrdered * priceEach) AS order_total
    FROM orderdetails
    GROUP BY orderNumber
)
SELECT
    orderNumber,
    ROUND(order_total, 2) AS order_total
FROM order_totals
WHERE order_total > (
    SELECT AVG(order_total)
    FROM order_totals
)
ORDER BY order_total DESC;
""")
```


## S14. 제품별 판매액

```python
run_sql("""
SELECT
    p.productName,
    p.productLine,
    ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS product_sales
FROM products AS p
JOIN orderdetails AS od
    ON p.productCode = od.productCode
GROUP BY p.productCode, p.productName, p.productLine
ORDER BY product_sales DESC;
""")
```


## S15. 주문 상태별 주문 수

```python
run_sql("""
SELECT
    status,
    COUNT(*) AS order_count
FROM orders
GROUP BY status
ORDER BY order_count DESC;
""")
```


## S16. CASE로 신용한도 등급

```python
run_sql("""
SELECT
    customerName,
    creditLimit,
    CASE
        WHEN creditLimit >= 100000 THEN 'high'
        WHEN creditLimit >= 70000 THEN 'mid'
        ELSE 'low'
    END AS credit_group
FROM customers
ORDER BY creditLimit DESC;
""")
```


## S17. NTILE로 4구간 나누기

```python
run_sql("""
SELECT
    customerName,
    creditLimit,
    NTILE(4) OVER (ORDER BY creditLimit DESC) AS credit_quartile
FROM customers
ORDER BY credit_quartile, creditLimit DESC;
""")
```


## S18. 제품별 평균 주문 단가

```python
run_sql("""
SELECT
    productCode,
    ROUND(AVG(priceEach), 2) AS avg_price
FROM orderdetails
GROUP BY productCode
ORDER BY avg_price DESC;
""")
```


## S19. CTE로 국가별 매출 TOP 3

```python
run_sql("""
WITH country_sales AS (
    SELECT
        c.country,
        SUM(od.quantityOrdered * od.priceEach) AS total_sales
    FROM customers AS c
    JOIN orders AS o
        ON c.customerNumber = o.customerNumber
    JOIN orderdetails AS od
        ON o.orderNumber = od.orderNumber
    GROUP BY c.country
)
SELECT
    country,
    ROUND(total_sales, 2) AS total_sales
FROM country_sales
ORDER BY total_sales DESC
LIMIT 3;
""")
```


## S20. UNION ALL로 연도별 주문 고객 합치기

```python
run_sql("""
SELECT
    '2003' AS year_label,
    customerNumber
FROM orders
WHERE strftime('%Y', orderDate) = '2003'

UNION ALL

SELECT
    '2004' AS year_label,
    customerNumber
FROM orders
WHERE strftime('%Y', orderDate) = '2004';
""")
```
