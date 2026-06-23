# SQL 20문제 표 해설 강화판

Colab에서는 각 문제의 `run_sql(""" ... """)` 셀을 그대로 실행하면 됩니다.


## 1. SQL 20문제 — 표로 읽는 설명 강화판

이 버전은 정답만 채운 것이 아니라, 문제마다 아래 6가지를 먼저 보게 만들었습니다.

```text
요구사항 → 필요한 컬럼 → 필요한 테이블 → 연결 흐름 → SQL 패턴 → 자주 틀리는 부분
```

초보 기준 핵심은 **SQL을 외우는 것보다 테이블 연결선을 먼저 보는 것**입니다.

### 전체 테이블 지도

| 테이블 | 역할 | 자주 쓰는 컬럼 |
|---|---|---|
| `customers` | 고객 정보 | `customerNumber`, `customerName`, `country`, `creditLimit` |
| `orders` | 주문 머리표 | `orderNumber`, `orderDate`, `status`, `customerNumber` |
| `orderdetails` | 주문 상세/금액 | `orderNumber`, `productCode`, `quantityOrdered`, `priceEach` |
| `products` | 제품 정보 | `productCode`, `productName`, `productLine`, `MSRP` |
| `payments` | 결제 정보 | `customerNumber`, `paymentDate`, `amount` |

### 대표 연결선

```text
customers.customerNumber → orders.customerNumber
orders.orderNumber → orderdetails.orderNumber
products.productCode → orderdetails.productCode
customers.customerNumber → payments.customerNumber
```


### S01. customers 전체 조회

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | customers 테이블의 모든 고객 정보를 확인한다. |
| SQL 패턴 | 단일 테이블 전체 조회 |
| 필요한 테이블 | `customers` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | `SELECT`만 쓰고 `FROM`을 빼먹지 않기. |
| 필요한 컬럼/계산 | 전체 컬럼 `*` |

#### 읽는 순서

```text
1. `FROM customers`로 고객 테이블 선택
2. `SELECT *`로 모든 컬럼 출력
```

#### 정답 코드


```python
run_sql("""
SELECT *
FROM customers;
""")
```


### S02. 국가별 고객 수

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 나라별로 고객이 몇 명인지 센다. |
| SQL 패턴 | GROUP BY + COUNT |
| 필요한 테이블 | `customers` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | `country`를 SELECT에 썼으면 집계하지 않은 컬럼이므로 `GROUP BY country`가 필요하다. |
| 필요한 컬럼/계산 | `country`<br>`COUNT(*) AS customer_count` |

#### 읽는 순서

```text
1. 고객의 나라 컬럼은 `customers.country`
2. 나라별로 묶기 위해 `GROUP BY country`
3. 각 묶음의 행 수를 `COUNT(*)`로 계산
4. 고객 수가 큰 순서로 정렬
```

#### 정답 코드


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


### S03. 2004년 주문일별 주문 수

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 2004년에 발생한 주문을 주문일별로 몇 건인지 센다. |
| SQL 패턴 | WHERE + strftime + GROUP BY |
| 필요한 테이블 | `orders` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | 날짜가 문자열처럼 보여도 연도 추출은 `strftime`을 쓰는 패턴으로 외우기. |
| 필요한 컬럼/계산 | `orderDate`<br>`COUNT(*) AS order_count_2004` |

#### 읽는 순서

```text
1. 주문 날짜는 `orders.orderDate`
2. SQLite에서는 `strftime('%Y', orderDate)`로 연도 추출
3. 2004년만 `WHERE`로 남김
4. 주문일별로 묶고 `COUNT(*)`
```

#### 정답 코드


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


### S04. 제품 라인별 평균 MSRP

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 제품 라인별 평균 권장소비자가격(MSRP)을 구한다. |
| SQL 패턴 | GROUP BY + AVG + ROUND |
| 필요한 테이블 | `products` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | 평균을 낼 때 `AVG(MSRP)`이고, 보기 좋게 할 때만 `ROUND(..., 2)`를 감싼다. |
| 필요한 컬럼/계산 | `productLine`<br>`MSRP`<br>`ROUND(AVG(MSRP), 2) AS avg_msrp` |

#### 읽는 순서

```text
1. 제품 라인은 `products.productLine`
2. MSRP도 `products.MSRP`
3. 제품 라인별로 묶고 평균 계산
4. 소수 둘째 자리까지 반올림
```

#### 정답 코드


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


