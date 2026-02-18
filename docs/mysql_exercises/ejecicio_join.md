```sql
    create table profesores (
        
        id int PRIMARY KEY auto_increment,
        name varchar(55),
        lastname varchar(55)
    )

    create table estudiante (
        id int PRIMARY KEY auto_increment,
        name varchar(55),
        lastname varchar(55),
        email varchar(100) UNIQUE 
    )

    create TABLE cursos (
        id int PRIMARY KEY auto_increment,
        name varchar(55),
        profesor_id int,
        foreign KEY (profesor_id) REFERENCES profesores(id)
    )

    create table estudiante_has_curso (
        id int PRIMARY KEY auto_increment,
        estudiante_id int,
        curso_id int,
        foreign KEY (estudiante_id) REFERENCES estudiante(id),
        foreign KEY (curso_id) REFERENCES cursos(id)
    )


    ---- eliminar constrain primero
    ALTER TABLE cursos DROP FOREIGN KEY cursos_ibfk_1;
    --- luego eliminar columna
    alter table cursos drop column estudiante_id


    ---------------- insert ------------------------------------
    insert into profesores (name,lastname) values
    ('Carlos', 'Pérez'),
    ('Laura', 'Gómez'),
    ('Miguel', 'Castro');

    insert into estudiante (name,lastname, email) values
    ('Ana', 'Torres','ana@email.com'),
    ('Luis', 'Martínez','luis@email.com'),
    ('Sofía', 'Ramírez','sofia@email.com');



    insert into cursos (name, profesor_id) values
    ('Bases de Datos', 1),
    ('Programación I', 2),
    ('Algoritmos', 3);


    insert into estudiante_has_curso (estudiante_id, curso_id) values
    (1,1),
    (1,2),
    (2,1),
    (3,3),
    (2,3),
    (3,1);

    -------------------- show data
    SELECT  * FROM estudiante e 

    SELECT * FROM profesores p 

    SELECT * FROM cursos c 

    SELECT * FROM estudiante_has_curso ehc

    ----------------------- join
    SELECT e.name, e.lastname, e.email,ehc.id, c.name, p.name, p.lastname 
    FROM estudiante_has_curso ehc
    inner join cursos c 
    on ehc.curso_id = c.id
    inner join estudiante e
    on ehc.estudiante_id = e.id
    INNER join profesores p
    on c.profesor_id = p.id 
```

