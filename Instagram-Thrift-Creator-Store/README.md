# Instagram Thrift Creator Store - DB Schema Summary

## Entities & Attributes

| Entity | Primary Key | Key Attributes | Description |
|---|---|---|---|
| `customer` | `id` | `name`, `phone` (unique), `email` (unique), `instagram_handle` (unique), `address` | Stores buyer details used for placing orders and delivery contact. |
| `products` | `id` | `name`, `description`, `product_type` (`thrifted`,`handmade`), `category` (`Tops`,`Bottoms`,`Accessories`,`Home`,`Jewelry`), `price`, `size` (`XS`,`S`,`M`,`L`,`XL`,`Free_Size`), `colour`, `condition`, `image_url`, `video_url`, `isActive` | Catalog of items that will be sold in the store, including media and availability status. |
| `inventory` | `id` | `productId` (FK), `quantity_available` (default `1`), `last_updated` (default `now()`) | Tracks current stock count for each product and last stock update time. |
| `orders` | `id` | `customer_id` (FK), `order_date`, `total_amount`, `rating` | Stores each purchase transaction created by a customer. |
| `order_items` | `id` | `orderId` (FK), `productId` (FK), `quantity` (default `1`), `unit_price` | Line-item breakdown of products included in an order. |
| `payments` | `id` | `orderId` (FK), `payment_method` (`UPI`,`Card`,`COD`), `amount_paid`, `payment_status` (`Pending`,`Paid`,`Failed`), `payment_date` (default `now()`) | Captures payment details and status for a specific order. |
| `shipping` | `id` | `orderId` (FK), `shipping_address`, `shipping_status` (`confirmed`,`out_for_delivery`,`delivered`) | Tracks shipping address and fulfillment progress for an order. |

## Table Relationships

| Parent Table | Child Table | Relationship | FK Column | Description |
|---|---|---|---|---|
| `customer` | `orders` | 1 : many | `orders.customer_id` | One customer can place multiple orders over time. |
| `orders` | `order_items` | 1 : many | `order_items.orderId` | One order can contain multiple items in one go. |
| `products` | `order_items` | 1 : many | `order_items.productId` | One product can appear in many order line items. |
| `products` | `inventory` | 1 : 1 | `inventory.productId` | Each product maps to a single inventory tracking record. |
| `orders` | `payments` | 1 : 1 | `payments.orderId` | Each order has exactly one payment record. |
| `orders` | `shipping` | 1 : 1 | `shipping.orderId` | Each order has one shipping record for delivery tracking. |
