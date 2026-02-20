### TASK 1 – Creación de la Base de Datos y Tablas (DDL)

- Crear base de datos
```sql
CREATE DATABASE gestion_academica_universidad;
USE gestion_academica_universidad;
```

- Tabla estudiantes
```sql
CREATE TABLE estudiantes (
    id_estudiante INT AUTO_INCREMENT PRIMARY KEY,
    nombre_completo VARCHAR(100) NOT NULL,
    correo_electronico VARCHAR(100) NOT NULL UNIQUE,
    genero ENUM('M','F','Otro') NOT NULL,
    identificacion VARCHAR(20) NOT NULL UNIQUE,
    carrera VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE NOT NULL,
    fecha_ingreso DATE NOT NULL,
    CHECK (fecha_ingreso >= fecha_nacimiento)
);
```
- Tabla docentes
```sql
CREATE TABLE docentes (
    id_docente INT AUTO_INCREMENT PRIMARY KEY,
    nombre_completo VARCHAR(100) NOT NULL,
    correo_institucional VARCHAR(100) NOT NULL UNIQUE,
    departamento_academico VARCHAR(100) NOT NULL,
    anios_experiencia INT NOT NULL CHECK (anios_experiencia >= 0)
);
```
- Tabla cursos
```sql
CREATE TABLE cursos (
    id_curso INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    codigo VARCHAR(20) NOT NULL UNIQUE,
    creditos INT NOT NULL CHECK (creditos > 0),
    semestre INT NOT NULL CHECK (semestre >= 1),
    id_docente INT,
    FOREIGN KEY (id_docente) REFERENCES docentes(id_docente)
        ON DELETE SET NULL
);
```
- Tabla inscripciones
```sql
CREATE TABLE inscripciones (
    id_inscripcion INT AUTO_INCREMENT PRIMARY KEY,
    id_estudiante INT NOT NULL,
    id_curso INT NOT NULL,
    fecha_inscripcion DATE NOT NULL,
    calificacion_final DECIMAL(4,2) CHECK (calificacion_final BETWEEN 0 AND 5),
    FOREIGN KEY (id_estudiante) REFERENCES estudiantes(id_estudiante)
        ON DELETE CASCADE,
    FOREIGN KEY (id_curso) REFERENCES cursos(id_curso)
        ON DELETE CASCADE
);
```
------------------------------------------------------------------------------

### TASK 2 – Inserción de Datos (DML)
- Insertar estudiantes
```sql
INSERT INTO estudiantes (nombre_completo, correo_electronico, genero, identificacion, carrera, fecha_nacimiento, fecha_ingreso) VALUES
('Juan Perez','juan@correo.com','M','1001','Ingenieria','2000-05-10','2018-02-01'),
('Maria Lopez','maria@correo.com','F','1002','Administracion','1999-08-15','2017-02-01'),
('Carlos Gomez','carlos@correo.com','M','1003','Derecho','2001-01-20','2019-02-01'),
('Laura Torres','laura@correo.com','F','1004','Ingenieria','2000-11-25','2018-02-01'),
('Andres Ruiz','andres@correo.com','M','1005','Medicina','1998-07-30','2016-02-01');
```
- Insertar docentes
```sql
INSERT INTO docentes (nombre_completo, correo_institucional, departamento_academico, anios_experiencia) VALUES
('Dr. Alberto Sanchez','alberto@uni.edu','Ingenieria',10),
('Dra. Patricia Ramirez','patricia@uni.edu','Administracion',7),
('Dr. Miguel Herrera','miguel@uni.edu','Derecho',4);
```
- Insertar cursos
```sql
INSERT INTO cursos (nombre, codigo, creditos, semestre, id_docente) VALUES
('Base de Datos','BD101',3,2,1),
('Matematicas I','MAT101',4,1,1),
('Derecho Civil','DER201',3,3,3),
('Administracion General','ADM101',2,2,2);
```

- Insertar inscripciones (8)
```sql
INSERT INTO inscripciones (id_estudiante, id_curso, fecha_inscripcion, calificacion_final) VALUES
(1,1,'2024-02-01',4.5),
(1,2,'2024-02-01',3.8),
(2,4,'2024-02-01',4.2),
(2,1,'2024-02-01',3.5),
(3,3,'2024-02-01',4.7),
(4,1,'2024-02-01',4.0),
(4,4,'2024-02-01',3.9),
(5,2,'2024-02-01',3.2);
```
------------------------------------------------------------------------------

