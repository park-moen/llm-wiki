# Original versus Current Pricing

> Source: https://docs.oracle.com/cd/E35319_01/Platform.10-2/ATGCommProgGuide/html/s2207originalversuscurrentpricing01.html
> Collected: 2026-09-28
> Published: Unknown
> Scope: Selected sections from the original page; original wording retained.

Original versus Current Pricing
An item’s original price instead of the current day price is used in the following situations:
When modifying a previously submitted order – This allows the product or SKU prices to remain consistent with the prices that were applied when the order was originally submitted. Original pricing is applied only towards items that were in the original order. Any new products or SKUs that are added to the order will use the current day pricing
When pricing items in an exchange order – This allows the product or SKU prices in the exchange order to remain consistent with the prices that were applied to the original order. As with previously submitted orders, the original pricing is applied only towards items that were in the original order. Any new products or SKUs that are added to the exchange will be priced using the current day pricing
The ItemPriceSource object allows you to provide pricing information based upon a specific Product/SKU combination. The override is then used in place of the default current day pricing.
This feature also provides the ability to generate a set of ItemPriceSource objects from the pricing information stored for the items within an order. This generates the source objects for pricing previously submitted orders and exchange orders by generating the source objects from the submitted order and the original purchase order.
