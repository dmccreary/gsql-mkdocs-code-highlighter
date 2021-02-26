# SQL Test

```sql
SELECT p.name      AS product,
       p.listprice AS 'List Price',
       p.discount  AS 'discount' 
FROM   production.product p 
       JOIN production.productsubcategory s ON p.productsubcategoryid = s.productsubcategoryid 
WHERE  s.name LIKE @product 
       AND p.listprice < @maxprice;
```