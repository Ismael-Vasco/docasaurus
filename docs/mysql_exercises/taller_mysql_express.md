# Taller de Consultas SQL por Niveles

---

## Nivel 1: Consultas Básicas y Relaciones Directas (2 Tablas)

1. Listar el nombre de un usuario (el que tu quieras), su correo electrónico y el código (`order_number`) de todos los pedidos que han realizado.

```js
router.get('/ejercicio-1/:userId', async (req, res) => {
    try {
        const { userId } = req.params;

        const [rows] = await pool.query(`
            SELECT u.name, u.email, o.order_number
            FROM users u
            JOIN orders o ON u.id = o.user_id
            WHERE u.id = ?
        `, [userId]);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

2. Obtener todos los pedidos (código y fecha) realizados por un usuario con un correo electrónico específico (ej: `isamel@pedrito.es`).

```js
router.get('/ejercicio-2/:email', async (req, res) => {
    try {
        const { email } = req.params;

        const [rows] = await pool.query(`
            SELECT o.order_number, o.order_date
            FROM orders o
            JOIN users u ON u.id = o.user_id
            WHERE u.email = ?
        `, [email]);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

3. Mostrar el nombre de cada producto junto con el nombre de la categoría a la que pertenece.
```js
router.get('/ejercicio-3', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT p.name AS product_name,
                   c.name AS category_name
            FROM products p
            JOIN categories c ON p.category_id = c.id
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

4. Obtener una lista de los usuarios que se han registrado en el sistema pero que nunca han realizado una compra.
```js
router.get('/ejercicio-4', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT u.id, u.name, u.email
            FROM users u
            LEFT JOIN orders o ON u.id = o.user_id
            WHERE o.id IS NULL
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

5. Calcular el monto total gastado de un usuario (el que ustedes elijan) en toda su historia, mostrando el nombre del usuario y el total.
```js
router.get('/ejercicio-5/:userId', async (req, res) => {
    try {
        const { userId } = req.params;

        const [rows] = await pool.query(`
            SELECT u.name,
                   SUM(op.quantity * o.total) AS total_spent
            FROM users u
            JOIN orders o ON o.user_id = u.id
            JOIN order_product op ON op.order_id = o.id
            WHERE u.id = ?
            GROUP BY u.name
        `, [userId]);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

6. Contar cuántos pedidos existen actualmente clasificados por cada estado (`status`).
```js
router.get('/ejercicio-6', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT status, COUNT(*) AS total_orders
            FROM orders
            GROUP BY status
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

7. Listar todos los productos de la categoría `Electrónica` ordenados por precio de venta, del más caro al más barato.
```js

```

8. Dado un número de orden específico, mostrar los IDs de los productos y la cantidad comprada de cada uno en esa orden.
```js

```

9.  Listar los nombres de los usuarios de una ciudad específica (ej: `Monterrey`) que tengan al menos un pedido registrado.
```js

```

10. Calcular el valor promedio de los pedidos realizados por cada usuario.
```js

```

---

## Nivel 2: Consultas Intermedias (3 Tablas)

11. Generar un recibo detallado que muestre:
   - Código de orden  
   - Fecha  
   - Nombre del producto comprado  
   - Precio al que se vendió  

```js

```
12. Calcular el ingreso total generado por cada categoría de productos.
```js

```

13. Listar los nombres únicos de todos los productos que ha comprado un cliente específico (buscar por nombre del cliente).
```js

```

14. Identificar los 5 productos más vendidos históricamente (basado en la cantidad total de unidades).
```js

```

15. Obtener la fecha de la última vez que se vendió cada producto, mostrando el nombre del producto y la fecha.
```js

```

16. Listar los nombres de los usuarios que han comprado al menos un producto que contenga la palabra `Gamer` en su nombre.
```js

```

17. Calcular los ingresos totales de la tienda agrupados por día.
```js

```

18. Identificar las categorías que tienen productos registrados pero que nunca han generado una venta.
```js

```

19. Mostrar el ticket promedio de compra (gasto promedio por orden) de cada usuario.
```js

```

20. Listar los nombres de los productos que formaban parte de órdenes que terminaron siendo canceladas.
```js

```

---

## Nivel 3: Consultas Complejas y Reportes (4+ Tablas)

21. **Reporte Global**  
   Mostrar una tabla con:
   - Nombre del Usuario  
   - Ciudad  
   - Número de Orden  
   - Nombre del Producto  
   - Categoría  
   - Cantidad  
   - Subtotal del ítem  

```js

```
22. Calcular cuánto dinero han generado las ventas de la categoría `Ropa` exclusivamente en una ciudad específica.
```js

```

23. Identificar al **Cliente del Año**:  
   El usuario que ha gastado más dinero en total dentro de la plataforma.
   ```js

```

24. Listar los productos que no han tenido ninguna venta registrada.
```js

```

25. Calcular la **Ganancia Real (Profit)** de la empresa:  
   `(Precio de venta histórico en la orden - Costo de compra del producto)`.
   ```js

```

26. Mostrar los usuarios que han comprado productos de la categoría `Videojuegos` pero no han comprado productos de `Hogar`.
```js

```

27. Generar un ranking de las 3 ciudades que más ingresos han generado a la tienda.
```js

```

28. Encontrar la orden que contiene la mayor variedad de productos distintos (mayor cantidad de ítems únicos).
```js

```

29. Listar los productos que se vendieron en el pasado a un precio menor que su precio de venta actual en catálogo.
```js

```

30. Mostrar el historial de compras de un producto específico:
   - Quién lo compró  
   - Cuándo  
   - A qué precio  

```js

```
---

## Nivel 4: Lógica de Negocio y Analítica Avanzada

31. Listar a los usuarios cuyo gasto total acumulado es superior al promedio de gasto de todos los clientes de la tienda.
```js

```

32. Identificar los productos **Estrella**:  
   Aquellos que representan individualmente más del 2% del total de ingresos de la empresa.
   ```js

```

33. **Churn Rate**  
   Listar los usuarios que hicieron compras en el pasado, pero que no han realizado ningún pedido en los últimos 6 meses.
   ```js

```

34. Clasificar a los clientes en tres niveles según su gasto total:
   - `VIP` → gasto > 5000  
   - `Frecuente` → entre 1000 y 5000  
   - `Regular` → < 1000  

```js

```
35. Determinar cuál ha sido el mes (y año) con mayor facturación en la historia de la tienda.
```js

```

36. **Alerta de Inventario**  
   Listar las órdenes pendientes que incluyen productos cuyo stock actual es menor a 5 unidades.
   ```js

```

37. Calcular qué porcentaje de las ventas totales representa cada categoría  
   (ej: Electrónica 40%, Ropa 20%, etc.).
   ```js

```

38. Comparar las ventas totales de cada ciudad contra el promedio de ventas de todas las ciudades.
```js

```

39. Calcular la tasa de cancelación:  
   Porcentaje de órdenes con estado `cancelled` respecto al total de órdenes por mes.
   ```js

```

40. **Análisis de Canasta**  
   Identificar qué pares de productos se venden juntos con mayor frecuencia en la misma orden.
   ```js

```