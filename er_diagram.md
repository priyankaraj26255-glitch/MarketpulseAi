# MarketplacePulse AI — ER Diagram (Olist Dataset)

Phase 1 deliverable. Shows the 9 Olist tables, their keys, and how they connect.
Renders as a picture on GitHub. In VS Code, use a Mermaid preview extension.

## Diagram

```mermaid
erDiagram
    customers ||--o{ orders : places
    orders ||--o{ order_items : contains
    orders ||--o{ payments : "paid by"
    orders ||--o{ reviews : receives
    products ||--o{ order_items : "sold as"
    sellers ||--o{ order_items : fulfils
    product_category_translation ||--o{ products : translates
    geolocation ||--o{ customers : "zip lookup"
    geolocation ||--o{ sellers : "zip lookup"

    customers {
        string customer_id PK
        string customer_unique_id
        int customer_zip_code_prefix FK
        string customer_city
        string customer_state
    }

    orders {
        string order_id PK
        string customer_id FK
        string order_status
        datetime order_purchase_timestamp
        datetime order_approved_at
        datetime order_delivered_carrier_date
        datetime order_delivered_customer_date
        datetime order_estimated_delivery_date
    }

    order_items {
        string order_id PK, FK
        int order_item_id PK
        string product_id FK
        string seller_id FK
        datetime shipping_limit_date
        float price
        float freight_value
    }

    payments {
        string order_id PK, FK
        int payment_sequential PK
        string payment_type
        int payment_installments
        float payment_value
    }

    reviews {
        string review_id
        string order_id FK
        int review_score
        string review_comment_title
        string review_comment_message
        datetime review_creation_date
        datetime review_answer_timestamp
    }

    products {
        string product_id PK
        string product_category_name FK
        int product_name_lenght
        int product_description_lenght
        int product_photos_qty
        float product_weight_g
        float product_length_cm
        float product_height_cm
        float product_width_cm
    }

    sellers {
        string seller_id PK
        int seller_zip_code_prefix FK
        string seller_city
        string seller_state
    }

    geolocation {
        int geolocation_zip_code_prefix
        float geolocation_lat
        float geolocation_lng
        string geolocation_city
        string geolocation_state
    }

    product_category_translation {
        string product_category_name PK
        string product_category_name_english
    }
```

## Relationships and what we verified in the notebook

| Parent | Child | Join key | Verified in Phase 1 |
|---|---|---|---|
| customers | orders | customer_id | FK check clean (0 orphans) |
| orders | order_items | order_id | FK check clean (0 orphans) |
| orders | payments | order_id | FK clean; 1:N confirmed (99,440 orders, 103,886 rows); 1 order has no payment |
| orders | reviews | order_id | FK clean; 1:N confirmed (98,673 orders, 99,224 rows); ~768 orders have no review |
| products | order_items | product_id | FK check clean (0 orphans) |
| sellers | order_items | seller_id | FK check clean (0 orphans) |
| product_category_translation | products | product_category_name | 2 categories missing from translation; 610 products have no category |
| geolocation | customers | zip code prefix | Soft link only; 278 customer zips not found in geolocation |
| geolocation | sellers | zip code prefix | Soft link only; 7 seller zips not found in geolocation |

## Grain (what one row means)

| Table | One row = | Rows |
|---|---|---|
| customers | one customer_id | 99,441 |
| orders | one order | 99,441 |
| order_items | one item within an order (order_id + order_item_id) | 112,650 |
| payments | one payment entry for an order (order_id + payment_sequential) | 103,886 |
| reviews | one review, an order can have more than one | 99,224 |
| products | one product | 32,951 |
| sellers | one seller | 3,095 |
| geolocation | one coordinate point, many per zip prefix (no key) | 1,000,163 |
| product_category_translation | one category name | 71 |

## Notes and open points

- **Geolocation is not keyed.** It has 19,015 unique zip prefixes over about 1M rows, so it must be aggregated per zip prefix before joining to customers or sellers, otherwise rows multiply.
- **customer_id vs customer_unique_id.** In the Olist documentation, customer_id is assigned per order and customer_unique_id identifies the actual person. Not yet verified in our notebook. Check this before building customer recency and frequency features in Phase 4.
- **review_id uniqueness** has not been verified yet, so it is not marked as a primary key in the diagram.
- Primary keys marked above follow the Olist schema. Uniqueness was directly confirmed for orders.order_id (99,441 unique values) and through the clean FK checks.
