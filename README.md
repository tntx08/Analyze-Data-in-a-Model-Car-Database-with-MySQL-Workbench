**Inventory Analysis**

_Exploring Current Inventory_

1. Unique Product Count in Inventory
   
   SELECT COUNT(DISTINCT productName) FROM products;
   
   - Findings: The inventory currently contains 110 unique products.

2. Product Category Counts
   
   SELECT COUNT(productLine) AS total_products, productLine FROM products GROUP BY productLine;
   
   - Findings: The inventory is divided into 7 distinct product categories.

3. Total Quantity of Each Product
   
   SELECT SUM(quantityInStock), productName FROM products GROUP BY productName;
   

4. Highest and Lowest Stocked Products
   - Highest Quantity:
     
     SELECT MAX(quantityInStock) AS highest_quantity, productName FROM products GROUP BY productName LIMIT 1;
     
   - Lowest Quantity:
     
     SELECT MIN(quantityInStock) AS lowest_quantity, productName FROM products GROUP BY productName ORDER BY lowest_quantity ASC LIMIT 1;
     
   - Findings: "1969 Harley Davidson Ultimate Chopper" has the highest stock, while "1960 BSA Gold Star DBD34" has the lowest stock.

_Inventory Reorganization and Reduction_

5. Current Storage Locations of Products
   
   SELECT p.productName, p.quantityInStock, w.warehouseCode, w.warehouseName FROM products p JOIN warehouses w ON p.warehouseCode = w.warehouseCode;
   

6. Products Stored in the Same Locations
   
   SELECT productName, warehouseCode FROM products ORDER BY warehouseCode;
   

7. Identifying Low Quantity Products for Consolidation
   
   SELECT productCode, productName, warehouseCode, SUM(quantityInStock) AS total_quantity FROM products GROUP BY productCode, warehouseCode ORDER BY total_quantity, warehouseCode ASC;
   
   - Insight: Products with minimal stock (e.g., "1960 BSA Gold Star DBD34" with 15 units in warehouse A and 68 units in warehouse B) could be consolidated for efficiency.

_Identifying Slow-Moving Products_

8. Optimizing Warehouse Usage by Rearranging Products
   
   SELECT p.productCode, p.productName, SUM(p.quantityInStock) AS total_quantity, SUM(o.quantityOrdered) AS total_sales, warehouseCode FROM orderdetails o JOIN products p ON p.productCode = o.productCode GROUP BY p.productCode ORDER BY total_sales ASC;
   

9. Products Exclusively Stored in Warehouse D
   
   SELECT DISTINCT p.productName FROM products p INNER JOIN orderdetails o ON p.productCode = o.productCode WHERE p.warehouseCode = 'D' AND p.productCode NOT IN (SELECT DISTINCT p2.productCode FROM products p2 INNER JOIN orderdetails o2 ON p2.productCode = o2.productCode WHERE p2.warehouseCode IN ('A', 'B', 'C'));
   

10. Products Not Sold in the Last 300 Days
    
    SELECT o.orderDate, od.quantityOrdered, p.productName, p.productCode, p.quantityInStock, p.warehouseCode FROM orders o INNER JOIN orderdetails od ON o.orderNumber = od.orderNumber INNER JOIN products p ON p.productCode = od.productCode WHERE o.orderDate IS NULL OR o.orderDate < DATE_SUB(NOW(), INTERVAL 300 DAY);
    

11. Identifying Low Turnover Products for Reevaluation
    
    SELECT SUM(py.amount) AS total_amount, SUM(od.quantityOrdered) AS total_orders, p.productName, p.warehouseCode, SUM(p.quantityInStock) FROM payments py JOIN orders o ON py.customerNumber = o.customerNumber JOIN orderdetails od ON o.orderNumber = od.orderNumber JOIN products p ON od.productCode = p.productCode GROUP BY p.productName, p.warehouseCode ORDER BY p.warehouseCode, total_amount ASC;
    
    - Findings: In Warehouse A, the "1960 BSA Gold Star DBD34" has low orders and revenue; similarly, "1972 Alfa Romeo GTA" in Warehouse B has low revenue, while "1941 Chevrolet Special Deluxe Cabriolet" in Warehouse C has low sales despite high stock.

12. High-Demand, Low-Stock Products
    
    SELECT SUM(py.amount) AS total_amount, SUM(od.quantityOrdered) AS total_orders, p.productName, p.warehouseCode, SUM(p.quantityInStock) FROM payments py JOIN orders o ON py.customerNumber = o.customerNumber JOIN orderdetails od ON o.orderNumber = od.orderNumber JOIN products p ON od.productCode = p.productCode GROUP BY p.productName, p.warehouseCode ORDER BY p.warehouseCode, total_amount ASC;
    
    - Findings: In Warehouse C, the "1928 Ford Phaeton Deluxe" needs increased stock, as does the "1968 Ford Mustang" in Warehouse B and "The Mayflower" in Warehouse D.