### S05. 국가별 총 주문 금액

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 국가별 총 주문 금액을 구한다. |
| SQL 패턴 | 3테이블 JOIN + GROUP BY + SUM |
| 필요한 테이블 | `customers`, `orders`, `orderdetails` |
| 연결 흐름 | `customers.customerNumber` → `orders.customerNumber` → `orders.orderNumber` → `orderdetails.orderNumber` |
| 자주 틀리는 부분 | `country`와 `quantityOrdered`가 다른 테이블에 있다. 둘 사이를 `orders`로 이어야 한다. |
| 필요한 컬럼/계산 | 나라: `customers.country`<br>주문 연결: `orders.customerNumber`, `orders.orderNumber`<br>금액: `orderdetails.quantityOrdered * orderdetails.priceEach` |

#### 읽는 순서

```text
1. 나라가 필요하므로 `customers`가 필요
2. 주문 금액은 `orderdetails`에서 수량×가격으로 계산
3. customers와 orderdetails는 직접 연결되지 않으므로 `orders`가 다리 역할
4. 나라별로 묶고 `SUM(수량 * 가격)` 계산
```

#### 정답 코드


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


### S06. 고객별 주문 수

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 고객별 주문 수를 구한다. 주문이 없는 고객도 0으로 보고 싶다. |
| SQL 패턴 | LEFT JOIN + COUNT |
| 필요한 테이블 | `customers`, `orders` |
| 연결 흐름 | `customers.customerNumber` → `orders.customerNumber` |
| 자주 틀리는 부분 | 주문 없는 고객까지 보려면 `JOIN`보다 `LEFT JOIN`이 안전하다. |
| 필요한 컬럼/계산 | `customers.customerName`<br>`orders.orderNumber` |

#### 읽는 순서

```text
1. 기준은 모든 고객이므로 `customers`에서 시작
2. 주문이 없을 수도 있으므로 `LEFT JOIN orders`
3. 고객별로 묶고 주문번호 개수 세기
```

#### 정답 코드


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


### S07. 주문 수가 2건 이상인 고객

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 주문을 2건 이상 한 고객만 찾는다. |
| SQL 패턴 | JOIN + GROUP BY + HAVING |
| 필요한 테이블 | `customers`, `orders` |
| 연결 흐름 | `customers.customerNumber` → `orders.customerNumber` |
| 자주 틀리는 부분 | `COUNT()` 같은 집계 결과 조건은 `WHERE`가 아니라 `HAVING`이다. |
| 필요한 컬럼/계산 | `customers.customerName`<br>`COUNT(orders.orderNumber) AS order_count` |

#### 읽는 순서

```text
1. 고객과 주문을 연결
2. 고객별로 묶어 주문 수 계산
3. 집계 결과가 2 이상인 그룹만 `HAVING`으로 남김
```

#### 정답 코드


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


### S08. 주문한 적 있는 고객만 조회 - IN 서브쿼리

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | orders에 등장한 적 있는 고객만 조회한다. |
| SQL 패턴 | IN 서브쿼리 |
| 필요한 테이블 | `customers`, `orders` |
| 연결 흐름 | `customers.customerNumber IN (orders.customerNumber 목록)` |
| 자주 틀리는 부분 | `IN`은 “목록 안에 있냐”라는 뜻이다. 서브쿼리 결과가 여러 행이어도 된다. |
| 필요한 컬럼/계산 | `customers.customerNumber`<br>`customers.customerName`<br>`customers.country`<br>서브쿼리: `orders.customerNumber` |

#### 읽는 순서

```text
1. 메인 쿼리는 고객 테이블 조회
2. 서브쿼리에서 주문한 고객번호 목록 생성
3. 고객번호가 그 목록 안에 있는 고객만 남김
```

#### 정답 코드


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


### S09. 주문이 없는 고객 찾기 - LEFT JOIN

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 주문 기록이 없는 고객을 찾는다. |
| SQL 패턴 | LEFT JOIN + IS NULL |
| 필요한 테이블 | `customers`, `orders` |
| 연결 흐름 | `customers`를 기준으로 `orders`를 붙인 뒤 주문번호가 NULL인 행 찾기 |
| 자주 틀리는 부분 | `= NULL`이 아니라 `IS NULL`을 써야 한다. |
| 필요한 컬럼/계산 | `customers.customerNumber`<br>`customers.customerName`<br>`customers.country`<br>`orders.orderNumber` |

#### 읽는 순서

```text
1. 모든 고객을 살리기 위해 `LEFT JOIN`
2. 주문이 없으면 orders 쪽 컬럼이 NULL
3. `WHERE o.orderNumber IS NULL`로 주문 없는 고객만 필터링
```