### TASK 3 – Consultas y Manipulación (DQL + DDL)
1. JOIN estudiantes con cursos
```sql
SELECT e.nombre_completo, c.nombre AS curso, i.calificacion_final
FROM estudiantes e
JOIN inscripciones i ON e.id_estudiante = i.id_estudiante
JOIN cursos c ON i.id_curso = c.id_curso;
```
2. Cursos dictados por docentes con > 5 años
```sql
SELECT c.nombre, d.nombre_completo, d.anios_experiencia
FROM cursos c
JOIN docentes d ON c.id_docente = d.id_docente
WHERE d.anios_experiencia > 5;
```
3. Promedio por curso
```sql
SELECT c.nombre, AVG(i.calificacion_final) AS promedio
FROM cursos c
JOIN inscripciones i ON c.id_curso = i.id_curso
GROUP BY c.nombre;
```
4. Estudiantes en más de un curso
```sql
SELECT e.nombre_completo, COUNT(*) AS total_cursos
FROM estudiantes e
JOIN inscripciones i ON e.id_estudiante = i.id_estudiante
GROUP BY e.nombre_completo
HAVING COUNT(*) > 1;
```
5. ALTER TABLE
```sql
ALTER TABLE estudiantes
ADD estado_academico VARCHAR(20) DEFAULT 'Activo';
```
6. Eliminar docente (ver efecto ON DELETE SET NULL)
```sql
DELETE FROM docentes WHERE id_docente = 3;
```
7. Cursos con más de 2 estudiantes
```sql
SELECT c.nombre, COUNT(i.id_estudiante) AS total_estudiantes
FROM cursos c
JOIN inscripciones i ON c.id_curso = i.id_curso
GROUP BY c.nombre
HAVING COUNT(i.id_estudiante) > 2;
```
------------------------------------------------------------------------------

### TASK 4 – Subconsultas y Funciones

1. Estudiantes con promedio > promedio general
```sql
SELECT e.nombre_completo, AVG(i.calificacion_final) AS promedio_estudiante
FROM estudiantes e
JOIN inscripciones i ON e.id_estudiante = i.id_estudiante
GROUP BY e.id_estudiante
HAVING AVG(i.calificacion_final) >
    (SELECT AVG(calificacion_final) FROM inscripciones);
```
2. Carreras con estudiantes en semestre ≥ 2
```sql
SELECT DISTINCT carrera
FROM estudiantes
WHERE id_estudiante IN (
    SELECT i.id_estudiante
    FROM inscripciones i
    JOIN cursos c ON i.id_curso = c.id_curso
    WHERE c.semestre >= 2
);
```
3. Indicadores con funciones
```sql
SELECT 
    ROUND(AVG(calificacion_final),2) AS promedio_general,
    SUM(calificacion_final) AS suma_total,
    MAX(calificacion_final) AS nota_maxima,
    MIN(calificacion_final) AS nota_minima,
    COUNT(*) AS total_registros
FROM inscripciones;
```
------------------------------------------------------------------------------

### TASK 5 – Crear Vista
```sql
CREATE VIEW vista_historial_academico AS
SELECT 
    e.nombre_completo AS estudiante,
    c.nombre AS curso,
    d.nombre_completo AS docente,
    c.semestre,
    i.calificacion_final
FROM inscripciones i
JOIN estudiantes e ON i.id_estudiante = e.id_estudiante
JOIN cursos c ON i.id_curso = c.id_curso
LEFT JOIN docentes d ON c.id_docente = d.id_docente;
```
------------------------------------------------------------------------------
### TASK 6 – Permisos y Transacciones (DCL + TCL)

1. Crear rol y otorgar permisos
```sql
CREATE ROLE revisor_academico;

GRANT SELECT ON gestion_academica_universidad.vista_historial_academico TO revisor_academico;

REVOKE INSERT, UPDATE, DELETE 
ON gestion_academica_universidad.inscripciones 
FROM revisor_academico;
```
2. Transacciones
```sql
START TRANSACTION;

UPDATE inscripciones 
SET calificacion_final = 4.8 
WHERE id_inscripcion = 1;

SAVEPOINT cambio1;

UPDATE inscripciones 
SET calificacion_final = 2.0 
WHERE id_inscripcion = 2;

ROLLBACK TO cambio1;

COMMIT;
```