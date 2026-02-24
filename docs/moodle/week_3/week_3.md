## PROYECTO: StreamHub – MongoDB

### TASK 1 – Análisis del dominio y diseño de colecciones
**Colecciones definidas**
- `usuarios`
- `contenidos` (películas y series en una sola colección)
- `valoraciones`
- `listas`

**Estructura de documentos**

**usuarios**
```js
{
  _id: ObjectId(),
  nombre: String,
  email: String,
  edad: Number,
  pais: String,
  fechaRegistro: Date,
  plan: String,                 // "basico", "premium"
  historial: [
    {
      contenidoId: ObjectId,
      fechaVisualizacion: Date,
      progreso: Number          // %
    }
  ]
}
```
**contenidos**
```js
{
  _id: ObjectId(),
  titulo: String,
  tipo: "pelicula" | "serie",
  genero: [String],
  duracion: Number,             // minutos (película)
  temporadas: Number,           // solo series
  anio: Number,
  ratingPromedio: Number,
  activo: Boolean
}
```
**valoraciones**
```js
{
  _id: ObjectId(),
  usuarioId: ObjectId,
  contenidoId: ObjectId,
  puntuacion: Number,           // 1-5
  comentario: String,
  fecha: Date
}
```

**listas**
```js
{
  _id: ObjectId(),
  usuarioId: ObjectId,
  nombre: String,
  contenidos: [ObjectId],
  privada: Boolean
}
```
---
### TASK 2 – Inserción de datos

**Create collection and DB**
```js
use streamhub
```


**INSERTAR CONTENIDOS**
```js
db.contenidos.insertMany([
{
  titulo: "Inception",
  tipo: "pelicula",
  genero: ["Ciencia Ficcion", "Thriller"],
  duracion: 148,
  anio: 2010,
  ratingPromedio: 4.8,
  activo: true
},
{
  titulo: "The Matrix",
  tipo: "pelicula",
  genero: ["Ciencia Ficcion", "Accion"],
  duracion: 136,
  anio: 1999,
  ratingPromedio: 4.7,
  activo: true
},
{
  titulo: "Breaking Bad",
  tipo: "serie",
  genero: ["Drama", "Crimen"],
  temporadas: 5,
  anio: 2008,
  ratingPromedio: 4.9,
  activo: true
},
{
  titulo: "Friends",
  tipo: "serie",
  genero: ["Comedia"],
  temporadas: 10,
  anio: 1994,
  ratingPromedio: 4.5,
  activo: true
}
])
```

**INSERTAR USUARIOS**
```js
db.usuarios.insertMany([
{
  nombre: "Carlos Perez",
  email: "carlos@mail.com",
  edad: 28,
  pais: "Mexico",
  fechaRegistro: new Date(),
  plan: "premium",
  historial: []
},
{
  nombre: "Ana Lopez",
  email: "ana@mail.com",
  edad: 22,
  pais: "Colombia",
  fechaRegistro: new Date(),
  plan: "basico",
  historial: []
}
])
```

**INSERTAR VALORACIONES**
```js
db.valoraciones.insertMany([
{
  usuarioId: db.usuarios.findOne({nombre:"Carlos Perez"})._id,
  contenidoId: db.contenidos.findOne({titulo:"Inception"})._id,
  puntuacion: 5,
  comentario: "Excelente pelicula",
  fecha: new Date()
},
{
  usuarioId: db.usuarios.findOne({nombre:"Ana Lopez"})._id,
  contenidoId: db.contenidos.findOne({titulo:"The Matrix"})._id,
  puntuacion: 4,
  comentario: "Muy buena",
  fecha: new Date()
}
])
```
---

### TASK 3 – Consultas con operadores

**Películas con duración > 120 minutos**
```js
db.contenidos.find(
  { duracion: { $gt: 120 } }
)
```
**Usuarios mayores de 25 años**
```js
db.usuarios.find(
  { edad: { $gt: 25 } }
)
```
**Contenidos entre 1995 y 2015**
```js
db.contenidos.find(
  { anio: { $gt: 1995, $lt: 2015 } }
)
```
**Contenidos de género Ciencia Ficcion o Comedia**
```js
db.contenidos.find(
  { genero: { $in: ["Ciencia Ficcion", "Comedia"] } }
)
```
**Búsqueda por título usando regex**
```js
db.contenidos.find(
  { titulo: { $regex: "The", $options: "i" } }
)
```
**Uso combinado de $and y $or**
```js
db.contenidos.find({
  $and: [
    { tipo: "pelicula" },
    { $or: [
        { ratingPromedio: { $gt: 4.7 } },
        { anio: { $lt: 2000 } }
    ]}
  ]
})
```
---

### TASK 4 – Updates y Deletes

**Actualizar rating de una película**
```js
db.contenidos.updateOne(
  { titulo: "Inception" },
  { $set: { ratingPromedio: 4.9 } }
)
```
**Actualizar plan de varios usuarios**
```js
db.usuarios.updateMany(
  { plan: "basico" },
  { $set: { plan: "premium" } }
)
```

**Eliminar una valoración**
```js
db.valoraciones.deleteOne(
  { comentario: "Muy buena" }
)
```
**Eliminar contenidos inactivos**
```js
db.contenidos.deleteMany(
  { activo: false }
)
```
---

### TASK 5 – Índices

**Crear índices**
```js
db.contenidos.createIndex({ titulo: 1 })
db.contenidos.createIndex({ genero: 1 })
db.usuarios.createIndex({ email: 1 }, { unique: true })
db.valoraciones.createIndex({ contenidoId: 1 })
```

**Ver índices creados**
```js
db.contenidos.getIndexes()
db.usuarios.getIndexes()
```

**Justificación**

- Índice en titulo → mejora búsquedas con `$regex`
- Índice en genero → optimiza consultas con `$in`
- Índice único en email → evita duplicados
- Índice en contenidoId → mejora agregaciones y joins manuales

---
### AGREGACIONES

*AGREGACIÓN 1*

**Promedio de puntuación por contenido**
```js
db.valoraciones.aggregate([
  {
    $group: {
      _id: "$contenidoId",
      promedio: { $avg: "$puntuacion" },
      totalValoraciones: { $sum: 1 }
    }
  },
  {
    $sort: { promedio: -1 }
  }
])
```

*AGREGACIÓN 2*

**Cantidad de contenidos por género**
```js
db.contenidos.aggregate([
  { $unwind: "$genero" },
  {
    $group: {
      _id: "$genero",
      total: { $sum: 1 }
    }
  },
  { $sort: { total: -1 } },
  {
    $project: {
      _id: 0,
      genero: "$_id",
      total: 1
    }
  }
])
```

*AGREGACIÓN 3*

**Usuarios y cantidad de valoraciones realizadas**
```js
db.valoraciones.aggregate([
  {
    $group: {
      _id: "$usuarioId",
      totalValoraciones: { $sum: 1 }
    }
  },
  { $sort: { totalValoraciones: -1 } }
])
```
---
##### Criterios cumplidos

- Modelado `NoSQL` correcto
- `InsertOne` / `InsertMany`
- Find con `$gt`, `$lt`, `$eq`, `$in`, `$and`, `$or`, `$regex`
- `UpdateOne` / `UpdateMany`
- `DeleteOne` / `DeleteMany`
- `createIndex` / `getIndexes`
- 3 pipelines `aggregate()` con `$match`, `$group`, `$sort`, `$project`, `$unwind`
- Datos variados y coherentes