#### 정답 코드


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


### S10. 고객 신용한도 순위 - RANK

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 신용한도 기준으로 고객 순위를 매긴다. |
| SQL 패턴 | 윈도우 함수 RANK OVER |
| 필요한 테이블 | `customers` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | 윈도우 함수는 `GROUP BY`처럼 행을 줄이지 않는다. 순위 컬럼을 옆에 붙인다. |
| 필요한 컬럼/계산 | `customerName`<br>`country`<br>`creditLimit`<br>`RANK() OVER (ORDER BY creditLimit DESC)` |

#### 읽는 순서

```text
1. 신용한도는 customers에 있음
2. `ORDER BY creditLimit DESC`로 큰 값이 1등
3. `RANK() OVER (...)`로 원래 행을 유지한 채 순위 컬럼 추가
```

#### 정답 코드


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


### S11. 상품라인별 주문수량 합계

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 상품라인별로 총 몇 개가 주문되었는지 구한다. |
| SQL 패턴 | products + orderdetails JOIN + SUM |
| 필요한 테이블 | `products`, `orderdetails` |
| 연결 흐름 | `products.productCode` → `orderdetails.productCode` |
| 자주 틀리는 부분 | 상품명/라인은 products, 실제 주문수량은 orderdetails에 있다. |
| 필요한 컬럼/계산 | `products.productLine`<br>`orderdetails.quantityOrdered` |

#### 읽는 순서

```text
1. 상품라인은 products에 있음
2. 주문수량은 orderdetails에 있음
3. 둘은 productCode로 연결
4. productLine별로 묶고 수량 합계
```

#### 정답 코드


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


### S12. 주문별 총액

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 각 주문번호별 총 주문 금액을 구한다. |
| SQL 패턴 | GROUP BY + SUM 계산식 |
| 필요한 테이블 | `orderdetails` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | 주문 하나에 상품이 여러 줄일 수 있으므로 `GROUP BY orderNumber`가 필요하다. |
| 필요한 컬럼/계산 | `orderNumber`<br>`quantityOrdered * priceEach` |

#### 읽는 순서

```text
1. 주문 상세에는 한 주문 안의 여러 상품 행이 있음
2. 각 행 금액은 수량×단가
3. 주문번호별로 묶고 합계
```

#### 정답 코드


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


### S13. 평균 주문총액보다 큰 주문

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 주문별 총액을 먼저 만든 뒤, 평균 주문총액보다 큰 주문만 찾는다. |
| SQL 패턴 | CTE + 평균 서브쿼리 |
| 필요한 테이블 | `orderdetails` |
| 연결 흐름 | 1단계 order_totals 생성 → 2단계 평균보다 큰 주문 필터링 |
| 자주 틀리는 부분 | 한 번에 하려 하지 말고 “주문별 총액 표를 먼저 만든다”고 생각한다. |
| 필요한 컬럼/계산 | CTE: `orderNumber`, `order_total`<br>조건: `order_total > AVG(order_total)` |

#### 읽는 순서

```text
1. CTE에서 주문별 총액 계산
2. 바깥 SELECT에서 CTE를 조회
3. 서브쿼리로 CTE의 평균 주문총액 계산
4. 그 평균보다 큰 주문만 남김
```

#### 정답 코드


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


### S14. 제품별 판매액

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 제품별 총 판매액을 구한다. |
| SQL 패턴 | products + orderdetails JOIN + SUM |
| 필요한 테이블 | `products`, `orderdetails` |
| 연결 흐름 | `products.productCode` → `orderdetails.productCode` |
| 자주 틀리는 부분 | `productName`만 GROUP BY하면 제품 코드가 다른 동명이 있을 수 있어 `productCode`도 같이 묶는 게 안전하다. |
| 필요한 컬럼/계산 | `products.productName`<br>`products.productLine`<br>`orderdetails.quantityOrdered * orderdetails.priceEach` |

#### 읽는 순서

```text
1. 제품명/제품라인은 products
2. 판매 금액은 orderdetails의 수량×가격
3. productCode로 연결
4. 제품별로 묶고 합계
```

#### 정답 코드


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


### S15. 주문 상태별 주문 수

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 주문 상태별로 주문이 몇 건인지 센다. |
| SQL 패턴 | GROUP BY + COUNT |
| 필요한 테이블 | `orders` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | 상태별 개수는 대표적인 `GROUP BY status + COUNT(*)` 패턴이다. |
| 필요한 컬럼/계산 | `status`<br>`COUNT(*) AS order_count` |

