# MarketplacePulse AI — Data Dictionary (Olist Dataset)

Phase 1 deliverable. Every column across all 9 tables: meaning, dtype, and
data-quality notes from Modules 2–7.

---

## 1. customers (99,441 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| customer_id | string (PK) | Unique ID per order-customer record | One per order, not per person — see open point in ER diagram |
| customer_unique_id | string | Identifies the actual person across orders | Uniqueness vs customer_id not yet verified |
| customer_zip_code_prefix | int | First digits of customer's zip code | 278 values not found in geolocation table |
| customer_city | string | Customer's city | — |
| customer_state | string | Customer's state (2-letter code) | — |

---

## 2. orders (99,441 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| order_id | string (PK) | Unique order identifier | Confirmed unique, 99,441 values |
| customer_id | string (FK) | Links to customers | FK clean, 0 orphans |
| order_status | string | Order lifecycle stage | 8 categories: delivered (96,478), shipped (1,107), canceled (625), unavailable (609), invoiced (314), processing (301), created (5), approved (2) |
| order_purchase_timestamp | datetime | When the order was placed | Converted from string in Module 3 |
| order_approved_at | datetime | When payment was approved | 0.16% missing |
| order_delivered_carrier_date | datetime | When handed to the carrier | 1.79% missing; 1,359 rows show this before order_approved_at |
| order_delivered_customer_date | datetime | When customer received it | 2.98% missing; 23 rows show this before delivered_carrier_date |
| order_estimated_delivery_date | datetime | Promised delivery date | Always midnight; compare at date level, not timestamp level |

---

## 3. order_items (112,650 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| order_id | string (PK part, FK) | Links to orders | FK clean, 0 orphans |
| order_item_id | int (PK part) | Item sequence number within the order | Together with order_id forms the row grain |
| product_id | string (FK) | Links to products | FK clean, 0 orphans |
| seller_id | string (FK) | Links to sellers | FK clean, 0 orphans |
| shipping_limit_date | datetime | Seller's shipping deadline | Not yet converted from string — do in Phase 2 if needed |
| price | float | Item price | No negatives, no zeros |
| freight_value | float | Shipping cost for this item | 383 rows are exactly 0 (possible free shipping) |

---

## 4. payments (103,886 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| order_id | string (PK part, FK) | Links to orders | FK clean; 1 order has no payment row at all |
| payment_sequential | int (PK part) | Sequence if an order has multiple payments | Grain is 1:N — 96,479 orders have 1 row, up to 29 for one order |
| payment_type | string | Payment method | credit_card (76,795), boleto (19,784), voucher (5,775), debit_card (1,529), not_defined (3) |
| payment_installments | int | Number of installments | 2 rows have 0 installments |
| payment_value | float | Amount paid in this row | 9 rows are exactly 0 (6 voucher, 3 not_defined); no negatives |

---

## 5. reviews (99,224 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| review_id | string | Review identifier | Uniqueness not yet verified |
| order_id | string (FK) | Links to orders | FK clean; ~768 orders have no review |
| review_score | int (1–5) | Customer satisfaction rating | Mean 4.09, skewed positive |
| review_comment_title | string | Optional short title | 88.34% missing — expected, optional field |
| review_comment_message | string | Optional free-text comment | 58.70% missing — expected, optional field |
| review_creation_date | datetime | When the review was requested | Not yet converted from string |
| review_answer_timestamp | datetime | When the customer submitted it | Not yet converted from string |

---

## 6. products (32,951 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| product_id | string (PK) | Unique product identifier | FK target, clean |
| product_category_name | string (FK) | Category, in Portuguese | 610 products (1.85%) have none; 2 categories missing from translation table |
| product_name_lenght | float | Character length of product name | 1.85% missing (same ~610 rows as category) |
| product_description_lenght | float | Character length of description | 1.85% missing |
| product_photos_qty | float | Number of photos listed | 1.85% missing |
| product_weight_g | float | Product weight in grams | 4 rows are exactly 0 — not physically valid, needs cleaning |
| product_length_cm | float | Package length | 0.01% missing |
| product_height_cm | float | Package height | 0.01% missing |
| product_width_cm | float | Package width | 0.01% missing |

---

## 7. sellers (3,095 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| seller_id | string (PK) | Unique seller identifier | FK target, clean |
| seller_zip_code_prefix | int | First digits of seller's zip code | 7 values not found in geolocation table |
| seller_city | string | Seller's city | — |
| seller_state | string | Seller's state (2-letter code) | — |

---

## 8. geolocation (1,000,163 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| geolocation_zip_code_prefix | int | Zip code prefix (not unique — many rows per prefix) | 19,015 unique prefixes; median 29 rows per prefix, max 1,146 |
| geolocation_lat | float | Latitude | 42 rows outside Brazil's bounds (-34 to 5) |
| geolocation_lng | float | Longitude | 42 rows outside Brazil's bounds (-74 to -34) |
| geolocation_city | string | City name | — |
| geolocation_state | string | State (2-letter code) | — |

No primary key — this table has 261,831 duplicate rows (26.18%) and must be deduplicated and aggregated (mean lat/lng per zip prefix) before joining to customers or sellers.

---

## 9. product_category_translation (71 rows)

| Column | Type | Description | Notes |
|---|---|---|---|
| product_category_name | string (PK) | Category name in Portuguese | All 71 entries are used by at least one product |
| product_category_name_english | string | English translation | 2 product categories (pc_gamer, portateis_cozinha_e_preparadores_de_alimentos) have no matching row here |

---

## Summary of known data-quality issues (carried into Phase 2)

1. Geolocation: 261,831 duplicate rows; 42 rows with coordinates outside Brazil
2. Orders: 1,359 "shipped before approved", 23 "delivered before shipped", 8 "delivered" with no delivery date, 6 non-delivered with a delivery date
3. Products: 610 missing category name; 2 categories missing a translation; 4 rows with 0 weight
4. Payments: 9 rows with 0 payment_value; 3 rows with payment_type "not_defined"; 2 rows with 0 installments
5. Coverage gaps: 278 customer zips and 7 seller zips missing from geolocation; 1 order with no payment row; ~768 orders with no review
6. Not yet verified: customer_id vs customer_unique_id relationship; review_id uniqueness; shipping_limit_date, review_creation_date, review_answer_timestamp still stored as strings
