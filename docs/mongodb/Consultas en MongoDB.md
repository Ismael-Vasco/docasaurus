# Consultas en MongoDB  

**Colección:** `users`

---

## Nivel 1 – Consultas Básicas

1. Listar todos los documentos de la colección `users`.
```js
db.users.find()
```

2. Mostrar únicamente los campos `first_name`, `last_name` y `email`.
```js
db.users.find(
  {},
  { first_name: 1, last_name: 1, email: 1, _id: 0 }
)
```

3. Obtener todos los usuarios cuyo `role` sea `"admin"`.
```js
db.users.find({ role: "admin" })
```

4. Buscar los usuarios cuyo `country` sea `"Colombia"`.
```js
db.users.find({ country: "Colombia" })

```

5. Listar los usuarios que estén activos (`is_active = true`).
```js
db.users.find({ is_active: true })
```

6. Buscar los usuarios que no estén verificados (`is_verified = false`).
```js
db.users.find({ is_verified: false })
```

7. Obtener los usuarios cuyo `gender` sea `"Masculino"`.
```js
db.users.find({ gender: "Masculino" })
```

8. Listar los usuarios que vivan en la ciudad `"Medellín"`.
```js
db.users.find({ city: "Medellín" })
```

9. Buscar los usuarios que tengan al menos un hijo (`children_count > 0`).
```js
db.users.find({ children_count: { $gt: 0 } })
```

10. Listar los usuarios cuya profesión (`profession`) no sea null.
```js
db.users.find({ profession: { $ne: null } })
```

---

## Nivel 2 – Filtros con Operadores

11. Buscar usuarios con `monthly_income` mayor a 3000000.
```js
db.users.find({ monthly_income: { $gt: 3000000 } })
```

12. Buscar usuarios con ingresos entre 2000000 y 5000000.
```js
db.users.find({
  monthly_income: { $gte: 2000000, $lte: 5000000 }
})
```

13. Buscar usuarios cuya fecha de nacimiento sea posterior al `2000-01-01`.
```js
db.users.find({
  birth_date: { $gt: new Date("2000-01-01") }
})
```

14. Buscar usuarios cuyo `document_type` esté en `["CC", "CE"]`.
```js
db.users.find({
  document_type: { $in: ["CC", "CE"] }
})
```

15. Buscar usuarios cuyo `city` no sea `"Bogotá"`.
```js
db.users.find({
  city: { $ne: "Bogotá" }
})
```

16. Buscar usuarios cuyo nombre empiece por la letra `"A"`.
```js
db.users.find({
  first_name: { $regex: /^A/i }
})
```

17. Buscar usuarios cuyo email termine en `"gmail.com"`.
```js
db.users.find({
  email: { $regex: /gmail\.com$/i }
})
```

18. Buscar usuarios que tengan más de 2 hijos y estén activos.
```js
db.users.find({
  children_count: { $gt: 2 },
  is_active: true
})
```

19. Buscar usuarios cuyo `marital_status` sea `"Casado"` y tengan hijos.
```js
db.users.find({
  marital_status: "Casado",
  children_count: { $gt: 0 }
})
```

20. Buscar usuarios que estén inactivos o no verificados.
```js
db.users.find({
  $or: [
    { is_active: false },
    { is_verified: false }
  ]
})
```

---

## Nivel 3 – Ordenamiento y Paginación

21. Listar usuarios ordenados por `monthly_income` de mayor a menor.
```js
db.users.find().sort({ monthly_income: -1 })
```

22. Obtener los 5 usuarios más recientes según `created_at`.
```js
db.users.find()
  .sort({ created_at: -1 })
  .limit(5)
```

23. Implementar paginación: mostrar la página 2 con 10 registros por página.
```js
db.users.find()
  .skip(10)   // (page - 1) * limit → (2 - 1) * 10
  .limit(10)
```

24. Mostrar nombre completo concatenado (`first_name` + `last_name`) y ciudad usando agregación.
```js
db.users.aggregate([
  {
    $project: {
      _id: 0,
      full_name: { $concat: ["$first_name", " ", "$last_name"] },
      city: 1
    }
  }
])
```

25. Listar usuarios ordenados por fecha de nacimiento del más joven al mayor.
```js
db.users.find()
  .sort({ birth_date: -1 })
```

---

## Nivel 4 – Aggregation Framework

26. Calcular el ingreso promedio (`monthly_income`) de todos los usuarios.
```js
db.users.aggregate([
  {
    $group: {
      _id: null,
      average_income: { $avg: "$monthly_income" }
    }
  }
])
```

27. Calcular el ingreso promedio por ciudad.
```js
db.users.aggregate([
  {
    $group: {
      _id: "$city",
      average_income: { $avg: "$monthly_income" }
    }
  }
])
```

28. Contar cuántos usuarios hay por cada `role`.
```js
db.users.aggregate([
  {
    $group: {
      _id: "$role",
      total: { $sum: 1 }
    }
  }
])
```

29. Contar cuántos usuarios están activos vs inactivos.
```js
db.users.aggregate([
  {
    $group: {
      _id: "$is_active",
      total: { $sum: 1 }
    }
  }
])
```

30. Obtener la cantidad total de hijos agrupados por `state`.
```js
db.users.aggregate([
  {
    $group: {
      _id: "$state",
      total_children: { $sum: "$children_count" }
    }
  }
])
```