#### 읽는 순서

```text
1. 주문 상태는 orders.status
2. 상태별로 묶음
3. 각 상태의 행 수를 셈
```

#### 정답 코드


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


### S16. CASE로 신용한도 등급

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 신용한도에 따라 high/mid/low 등급을 붙인다. |
| SQL 패턴 | CASE WHEN 조건 분류 |
| 필요한 테이블 | `customers` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | CASE는 위에서 아래로 검사한다. 100000 조건을 먼저 써야 high가 mid로 빠지지 않는다. |
| 필요한 컬럼/계산 | `customerName`<br>`creditLimit`<br>`CASE WHEN ... END AS credit_group` |

#### 읽는 순서

```text
1. 신용한도는 customers.creditLimit
2. 100000 이상이면 high
3. 70000 이상이면 mid
4. 나머지는 low
```

#### 정답 코드


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


### S17. NTILE로 4구간 나누기

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 신용한도 기준으로 고객을 4개 구간으로 나눈다. |
| SQL 패턴 | NTILE(4) OVER |
| 필요한 테이블 | `customers` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | `NTILE(4)`는 4등분 구간이다. 순위 `RANK()`와 다르다. |
| 필요한 컬럼/계산 | `customerName`<br>`creditLimit`<br>`NTILE(4) OVER (ORDER BY creditLimit DESC)` |

#### 읽는 순서

```text
1. 신용한도 큰 순서로 정렬
2. `NTILE(4)`로 4개 구간 부여
3. 구간 번호와 신용한도 순으로 정렬
```

#### 정답 코드


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


### S18. 제품별 평균 주문 단가

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 제품코드별 평균 주문 단가를 구한다. |
| SQL 패턴 | GROUP BY + AVG |
| 필요한 테이블 | `orderdetails` |
| 연결 흐름 | 연결 없음 |
| 자주 틀리는 부분 | 제품명까지 필요하면 products와 JOIN하지만, 이 문제는 productCode만 요구하므로 orderdetails만으로 가능하다. |
| 필요한 컬럼/계산 | `productCode`<br>`priceEach`<br>`ROUND(AVG(priceEach), 2)` |

#### 읽는 순서

```text
1. 주문 상세에는 제품코드와 실제 주문 단가가 있음
2. 제품코드별로 묶기
3. 평균 priceEach 계산
```

#### 정답 코드


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


### S19. CTE로 국가별 매출 TOP 3

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 국가별 매출 표를 먼저 만든 뒤 상위 3개 국가만 출력한다. |
| SQL 패턴 | CTE + 3테이블 JOIN + TOP N |
| 필요한 테이블 | `customers`, `orders`, `orderdetails` |
| 연결 흐름 | `customers` → `orders` → `orderdetails`, 이후 CTE 결과에서 TOP 3 |
| 자주 틀리는 부분 | S19는 새로운 문제가 아니라 S05를 `WITH country_sales AS (...)`로 감싼 버전이다. |
| 필요한 컬럼/계산 | CTE 내부: `country`, `SUM(quantityOrdered * priceEach)`<br>최종: `ORDER BY total_sales DESC LIMIT 3` |

#### 읽는 순서

```text
1. S05와 같은 국가별 매출 계산을 CTE로 만든다
2. 바깥 SELECT에서 CTE를 조회한다
3. 매출 내림차순으로 정렬하고 3개만 출력
```

#### 정답 코드


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


### S20. UNION ALL로 연도별 주문 고객 합치기

| 구분 | 내용 |
|---|---|
| 문제에서 요구한 것 | 2003년 주문 고객번호와 2004년 주문 고객번호를 year_label과 함께 한 표로 합친다. |
| SQL 패턴 | UNION ALL |
| 필요한 테이블 | `orders` |
| 연결 흐름 | 2003 SELECT 결과 + 2004 SELECT 결과를 세로로 붙이기 |
| 자주 틀리는 부분 | UNION 계열은 양쪽 SELECT의 컬럼 개수와 순서가 같아야 한다. |
| 필요한 컬럼/계산 | 고정값: `'2003' AS year_label`, `'2004' AS year_label`<br>`orders.customerNumber`<br>`orders.orderDate` |

#### 읽는 순서

```text
1. 첫 번째 SELECT에서 2003년 주문 고객 추출
2. 두 번째 SELECT에서 2004년 주문 고객 추출
3. `UNION ALL`로 두 결과를 그대로 합침
```

#### 정답 코드


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