13. Products Not Sold Within the Last Year
    
    SELECT p.productName, p.warehouseCode, o.orderDate FROM products p LEFT JOIN orderdetails od ON p.productCode = od.productCode JOIN orders o ON o.orderNumber = od.orderNumber WHERE o.orderDate IS NULL OR (o.orderDate >= '2004-01-01' AND o.orderDate <= '2005-12-01') ORDER BY p.warehouseCode;
    
    - Findings: Identifying items unsold in the last year may highlight products that require additional promotional strategies or discontinuation.


_Analyzing Supplier Relationships_

14. Top Customers by Quantity Ordered
    
    SELECT c.customerName, c.customerNumber, c.country, SUM(od.quantityOrdered) AS total_order FROM customers c JOIN orders o ON c.customerNumber = o.customerNumber JOIN orderdetails od ON od.orderNumber = o.orderNumber GROUP BY c.customerNumber, c.country ORDER BY total_order DESC LIMIT 10;
    

15. Total Unique Customers
    
    SELECT COUNT(DISTINCT customerName) AS total_unique_customers FROM customers;
    
    - Findings: The inventory serves 122 unique customers.

16. Top 10 Customers by Orders and Warehouse
    
    SELECT c.customerName, c.customerNumber, c.country, SUM(od.quantityOrdered) AS total_order, p.warehouseCode FROM customers c JOIN orders o ON c.customerNumber = o.customerNumber JOIN orderdetails od ON od.orderNumber = o.orderNumber JOIN products p ON p.productCode = od.productCode GROUP BY c.customerNumber, c.country, p.warehouseCode ORDER BY total_order DESC LIMIT 10;
    

17. Sales Data by Warehouse
    
    SELECT SUM(od.quantityOrdered) AS total_order, p.warehouseCode FROM orderdetails od JOIN products p ON p.productCode = od.productCode GROUP BY p.warehouseCode ORDER BY p.warehouseCode;
    
    - Findings: Warehouse A has 24,650 orders, Warehouse B has 35,582 orders, and Warehouse D has the fewest orders.

18. High-Demand Vendors by Warehouse
    
    SELECT p.productVendor, SUM(od.quantityOrdered) AS total_order, p.warehouseCode FROM orderdetails od JOIN products p ON p.productCode = od.productCode GROUP BY p.productVendor, p.warehouseCode ORDER BY p.warehouseCode DESC;
    

19. Top 10 Low-Performing Vendors
    
    SELECT p.productVendor, SUM(od.quantityOrdered) AS total_order FROM orderdetails od JOIN products p ON p.productCode = od.productCode GROUP BY p.productVendor ORDER BY total_order ASC LIMIT 10;
    

20. Top 50 Customers by Warehouse and Orders
    
    SELECT c.customerName, c.customerNumber, c.country, COUNT(c.country) AS no_of_customer, SUM(od.quantityOrdered) AS total_order, p.warehouseCode, p.productName FROM customers c JOIN orders o ON c.customerNumber = o.customerNumber JOIN orderdetails od ON od.orderNumber = o.orderNumber JOIN products p ON p.productCode = od.productCode GROUP BY c.customerNumber, c.city, c.country, warehouseCode, p.productName ORDER BY no_of_customer DESC LIMIT 50;
    

21. Low-Selling Products by Warehouse
    
    SELECT c.customerName, c.customerNumber, c.country, COUNT(c.country) AS no_of_customer, SUM(od.quantityOrdered) AS total_order, p.warehouseCode, p.productName FROM customers c JOIN orders o ON c.customerNumber = o.customerNumber JOIN orderdetails od ON od.orderNumber = o.orderNumber JOIN products p ON p.productCode = od.productCode GROUP BY c.customerNumber, c.city, c.country, p.warehouseCode, p.productName ORDER BY p.warehouseCode DESC;

**Key Insights and Recommendations**

1. Inventory Summary: There are currently 110 unique products across 7 categories.
2. Stock Balancing: Products such as "F/A 18 Hornet 1/72" have significant stock but low sales. Consider reducing stock or improving marketing for such items.
3. Consolidation Needs: Low-quantity items should be consolidated within fewer locations to optimize storage.
4. Customer Insights: Top customers and their locations may indicate areas to consider for opening new warehouses or focusing on specific customer groups with targeted promotions.
5. Supplier Review: Vendors with low-performing products should be evaluated, with possible adjustments in stock allocations or vendor replacements.
