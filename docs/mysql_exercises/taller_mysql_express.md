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
router.get('/ejercicio-7', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT p.id, p.name, p.sale_price
            FROM products p
            JOIN categories c ON c.id = p.category_id
            WHERE c.id in (1,2,3,4,5,7)
            ORDER BY p.sale_price DESC
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

8. Dado un número de orden específico, mostrar los IDs de los productos y la cantidad comprada de cada uno en esa orden.
```js
router.get('/ejercicio-8/:orderNumber', async (req, res) => {
    try {
        const { orderNumber } = req.params;

        const [rows] = await pool.query(`
            SELECT op.product_id, op.quantity
            FROM order_product op
            JOIN orders o ON o.id = op.order_id
            WHERE o.order_number = ?
        `, [orderNumber]);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
})
```

9.  Listar los nombres de los usuarios de una ciudad específica (ej: `Monterrey`) que tengan al menos un pedido registrado.
```js
router.get('/ejercicio-9/:city', async (req, res) => {
    try {
        const { city } = req.params;

        const [rows] = await pool.query(`
            SELECT DISTINCT u.name
            FROM users u
            JOIN orders o ON o.user_id = u.id
            WHERE u.city = ?
        `, [city]);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

10. Calcular el valor promedio de los pedidos realizados por cada usuario.
```js
router.get('/ejercicio-10', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT u.name,
                   AVG(order_total) AS average_order_value
            FROM (
                SELECT o.id,
                       o.user_id,
                       SUM(op.quantity * op.price_at_purchase) AS order_total
                FROM orders o
                JOIN order_product op ON op.order_id = o.id
                GROUP BY o.id
            ) AS sub
            JOIN users u ON u.id = sub.user_id
            GROUP BY u.name
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
})
```

---

## Nivel 2: Consultas Intermedias (3 Tablas)

11. Generar un recibo detallado que muestre:
   - Código de orden  
   - Fecha  
   - Nombre del producto comprado  
   - Precio al que se vendió  

```js
router.get('/ejercicio-11/:orderNumber', async (req, res) => {
    try {
        const { orderNumber } = req.params;

        const [rows] = await pool.query(`
            SELECT 
                o.order_number AS codigo_orden,
                o.order_date AS fecha,
                p.name AS producto,
                op.price_at_purchase AS precio_venta
            FROM orders o
            JOIN order_product op ON op.order_id = o.id
            JOIN products p ON p.id = op.product_id
            WHERE o.order_number = ?
        `, [orderNumber]);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```
12. Calcular el ingreso total generado por cada categoría de productos.
```js
router.get('/ejercicio-12', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                c.name AS categoria,
                SUM(op.quantity * op.price_at_purchase ) AS ingresos_totales
            FROM categories c
            JOIN products p ON p.category_id = c.id
            JOIN order_product op ON op.product_id = p.id
            GROUP BY c.name
            ORDER BY ingresos_totales DESC
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

13. Listar los nombres únicos de todos los productos que ha comprado un cliente específico (buscar por nombre del cliente).
```js
router.get('/ejercicio-13/:customerName', async (req, res) => {
    try {
        const { customerName } = req.params;

        const [rows] = await pool.query(`
            SELECT DISTINCT p.name AS producto
            FROM users u
            JOIN orders o ON o.user_id = u.id
            JOIN order_product op ON op.order_id = o.id
            JOIN products p ON p.id = op.product_id
            WHERE u.name = ?
        `, [customerName]);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

14. Identificar los 5 productos más vendidos históricamente (basado en la cantidad total de unidades).
```js
router.get('/ejercicio-14', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                p.name AS producto,
                SUM(op.quantity) AS total_vendidos
            FROM order_product op
            JOIN products p ON p.id = op.product_id
            GROUP BY p.name
            ORDER BY total_vendidos DESC
            LIMIT 5
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
})
```

15. Obtener la fecha de la última vez que se vendió cada producto, mostrando el nombre del producto y la fecha.
```js
router.get('/ejercicio-15', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                p.name AS producto,
                MAX(o.order_date) AS ultima_venta
            FROM products p
            JOIN order_product op ON op.product_id = p.id
            JOIN orders o ON o.id = op.order_id
            GROUP BY p.name
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

16. Listar los nombres de los usuarios que han comprado al menos un producto que contenga la palabra `Gamer` en su nombre.
```js
router.get('/ejercicio-16', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT DISTINCT u.name
            FROM users u
            JOIN orders o ON o.user_id = u.id
            JOIN order_product op ON op.order_id = o.id
            JOIN products p ON p.id = op.product_id
            WHERE p.name LIKE '%Gamer%'
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

17. Calcular los ingresos totales de la tienda agrupados por día.
```js
router.get('/ejercicio-17', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                DATE(o.order_date) AS fecha,
                SUM(op.quantity * op.price_at_purchase) AS ingresos
            FROM orders o
            JOIN order_product op ON op.order_id = o.id
            GROUP BY DATE(o.order_date)
            ORDER BY fecha
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

18. Identificar las categorías que tienen productos registrados pero que nunca han generado una venta.
```js
router.get('/ejercicio-18', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT c.name AS categoria
            FROM categories c
            LEFT JOIN products p ON p.category_id = c.id
            LEFT JOIN order_product op ON op.product_id = p.id
            WHERE op.id IS NULL
            GROUP BY c.name
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

19. Mostrar el ticket promedio de compra (gasto promedio por orden) de cada usuario.
```js
router.get('/ejercicio-19', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                u.name,
                AVG(order_total) AS ticket_promedio
            FROM (
                SELECT 
                    o.id,
                    o.user_id,
                    SUM(op.quantity * op.price_at_purchase) AS order_total
                FROM orders o
                JOIN order_product op ON op.order_id = o.id
                GROUP BY o.id
            ) AS sub
            JOIN users u ON u.id = sub.user_id
            GROUP BY u.name
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

20. Listar los nombres de los productos que formaban parte de órdenes que terminaron siendo canceladas.
```js
router.get('/ejercicio-20', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT DISTINCT p.name AS producto, o.status AS staus
            FROM orders o
            JOIN order_product op ON op.order_id = o.id
            JOIN products p ON p.id = op.product_id
            WHERE o.status = 'cancelled'
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
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
router.get('/ejercicio-21', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                u.name AS usuario,
                u.city AS ciudad,
                o.order_number AS numero_orden,
                p.name AS producto,
                c.name AS categoria,
                op.quantity AS cantidad,
                (op.quantity * op.price_at_purchase) AS subtotal
            FROM orders o
            JOIN users u ON u.id = o.user_id
            JOIN order_product op ON op.order_id = o.id
            JOIN products p ON p.id = op.product_id
            JOIN categories c ON c.id = p.category_id
            ORDER BY o.order_number
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```
22. Calcular cuánto dinero han generado las ventas de la categoría `Ropa` exclusivamente en una ciudad específica.
```js
// router.get('/ejercicio-22/:city', async (req, res) => {
//     try {
//         const { city } = req.params;

//         const [rows] = await pool.query(`
//             SELECT 
//                 SUM(op.quantity * op.price_at_purchase) AS total_ropa
//             FROM orders o
//             JOIN users u ON u.id = o.user_id
//             JOIN order_product op ON op.order_id = o.id
//             JOIN products p ON p.id = op.product_id
//             JOIN categories c ON c.id = p.category_id
//             WHERE c.name = 'Ropa'
//             AND u.city = ?
//         `, [city]);

//         res.json(rows);
//     } catch (error) {
//         res.status(500).json({ error: error.message });
//     }
// })
```

23. Identificar al **Cliente del Año**:  
   El usuario que ha gastado más dinero en total dentro de la plataforma.
```js
router.get('/ejercicio-23', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                u.name,
                SUM(op.quantity * op.price_at_purchase) AS total_gastado
            FROM users u
            JOIN orders o ON o.user_id = u.id
            JOIN order_product op ON op.order_id = o.id
            GROUP BY u.id
            ORDER BY total_gastado DESC
            LIMIT 1
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

24. Listar los productos que no han tenido ninguna venta registrada.
```js
// router.get('/ejercicio-24', async (req, res) => {
//     try {
//         const [rows] = await pool.query(`
//             SELECT p.name
//             FROM products p
//             LEFT JOIN order_product op ON op.product_id = p.id
//             WHERE op.product_id IS NULL
//         `);

//         res.json(rows);
//     } catch (error) {
//         res.status(500).json({ error: error.message });
//     }
// });
```

25. Calcular la **Ganancia Real (Profit)** de la empresa:  
   `(Precio de venta histórico en la orden - Costo de compra del producto)`.
```js
router.get('/ejercicio-25', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                SUM((op.price_at_purchase - p.purchase_price) * op.quantity) AS ganancia_total
            FROM order_product op
            JOIN products p ON p.id = op.product_id
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

26. Mostrar los usuarios que han comprado productos de la categoría `Videojuegos` pero no han comprado productos de `Hogar`.
```js
// router.get('/ejercicio-26', async (req, res) => {
//     try {
//         const [rows] = await pool.query(`
//             SELECT DISTINCT u.name
//             FROM users u
//             WHERE EXISTS (
//                 SELECT 1
//                 FROM orders o
//                 JOIN order_product op ON op.order_id = o.id
//                 JOIN products p ON p.id = op.product_id
//                 JOIN categories c ON c.id = p.category_id
//                 WHERE o.user_id = u.id
//                 AND c.name = 'Videojuegos'
//             )
//             AND NOT EXISTS (
//                 SELECT 1
//                 FROM orders o
//                 JOIN order_product op ON op.order_id = o.id
//                 JOIN products p ON p.id = op.product_id
//                 JOIN categories c ON c.id = p.category_id
//                 WHERE o.user_id = u.id
//                 AND c.name = 'Hogar'
//             )
//         `);

//         res.json(rows);
//     } catch (error) {
//         res.status(500).json({ error: error.message });
//     }
// });
```

27. Generar un ranking de las 3 ciudades que más ingresos han generado a la tienda.
```js
router.get('/ejercicio-27', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                u.city,
                SUM(op.quantity * op.price_at_purchase) AS ingresos
            FROM users u
            JOIN orders o ON o.user_id = u.id
            JOIN order_product op ON op.order_id = o.id
            GROUP BY u.city
            ORDER BY ingresos DESC
            LIMIT 3
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

28. Encontrar la orden que contiene la mayor variedad de productos distintos (mayor cantidad de ítems únicos).
```js
router.get('/ejercicio-28', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                o.order_number,
                COUNT(DISTINCT op.product_id) AS productos_distintos
            FROM orders o
            JOIN order_product op ON op.order_id = o.id
            GROUP BY o.id
            ORDER BY productos_distintos DESC
            LIMIT 1
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

29. Listar los productos que se vendieron en el pasado a un precio menor que su precio de venta actual en catálogo.
```js
// router.get('/ejercicio-29', async (req, res) => {
//     try {
//         const [rows] = await pool.query(`
//             SELECT DISTINCT p.name
//             FROM products p
//             JOIN order_product op ON op.product_id = p.id
//             WHERE op.price_at_purchase < p.purchase_price
//         `);

//         res.json(rows);
//     } catch (error) {
//         res.status(500).json({ error: error.message });
//     }
// });
```

30. Mostrar el historial de compras de un producto específico:
   - Quién lo compró  
   - Cuándo  
   - A qué precio  

```js
router.get('/ejercicio-30/:productId', async (req, res) => {
    try {
        const { productId } = req.params;

        const [rows] = await pool.query(`
            SELECT 
                u.name AS usuario,
                o.order_date AS fecha,
                op.price_at_purchase AS precio_pagado,
                op.quantity
            FROM order_product op
            JOIN orders o ON o.id = op.order_id
            JOIN users u ON u.id = o.user_id
            WHERE op.product_id = ?
            ORDER BY o.order_date DESC
        `, [productId]);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```
---

## Nivel 4: Lógica de Negocio y Analítica Avanzada

31. Listar a los usuarios cuyo gasto total acumulado es superior al promedio de gasto de todos los clientes de la tienda.
```js
// router.get('/ejercicio-31', async (req, res) => {
//     try {
//         const [rows] = await pool.query(`
//             SELECT u.name, total_gastado
//             FROM (
//                 SELECT u.id, u.name,
//                        SUM(op.quantity * op.price_at_purchase) AS total_gastado
//                 FROM users u
//                 JOIN orders o ON o.user_id = u.id
//                 JOIN order_product op ON op.order_id = o.id
//                 GROUP BY u.id
//             ) AS sub
//             WHERE total_gastado > (
//                 SELECT AVG(total_cliente)
//                 FROM (
//                     SELECT SUM(op.quantity * op.price_at_purchase) AS total_cliente
//                     FROM orders o
//                     JOIN order_product op ON op.order_id = o.id
//                     GROUP BY o.user_id
//                 ) AS promedio
//             )
//         `);

//         res.json(rows);
//     } catch (error) {
//         res.status(500).json({ error: error.message });
//     }
// });
```

32. Identificar los productos **Estrella**:  
   Aquellos que representan individualmente más del 2% del total de ingresos de la empresa.
```js
// router.get('/ejercicio-32', async (req, res) => {
//     try {
//         const [rows] = await pool.query(`
//             SELECT p.name,
//                    SUM(op.quantity * op.price_at_purchase) AS ingresos_producto,
//                    ROUND(
//                        (SUM(op.quantity * op.price_at_purchase) /
//                        (SELECT SUM(quantity * price_at_purchase) FROM order_product)) * 100, 2
//                    ) AS porcentaje
//             FROM products p
//             JOIN order_product op ON op.product_id = p.id
//             GROUP BY p.id
//             HAVING porcentaje > 2
//             ORDER BY porcentaje DESC
//         `);

//         res.json(rows);
//     } catch (error) {
//         res.status(500).json({ error: error.message });
//     }
// });
```

33. **Churn Rate**  
   Listar los usuarios que hicieron compras en el pasado, pero que no han realizado ningún pedido en los últimos 6 meses.
```js
router.get('/ejercicio-33', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT u.name, MAX(o.order_date) AS ultima_compra
            FROM users u
            JOIN orders o ON o.user_id = u.id
            GROUP BY u.id
            HAVING MAX(o.order_date) < DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

34. Clasificar a los clientes en tres niveles según su gasto total:
   - `VIP` → gasto > 5000  
   - `Frecuente` → entre 1000 y 5000  
   - `Regular` → < 1000  

```js
router.get('/ejercicio-34', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT u.name,
                   SUM(op.quantity * op.price_at_purchase) AS total_gastado,
                   CASE
                       WHEN SUM(op.quantity * op.price_at_purchase) > 5000 THEN 'VIP'
                       WHEN SUM(op.quantity * op.price_at_purchase) BETWEEN 1000 AND 5000 THEN 'Frecuente'
                       ELSE 'Regular'
                   END AS categoria_cliente
            FROM users u
            JOIN orders o ON o.user_id = u.id
            JOIN order_product op ON op.order_id = o.id
            GROUP BY u.id
            ORDER BY total_gastado DESC
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```
35. Determinar cuál ha sido el mes (y año) con mayor facturación en la historia de la tienda.
```js
router.get('/ejercicio-35', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                YEAR(o.order_date) AS año,
                MONTH(o.order_date) AS mes,
                SUM(op.quantity * op.price_at_purchase) AS facturacion
            FROM orders o
            JOIN order_product op ON op.order_id = o.id
            GROUP BY año, mes
            ORDER BY facturacion DESC
            LIMIT 1
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

36. **Alerta de Inventario**  
   Listar las órdenes pendientes que incluyen productos cuyo stock actual es menor a 5 unidades.
```js
router.get('/ejercicio-36', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT DISTINCT o.order_number, p.name, p.stock
            FROM orders o
            JOIN order_product op ON op.order_id = o.id
            JOIN products p ON p.id = op.product_id
            WHERE o.status = 'pending'
            AND p.stock < 5
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

37. Calcular qué porcentaje de las ventas totales representa cada categoría  
   (ej: Electrónica 40%, Ropa 20%, etc.).
```js
router.get('/ejercicio-37', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT c.name,
                   ROUND(
                       (SUM(op.quantity * op.price_at_purchase) /
                       (SELECT SUM(quantity * price_at_purchase) FROM order_product)) * 100, 2
                   ) AS porcentaje
            FROM categories c
            JOIN products p ON p.category_id = c.id
            JOIN order_product op ON op.product_id = p.id
            GROUP BY c.id
            ORDER BY porcentaje DESC
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

38. Comparar las ventas totales de cada ciudad contra el promedio de ventas de todas las ciudades.
```js
router.get('/ejercicio-38', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT u.city,
                   SUM(op.quantity * op.price_at_purchase) AS total_ciudad,
                   (
                       SELECT AVG(total_ciudad)
                       FROM (
                           SELECT SUM(op2.quantity * op2.price_at_purchase) AS total_ciudad
                           FROM users u2
                           JOIN orders o2 ON o2.user_id = u2.id
                           JOIN order_product op2 ON op2.order_id = o2.id
                           GROUP BY u2.city
                       ) AS promedio
                   ) AS promedio_general
            FROM users u
            JOIN orders o ON o.user_id = u.id
            JOIN order_product op ON op.order_id = o.id
            GROUP BY u.city
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

39. Calcular la tasa de cancelación:  
   Porcentaje de órdenes con estado `cancelled` respecto al total de órdenes por mes.
```js
router.get('/ejercicio-39', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                YEAR(order_date) AS año,
                MONTH(order_date) AS mes,
                ROUND(
                    (SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) 
                    / COUNT(*)) * 100, 2
                ) AS tasa_cancelacion
            FROM orders
            GROUP BY año, mes
            ORDER BY año, mes
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

40. **Análisis de Canasta**  
   Identificar qué pares de productos se venden juntos con mayor frecuencia en la misma orden.
```js
router.get('/ejercicio-40', async (req, res) => {
    try {
        const [rows] = await pool.query(`
            SELECT 
                p1.name AS producto_1,
                p2.name AS producto_2,
                COUNT(*) AS veces_juntos
            FROM order_product op1
            JOIN order_product op2 
                ON op1.order_id = op2.order_id
                AND op1.product_id < op2.product_id
            JOIN products p1 ON p1.id = op1.product_id
            JOIN products p2 ON p2.id = op2.product_id
            GROUP BY producto_1, producto_2
            ORDER BY veces_juntos DESC
            LIMIT 10
        `);

        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```