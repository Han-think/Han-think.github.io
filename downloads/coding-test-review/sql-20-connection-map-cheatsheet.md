# SQL 20문제 한눈에 보는 테이블 연결표

| 문제 | 패턴 | 필요한 테이블 | 연결 흐름 | 핵심 계산/조건 |
|---|---|---|---|---|
| S01 customers 전체 조회 | 단일 테이블 전체 조회 | `customers` | 연결 없음 | 전체 컬럼 `*` |
| S02 국가별 고객 수 | GROUP BY + COUNT | `customers` | 연결 없음 | `country`, `COUNT(*) AS customer_count` |
| S03 2004년 주문일별 주문 수 | WHERE + strftime + GROUP BY | `orders` | 연결 없음 | `orderDate`, `COUNT(*) AS order_count_2004` |
| S04 제품 라인별 평균 MSRP | GROUP BY + AVG + ROUND | `products` | 연결 없음 | `productLine`, `MSRP` |
| S05 국가별 총 주문 금액 | 3테이블 JOIN + GROUP BY + SUM | `customers`, `orders`, `orderdetails` | `customers.customerNumber` → `orders.customerNumber` → `orders.orderNumber` → `orderdetails.orderNumber` | 나라: `customers.country`, 주문 연결: `orders.customerNumber`, `orders.orderNumber` |
| S06 고객별 주문 수 | LEFT JOIN + COUNT | `customers`, `orders` | `customers.customerNumber` → `orders.customerNumber` | `customers.customerName`, `orders.orderNumber` |
| S07 주문 수가 2건 이상인 고객 | JOIN + GROUP BY + HAVING | `customers`, `orders` | `customers.customerNumber` → `orders.customerNumber` | `customers.customerName`, `COUNT(orders.orderNumber) AS order_count` |
| S08 주문한 적 있는 고객만 조회 - IN 서브쿼리 | IN 서브쿼리 | `customers`, `orders` | `customers.customerNumber IN (orders.customerNumber 목록)` | `customers.customerNumber`, `customers.customerName` |
| S09 주문이 없는 고객 찾기 - LEFT JOIN | LEFT JOIN + IS NULL | `customers`, `orders` | `customers`를 기준으로 `orders`를 붙인 뒤 주문번호가 NULL인 행 찾기 | `customers.customerNumber`, `customers.customerName` |
| S10 고객 신용한도 순위 - RANK | 윈도우 함수 RANK OVER | `customers` | 연결 없음 | `customerName`, `country` |
| S11 상품라인별 주문수량 합계 | products + orderdetails JOIN + SUM | `products`, `orderdetails` | `products.productCode` → `orderdetails.productCode` | `products.productLine`, `orderdetails.quantityOrdered` |
| S12 주문별 총액 | GROUP BY + SUM 계산식 | `orderdetails` | 연결 없음 | `orderNumber`, `quantityOrdered * priceEach` |
| S13 평균 주문총액보다 큰 주문 | CTE + 평균 서브쿼리 | `orderdetails` | 1단계 order_totals 생성 → 2단계 평균보다 큰 주문 필터링 | CTE: `orderNumber`, `order_total`, 조건: `order_total > AVG(order_total)` |
| S14 제품별 판매액 | products + orderdetails JOIN + SUM | `products`, `orderdetails` | `products.productCode` → `orderdetails.productCode` | `products.productName`, `products.productLine` |
| S15 주문 상태별 주문 수 | GROUP BY + COUNT | `orders` | 연결 없음 | `status`, `COUNT(*) AS order_count` |
| S16 CASE로 신용한도 등급 | CASE WHEN 조건 분류 | `customers` | 연결 없음 | `customerName`, `creditLimit` |
| S17 NTILE로 4구간 나누기 | NTILE(4) OVER | `customers` | 연결 없음 | `customerName`, `creditLimit` |
| S18 제품별 평균 주문 단가 | GROUP BY + AVG | `orderdetails` | 연결 없음 | `productCode`, `priceEach` |
| S19 CTE로 국가별 매출 TOP 3 | CTE + 3테이블 JOIN + TOP N | `customers`, `orders`, `orderdetails` | `customers` → `orders` → `orderdetails`, 이후 CTE 결과에서 TOP 3 | CTE 내부: `country`, `SUM(quantityOrdered * priceEach)`, 최종: `ORDER BY total_sales DESC LIMIT 3` |
| S20 UNION ALL로 연도별 주문 고객 합치기 | UNION ALL | `orders` | 2003 SELECT 결과 + 2004 SELECT 결과를 세로로 붙이기 | 고정값: `'2003' AS year_label`, `'2004' AS year_label`, `orders.customerNumber` |